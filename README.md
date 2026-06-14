# Drilling Force Prediction for Ti-6Al-4V Titanium

Machine-learning prediction and minimisation of CNC **drilling cutting forces (Fx, Fy, Fz)** on
Ti-6Al-4V titanium, modelled from the cutting parameters and validated entirely on the machine's own
instrumented experiments. Includes an interactive in-browser predictor and full analysis reports.

**Live app:** https://OnlyMrUsman.github.io/ti-drilling-force/

---

## Problem

Drilling titanium generates large thrust forces that can damage the workpiece. The goal is to (1) **predict**
the cutting forces from the process parameters and (2) **find the parameter settings that minimise** them,
so the process can be planned before cutting.

- **Inputs:** coolant strategy, cutting speed `Vc`, pecking on/off, feed.
- **Outputs:** thrust force `Fz` (dominant) and radial forces `Fx`, `Fy`.
- **Workpiece:** Ti-6Al-4V, 22 mm thickness.

## Data

A 16-run **designed experiment** (4 coolants × 2 cutting speeds × 2 pecking states): Dry / MQL / CO₂ / LN₂.
Each run records the peak thrust force; the four dry runs also have full high-rate Kistler dynamometer
signals (Fx/Fy/Fz at 5050 Hz, 3 holes each), used to extract all three forces and the cutting timing.

## What this project does

- **Thrust predictor (all coolants):** predicts peak `Fz` from coolant, speed and pecking.
- **Dry full-force analysis:** measured `Fx`, `Fy`, `Fz` with uncertainty bands, plus a timing
  reconstruction (approach time vs. active cutting, and the peck-retraction gaps visible in the signal).
- **Force minimisation:** ranks the parameters by their effect on force.
- **Interactive web app:** a two-tab tool that runs the trained model in the browser (no server).
- **Reports:** a full report and a focused dry-drilling report.

## Key results

| Question | Result |
|---|---|
| Predict thrust `Fz`? | Yes — ~6% leave-one-out error (≈55 N) across all coolants; ~4.4% on the dry block |
| Predict radial `Fx`, `Fy`? | Yes, with larger relative error (~16–20%) — they are small (~50 N) and noisy |
| Lowest-force settings? | **CO₂ or MQL** cooling and **no pecking**; cryogenic **LN₂** gives the highest force |
| Reproducible? | Forces repeat to ~4% hole-to-hole, with a slow upward drift from tool wear |

## Method

Forces are extracted from the raw signals (engaged-region peak and mean). Six model families
(linear, ridge, decision tree, random forest, gradient boosting, Gaussian process) are compared under
**leave-one-out cross-validation** — the honest validation choice at this sample size. The final model
is a random forest exported to JSON and run client-side in the web app.

## Repository structure
ti-drilling-force/
├── notebooks/ # full reproducible analysis (Part A: all coolants, Part B: dry deep-dive)
├── src/ # web-app template + build script + report generators
├── webapp/ # built two-tab predictor (index.html) + model.json
├── docs/ # GitHub Pages copy of the app
├── figures/ # generated charts
├── report/ # Word reports
└── data/ # raw experiment files go in data/raw/ (not committed)
## Reproduce

```bash
uv venv && source .venv/bin/activate
uv pip install -r requirements.txt
# place the raw experiment .xlsx files in data/raw/, then run the notebook top-to-bottom:
jupyter lab        # run notebooks/drilling_force_prediction.ipynb
python src/build_webapp.py     # -> webapp/index.html
```

## Tech stack

Python · scikit-learn · openpyxl · pandas · matplotlib · Jupyter · vanilla-JS web app · docx.js ·
GitHub Pages · uv

## Honesty & limitations

The model is trained on a 16-run designed experiment (full 3-axis signals for the 4 dry runs). No model can
be "100% accurate": repeated holes at identical settings produce slightly different forces, and force drifts
as the tool wears. The model is reliable for **process planning and setup estimation**; for production use it
should run alongside live force monitoring and be re-checked as the drill ages.
