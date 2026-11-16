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
