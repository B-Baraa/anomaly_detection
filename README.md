# Anomaly Detection on Real Sensor Data

Mining real machine-failure patterns from sensor readings using [PyOD](https://github.com/yzhao062/pyod). Built as a complete, presentable data mining project — no deployment required.

## What this is

Most anomaly-detection demos use synthetic data with perfect, made-up labels. This project uses **real hourly ambient-temperature readings from a machine room**, sourced from the [Numenta Anomaly Benchmark (NAB)](https://github.com/numenta/NAB) — a public research benchmark — along with NAB's **officially documented ground-truth failure windows**. The goal: detect real equipment failures using unsupervised anomaly detection, and check the results against what actually happened.

## Dataset

| | |
|---|---|
| **Source** | Numenta Anomaly Benchmark (NAB) |
| **File** | `realKnownCause/ambient_temperature_system_failure.csv` |
| **Records** | 7,267 hourly readings |
| **Time span** | July 4, 2013 – May 28, 2014 (~10 months) |
| **Documented real failures** | 2 windows (Dec 15–30, 2013 and Mar 29–Apr 20, 2014) |

Both the sensor readings (`data/ambient_temperature_system_failure.csv`) and the ground-truth labels (`data/combined_windows.json`) are bundled in this repo, so the notebook runs fully offline.

## Pipeline

1. **Load** the real sensor readings and the NAB ground-truth failure windows
2. **Engineer features** — rolling mean, rolling standard deviation, and hour-over-hour difference (24-hour window), so the model sees deviations from *recent* normal behavior, not just the raw value
3. **Train detectors** — three unsupervised PyOD models, each mining outliers differently:
   - **ECOD** — empirical cumulative distribution, no hyperparameters
   - **Isolation Forest** — isolates points via random splits
   - **KNN** — flags points far from their nearest neighbors
4. **Evaluate** each detector against the real labeled failure windows using ROC-AUC
5. **Visualize** the best detector's flagged points overlaid on the real time series next to the true failure windows

## Results

| Detector | ROC-AUC |
|---|---|
| **ECOD** | **0.815** |
| Isolation Forest | 0.758 |
| KNN | 0.647 |

ECOD performs best and requires no tuning. Its flagged points visibly cluster inside the two real, documented failure windows — the model is catching genuine equipment degradation, not noise.

## Project structure

## How to run

```bash
pip install -r requirements.txt
jupyter notebook real_anomaly_detection_demo.ipynb
```
(On Windows, if `jupyter` isn't on your PATH, use `python -m notebook real_anomaly_detection_demo.ipynb` instead.)

Then **Run All Cells**. Everything — data loading, feature engineering, model training, evaluation, and plots — runs top to bottom with no external downloads needed.

## Why this matters

Undetected equipment failures are expensive: by the time a threshold alarm fires, downtime has usually already started. Statistical anomalies in sensor data often appear hours-to-days before a hard failure is visible to a human, so catching them early turns unplanned outages into scheduled maintenance. Because the detectors here are unsupervised, this approach works even without a history of labeled past failures — the normal case for most real facilities. The same 5-step pipeline generalizes directly to other sensor types: vibration, pressure, network traffic, transaction logs, and more.

## Possible extensions

- Test the pipeline on other real NAB datasets (network traffic, CPU utilization, taxi demand)
- Add weekly/seasonal features to catch slower-building drift
- Ensemble multiple detectors into a single combined anomaly score
- Wire the pipeline into a live alerting system for continuous monitoring

## Credits

- Dataset & ground truth: [Numenta Anomaly Benchmark (NAB)](https://github.com/numenta/NAB)
- Detection library: [PyOD](https://github.com/yzhao062/pyod)
