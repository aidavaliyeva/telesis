# Telesis

**Operator-state sensing for safe teleoperation — contactless, in the browser.**

> Before a human takes control of a robot, how do we know they're physically
> ready? Telesis reads an operator's heart rate from their face through the same
> camera that's already in the control loop — no wearable — and flags when the
> operator, or the reading itself, can't be trusted.

> **[ Screenshot placeholder ]** Add a screenshot of the locked-reading state as
> `docs/screenshot.png` and replace this block with
> `![Telesis showing a locked heart-rate reading](docs/screenshot.png)`.

<!-- Live demo: add after deploy -->

## What it is
Telesis is a research prototype built for The Knowledge Society (TKS) rPPG Focus.
It uses remote photoplethysmography (rPPG) to measure heart rate from subtle
colour changes in facial skin, then turns that into an **operator-state verdict**
— a go/no-go signal for whether a human is in a fit state to take control of a
teleoperated robot.

## Why it matters
Teleoperation puts a human in direct control of a physical robot, often in
safety-critical settings — remote surgery, hazardous environments, logistics. A
fatigued or distracted operator is a real risk. Telesis explores sensing operator
readiness continuously and contactlessly, as a prerequisite for trusting a
human-in-the-loop system.

## How it works
1. Face tracking locates the forehead and cheeks as regions of interest.
2. The average red, green and blue values in those regions are sampled each frame.
3. The POS algorithm (Wang et al., 2017) separates the pulse signal from lighting
   changes.
4. The strongest frequency peak between 42–210 bpm gives the heart rate.
5. Measurement session: once the operator is in position and the signal is good, a
   10-second hold is captured and the reading is locked as the median of trusted
   estimates, rejecting unstable windows.
6. An operator-state verdict (Ready / Caution / Not recommended) is derived from
   the heart rate and its stability, gated by a trust score so an unreliable
   reading never produces a verdict.

## Honesty by design
Telesis shows a trust score and four signal sub-metrics, and refuses to show a
confident number when the signal is poor — a confident wrong reading is worse than
none. The operator-state verdict is a heuristic proxy from heart rate and its
stability, **not a clinical fatigue measure**. Everything runs on-device; no video
leaves the browser.

## Run locally
```bash
git clone https://github.com/aidavaliyeva/telesis.git
cd telesis
python3 -m http.server 8000
# open http://localhost:8000
```
Requires a webcam and good lighting.

## Tech stack
| Area | Tool |
| --- | --- |
| Signal processing | rPPG + POS algorithm (vanilla JavaScript) |
| Face tracking | MediaPipe Face Landmarker (`@mediapipe/tasks-vision` 1.1.1, self-hosted) |
| Frontend | HTML, CSS, JavaScript (no framework) |
| Persistence | localStorage |
| Hosting | Vercel |

## Limitations
- rPPG accuracy depends on lighting, skin tone and stillness; tested so far on a
  narrow range.
- Heart rate alone is not fatigue; true operator-state detection needs HRV and
  per-person calibration.
- The manual pulse-count comparison is itself imprecise.

## Roadmap
- **Iterate:** automatic detection of talking, head movement and poor lighting.
- **Create:** a teleoperation demo environment; HRV-based fatigue detection;
  per-operator calibration via Hemio biomarker data.

## About
Built by [Aida Valiyeva](https://aidavaliyeva.site) for the TKS rPPG Focus, as
part of a broader interest in teleoperation and health sensing.

## License
MIT
