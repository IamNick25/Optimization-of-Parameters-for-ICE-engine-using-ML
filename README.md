# Engine Log Analysis & Forecasting — *ICE.ipynb*

A simple, end‑to‑end notebook to **load engine CSV logs**, **clean and explore the data**, and **train a small N‑BEATS time‑series model** (TensorFlow/Keras) to predict **ignition timing** from recent history and related signals.

This README explains what the notebook does and how you can run it in **Google Colab** or **local Jupyter** using plain, simple English.

---

## What this notebook does

1. **Reads multiple CSV log files** from a folder (default pattern: `.../EngineData/*.csv`).
   - It parses the header lines (until the line starting with `Log :`) to auto‑name columns like `Channel` and `Type`.
   - Builds a single table with a proper **timestamp**, **elapsed time** (`#time_seq`), and a **route** name inferred from each filename.
2. **Visualizes and cleans data**  
   - Basic scatter/time plots (e.g. engine **Load** vs **RPM**, colored by **AFR**).  
   - Drops minute‑sized intervals with too many missing values (default threshold **40%** NaN).  
   - Fills remaining gaps with linear interpolation + forward/backward fill.
3. **Feature preparation**  
   - Selects key signals (for example: `AirTemp[Temperature]`, `AirTemp_K`, `IgnitionTiming[Angle]`, `IgnitionTiming[Angle]_Final`, `LambdaSensor1[AFR]`, `Load[Pressure]`, `Load[Pressure]_Final`, `MAPSource[Pressure]`, `RPM[EngineSpeed]`, `RPM[EngineSpeed]_Final`, `TargetAFR[AFR]`).  
   - Scales features (Min‑Max scaler) and optionally inspects variance with **PCA**.
4. **Builds a small N‑BEATS forecaster** (TensorFlow/Keras)  
   - Window size (lookback) and horizon are kept small by default (e.g. 5 → predict next 1 step).  
   - Target column is **`IgnitionTiming[Angle]`**.  
   - Uses **Optuna** to search a few hyperparameters (stacks/layers/neurons/learning rate).
5. **Evaluates and plots predictions**  
   - Compares **predicted vs. true** ignition timing and shows simple scatter/line plots.

> **Note:** The notebook doesn’t save models or figures by default; it focuses on a clean, readable workflow.

---

## Requirements

Tested in **Google Colab**. For local runs you’ll need:

- Python 3.9+
- Packages: `pandas`, `numpy`, `matplotlib`, `seaborn`, `scikit-learn`, `tensorflow (2.x)`, `optuna`, `yaspin`, `IPython`

Quick install (local):
```bash
pip install pandas numpy matplotlib seaborn scikit-learn tensorflow optuna yaspin ipython
```

---

## Data format & folder

- Place your CSV logs in a folder, e.g. `EngineData/`, and point the notebook to:  
  `filepath = "/content/drive/MyDrive/EngineData/*.csv"` (Colab default).
- Each CSV is expected to have a short **header** with lines like:
  - `Channel : <name>`
  - `Type : <unit>`
  - and later a line `Log :` before the actual data rows.
- Filenames are used to infer `date` and `route` (split by `-` before `.csv`).

**Time format:** `YYYYMMDD HH:MM:SS.ssssss`

---

## How to run (Colab)

1. Open **Google Colab** and upload `ICE.ipynb`.
2. If your data lives on Google Drive, mount it and set:
   ```python
   filepath = "/content/drive/MyDrive/EngineData/*.csv"
   ```
3. Run the notebook **top to bottom**:
   - Install cells (`pip install ...`) if Colab didn’t auto‑install.
   - Data loading and cleaning cells.
   - Feature prep (scaling/PCA) cells.
   - N‑BEATS model build + Optuna tuning cells.
   - Evaluation/plot cells.

You should see tables, progress logs, and plots inline.

---

## How to run (local Jupyter)

1. Create/activate a Python environment and install the **Requirements** above.
2. Put your CSV files in a folder (e.g. `data/EngineData/`) and change:
   ```python
   filepath = "data/EngineData/*.csv"
   ```
3. Start Jupyter and run all cells in order. Plots will appear inline.

---

## Change what the model predicts

- Default **target**: `IgnitionTiming[Angle]` (set in the cell where `TARGET_COL = ...`).  
- To predict a different signal, change `TARGET_COL` and re‑run the **feature prep** and **model** cells.

**Lookback / horizon**: edit `WINDOW_SIZE` and `HORIZON` near the N‑BEATS section.

---

## Assumptions & notes

- CSVs follow the same header/data layout and share common columns.
- Route handling: if a route equals `mimos2home`, a small reversal is applied to a sequence column to keep direction consistent.
- Interpolation assumes short gaps; very sparse logs may need stronger filtering.
- This is a **teaching/experiment** notebook: the N‑BEATS model is intentionally small and may not be optimal for production.

---

## Outputs

- Inline figures: scatter plots and prediction vs. truth comparisons.
- Printed metrics from the evaluation step.
- No files are written unless you add your own `to_csv()` / `savefig()` / `model.save()`.

---

## Troubleshooting

- **No files found**: check `filepath` glob and Drive mount.
- **Missing columns**: verify your CSV headers/units; adjust selected feature names in the “Factors to analyze” cell.
- **TensorFlow errors on CPU‑only machines**: ensure you installed a **TensorFlow 2.x** CPU build and restarted the kernel.

---
