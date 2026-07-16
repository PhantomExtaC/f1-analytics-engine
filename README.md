```markdown
# 🏎️ Fanalytics Data Engine

![Python](https://img.shields.io/badge/Python-3.11-blue?logo=python)
![Pandas](https://img.shields.io/badge/Pandas-Data_Engineering-150458?logo=pandas)
![FastF1](https://img.shields.io/badge/FastF1-Jolpica_API-red)
![GitHub Actions](https://img.shields.io/badge/Automated-Cron_Job-2088FF?logo=github-actions)

## 🚀 Overview
The Fanalytics Data Engine is a standalone, automated Python ETL (Extract, Transform, Load) pipeline. It acts as the algorithmic "brain" behind the Fanalytics React dashboard. 

Instead of relying on basic API queries that just list static point totals, this engine processes deep historical Formula 1 data spanning two decades. It dynamically engineers complex predictive metrics (like Driver Momentum, Racecraft Indices, and Track Degradation Scales) and exports polished, lightweight JSON models for the React frontend to serve directly.

---

## 🏗️ Architecture & Data Flow

The pipeline executes in a strict chronological sequence, turning raw relational database inputs into highly structured features.

```text
[Historical Kaggle DB Dump (2006-2023)] + [Live API Updates via FastF1/Jolpica]
          │
          ▼
1. INGESTION LAYER (data/raw/)
          │
          ▼
2. FEATURE FACTORY (pandas DataFrames)
   ├─ Dynamic Power Ratings (Time-decayed performance momentum)
   ├─ Track Fingerprinting (Pit-stop based tire deg, street vs. perm)
   └─ Driver/Track Mastery Matrix (Hierarchical track suitability)
          │
          ▼
3. EXPORT LAYER (data/export/)
   └─ insights.json, tracks.json (Ready for React TypeScript Interfaces)
          │
          ▼
4. CLOUD AUTOMATION (GitHub Actions)
   └─ Commits JSON files to repo -> Triggers Vercel UI deploy

```

---

## 📂 Directory Structure

```text
analytics_engine/
│
├── .github/
│   └── workflows/
│       └── analytics_pipeline.yml   # Cloud Automation script (runs every Monday)
│
├── data/
│   ├── raw/                         # Immutable CSVs (Historical Lake)
│   ├── cache/                       # FastF1 SQLite local caching (prevents API rate limits)
│   ├── features/                    # Engineered ML feature metrics
│   └── export/                      # Final structured JSONs for the React UI
│
├── src/
│   ├── ingestion/
│   │   ├── bootstrap.py             # One-time script: Processes the 2006-2023 Kaggle DB Dump
│   │   ├── bridge_seasons.py        # Connects the Kaggle dump to current day via Jolpica API
│   │   └── update.py                # Weekly surgical script: Fetches only the newest race
│   │
│   ├── features/
│   │   ├── track_profile.py         # Generates Tire Deg proxies & identifies Street Circuits
│   │   ├── power_ratings.py         # Calculates Team/Driver Momentum via Exponential Averages
│   │   └── mastery.py               # Evaluates driver strengths against Track Fingerprints
│   │
│   └── export/
│       └── json_builder.py          # Formats analytical tables into clean JSON objects
│
├── main.py                          # The Orchestrator (Executes the pipeline sequentially)
├── requirements.txt                 # Python dependencies (pandas, fastf1)
└── README.md                        # Documentation

```

---

## 🧠 Engineered Features (The Analytics Core)

The Fanalytics Engine extracts context-aware signals from raw historical logs to lay the foundation for advanced machine learning model training:

### 1. Dynamic Power Ratings (`power_ratings.py`)

Formula 1 performance is highly fluid. We use an **Exponential Moving Average (EMA)** spanning the last 10 races to compute team and driver "Current Form."

* **Data Leakage Prevention:** All moving calculations incorporate a `.shift(1)` step. This guarantees that a competitor's momentum rating strictly reflects what was known *going into* Sunday morning, isolating it completely from that afternoon's race results.

### 2. Track Fingerprinting (`track_profile.py`)

* **Tire Degradation Index:** Real-time tire compounds and wear metrics are heavily restricted. As a proxy, this script analyzes decades of historical pit stop frequencies under dry race conditions. Using Quantile Cuts (`pd.qcut`), tracks are mathematically bucketed into a standardized 1–5 structural degradation scale.
* **Circuit Layout Typology:** Circuits are rigidly mapped as Street Circuit (1) or Permanent Road Course (0) to dictate handling, downforce, and qualifying-weight contexts.

### 3. Hierarchical Driver Mastery (`mastery.py`)

This measures how effectively a driver's style matches a circuit's architecture. To prevent breaking the system when a driver encounters a track for the first time (e.g., a rookie racing at Las Vegas), the engine utilizes a custom **Hierarchical Imputation** fallback chain:

```text
Track-Specific History Average ──► Layout Average (e.g., Street Performance) ──► General Career Average ──► Grid Midpoint (10.5)

```

---

## ⚙️ Cloud Automation (GitHub Actions)

The backend pipeline operates completely hands-free in the cloud:

* **The Trigger:** A GitHub Actions cron scheduler set for `0 8 * * 1` (Every Monday at 08:00 UTC).
* **The Execution:** Spins up an isolated Ubuntu runner, mounts Python 3.11, restores dependencies, and handles incremental data aggregation via `main.py`.
* **The Output:** A deployment bot verifies if new telemetry rows were pushed. If a race concluded over the weekend, the bot automatically signs, commits, and pushes the new data chunks back to the repository, instantly causing your live Vercel dashboard UI to refresh.

---

## 🛠️ Local Setup & Usage

To execute or develop the pipeline locally on your machine:

### 1. Install Dependencies

```bash
cd analytics_engine
pip install -r requirements.txt

```

### 2. Hydrate the Data Lake (First Time Only)

Ensure the initial open-source Ergast CSV base tables (`races.csv`, `results.csv`, `drivers.csv`, `constructors.csv`, `pit_stops.csv`) have been extracted into your `data/raw/` directory. Then execute the setup chain:

```bash
python src/ingestion/bootstrap.py
python src/ingestion/bridge_seasons.py

```

### 3. Run the Production Pipeline

To run an incremental check, rebuild your feature engines, and re-export the React JSON schema files, simply execute the orchestrator root file:

```bash
python main.py

```

```

```