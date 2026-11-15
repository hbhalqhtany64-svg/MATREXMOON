MOONMATREX TOOL
**NASA Space Apps Challenge 2026 – Tabuk team · Challenge: Lunar Landing Site Navigator**

## Summary
An interactive web tool to compare lunar landing sites and time windows. Click any point on the Moon to see the nearest past mission, terrain elevation and slope (NASA LRO/LOLA), estimated power needs, illumination, shadow, temperature, Earth-communication status and dust level. A date/time selector moves the Sun across the surface.

## Run
    pip install -r requirements.txt
    python backend/app.py      # open http://localhost:5000
Without the server, open `index.html` directly (or host it on GitHub Pages): the interface falls back to local calculation.

## API
- `GET /api/missions`
- `GET /api/site?lat=-69.37&lon=32.32&sun=0`  (`sun` = sub-solar longitude, degrees)

## NASA data used
- **LRO LOLA LDEM_4** (elevation; slope derived) – https://imbrium.mit.edu/DATA/LOLA_GDR/CYLINDRICAL/IMG/ · rebuild: `python tools/build_terrain.py LDEM_4.IMG backend/terrain.json`
- References: LRO QuickMap, NASA Moon Trek, JPL Horizons, CSA Moon Exploration.
- Illumination, temperature, dust, communication and power values are approximate models, not NASA measurements. Slope is regional (1° grid, ~15 km baseline).

## Use of AI
Built with help from Claude (Anthropic) for code and text.

## Team
[Add team member names]

## Machine learning
`tools/train_suitability.py` trains a logistic regression on NASA LOLA elevation, slope, |latitude| and Earth line-of-sight. The 11 past soft-landing sites are the positives; the rest of the Moon is unlabeled background. The output is a **0–100 percentile of similarity to past landing sites** (data panel, map layer, ML panel and ML Lab).
- **Validation:** leave-one-site-out (hold one site out, rank it among all 64,800 cells). Mean percentile **76** (≈ AUC 0.76, random = 50), median 88.8. It works for near-side equatorial sites but **fails for the far side and poles** (Chang'e 4: 5th, Chandrayaan-3: 27th percentile). See `data/ml_report.json`.
- **Not** a safety or mission-success predictor: only 11 examples, selection bias, no failed-landing labels.
- **ML panel and ML Lab** (in `index.html`): score gauge, per-feature contribution bars, validation chart, model card, and a full-screen lab with a scan animation and per-feature contribution maps. The coefficients are hard-coded in the `MLM` constant; regenerate them with `python tools/export_ml_coefficients.py backend/terrain.json` if you retrain.

## Project layout
    index.html            frontend (works standalone, falls back to local calculation)
    backend/app.py        Flask API + SQLite   |  backend/terrain.json  LOLA 1° grid + ML scores
    tools/                build_terrain.py, train_suitability.py, export_ml_coefficients.py
    data/                 ml_report.json, LDEM_4.LBL
    assets/               put nasa-logo.png here (official, unmodified)
