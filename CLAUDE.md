# Pulse Check

Webcam heart-rate prototype (rPPG) with a built-in trust score. Built by Aida Valiyeva as the build phase of her TKS rPPG focus. Long-term goal: a reliable rPPG data pipeline that becomes a camera layer in the Hemio app.

- Live: https://rppg-pulse-check.vercel.app (Vercel project `rppg-pulse-check`)
- Learn article: https://medium.com/@aida.valiyeva2017/whats-stopping-your-webcam-from-becoming-a-health-sensor-d11c61aa8964

## How it works (current V1)

Everything lives in `index.html` (HTML, CSS and JS inline; GSAP 3.12.5 + ScrollTrigger from cdnjs; Google Fonts Archivo + Newsreader).

1. `getUserMedia` webcam, each frame drawn to a 160x120 canvas.
2. Mean RGB of a fixed forehead ROI (`ROI` constant), plus frame-to-frame pixel difference (motion) and luminance.
3. Last 10 s resampled to 30 Hz, POS algorithm (Wang et al. 2017, window 1.6 s), detrend + light smoothing.
4. Hann-windowed DFT over 0.6 to 4.5 Hz; peak in 0.7 to 3.5 Hz = heart rate. SNR = power near peak and 2nd harmonic vs rest.
5. Trust score = 0.55 * pulse(SNR) + 0.3 * stillness + 0.15 * light. Below 40 the BPM is crossed out.
6. `analyze()` runs every 500 ms. Displayed BPM uses an EMA (0.7 old / 0.3 new) when score >= 40.
7. Test section: user saves readings and types a reference BPM (manual pulse count); stored in localStorage key `rppg-rows`.

## Rules

- All processing stays on device. Never upload video or frames.
- It is a wellness/research prototype, not a medical device. Keep the disclaimer.
- Never label a heart rate as healthy or unhealthy.
- Camera needs https or localhost. Test locally with `python3 -m http.server 8000` and open http://localhost:8000.
- Keep the visual identity: dark green-black, lime `#DAF56B`, Archivo sans with Newsreader italic accents, minimal boxes, GSAP motion that respects `prefers-reduced-motion`.
- User-facing copy: plain and natural, no em dashes, no generic AI phrasing, no "not X but Y" sentences.
- Small, focused commits with clear messages. Explain changes to Aida in plain language; she is learning to work professionally.

## V1 feedback (TKS Slack, Nov 2)

- Ekam: camera turns on but her image does not show, so she can't see where to move. Readings change rapidly; hard to find an accurate one.
- Faith: also saw readings jump around. Wants a short in-app explanation of how it works and the science behind it.
- Aida's own test: within about 2 to 3 bpm of manual pulse counts, but the trust score was too cautious (47 to 55 on readings that were correct). Sitting-still readings varied 60 to 70 bpm between takes.
- Aida promised: usage instructions, a clear explanation of the tech, stable readings. Tag Faith and Ekam in the V2 post.

## V2 roadmap (in order)

1. **Fix the invisible camera preview (bug).** `.idle` sets `display:flex`, which overrides the `hidden` attribute, so the "camera off" overlay never disappears and covers the video. Add `[hidden]{display:none !important}` or toggle a class. Acceptance: live mirrored video visible with the oval and forehead box on top.
2. **Stable readings.** Ideas: median of the last 5 to 7 estimates, reject jumps larger than ~12 bpm unless confirmed by several windows, update the big number once per second, show a "Settling" state until estimates agree, consider longer window (12 to 15 s). Acceptance: while sitting still, displayed BPM varies by no more than about 3 bpm over 30 s.
3. **In-app guidance.** Short 3-step "How to use" next to Start camera, live hints on the video (move closer, more light, hold still), and a compact "The science in 30 seconds" block near the demo. Keep the longer How it works section.
4. **Recalibrate the trust score** so correct, steady readings score in the trustworthy zone. Use logged readings vs reference counts.
5. **Face tracking** so the forehead box follows the face (e.g. MediaPipe Face Landmarker, self-hosted model files).
6. **Cleanup.** Rename leftover "Watch bpm" labels to "Your count" (table aria-label, copy-results header). Consider splitting into `index.html`, `styles.css`, `app.js` once stable.
7. **Ship V2.** Push to GitHub, connect the repo to Vercel for automatic deploys, update the build log on the page with V2 changes.

## V2 plan (three phases, Nov 16)

Goal: turn Pulse Check into a polished V2. Work one phase at a time, show the plan for a phase before writing code, and commit after each item Aida approves.

This plan replaces the order of the V2 roadmap above. Roadmap 1 (camera preview) is done. Roadmap 2 (stable readings) is not: the first attempt (a median stabilizer) still jumped between 78 and 86 bpm while sitting still, so it gets finished together with face tracking in Phase 1 item 1, since the box sitting on the hairline is likely part of the problem. Roadmap 3 (guidance) is mostly covered by Phase 1 item 2, roadmap 4 (trust score) by item 3, and roadmap 5 (face tracking) by item 1. Roadmap 6 and 7 still apply.

### Phase 1: Structure and logic (correct data, easy to use)

1. **Forehead box placement.** In Aida's test the box sat on her hairline, pulse strength was 9 and quality stuck around 50. Add face tracking (MediaPipe Face Landmarker, model files self-hosted in the repo) so the measurement area follows the forehead, and add both cheeks as extra areas. Fall back to the fixed box if tracking fails.
2. **Clear measurement flow.** Replace the endlessly changing number with states the user understands: Ready, Positioning (live hints: move closer, more light, hold still), Measuring (30 second progress ring), Result (final BPM, trust score, and why). Add a "Measure again" button.
3. **Accuracy and trust.** Recalibrate the trust score so steady, correct readings land in the trusted zone. Keep the honest "use with caution" states.
4. **Button states.** Start and Stop must clearly show which is active. Right now Start looks disabled while the camera runs.
5. **Phones and browsers.** Check it works on phone screens and in Safari and Chrome.

### Phase 2: Design (layout, graphics, fonts, colors)

Show Aida 2 design directions (screenshots or descriptions) before applying one.

1. **Better layout.** On smaller screens the camera, number and button are far apart. Put the live BPM and status next to or on top of the camera so everything important is visible at once.
2. **Clear type hierarchy.** Keep Archivo plus Newsreader italic accents, but use fewer label styles and make the main number and result the focus.
3. **Color.** Keep dark green-black and lime `#DAF56B`, and define clear colors for trusted, caution and not trusted.
4. **Shareable result card.** A result card the user can save as an image to share.

### Phase 3: Fun and unique

1. **Hear your heartbeat.** A soft, generated beat sound (Web Audio, no copyrighted music) in sync with the detected pulse, with a mute toggle. A subtle visual pulse on screen at the same rhythm.
2. **Calm challenge.** A 60 second breathing guide where the goal is to lower your heart rate. Show start BPM vs end BPM.
3. **Under the hood view.** A toggle that shows the live forehead color signal, the cleaned pulse wave, and the frequency spectrum with the peak marked, each with one plain sentence explaining that step.

Never label a heart rate as healthy or unhealthy. It is a wellness prototype, not a medical device.
