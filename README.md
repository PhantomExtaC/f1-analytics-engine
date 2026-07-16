# Fanalytics Data Engine

## 🚀 Overview
The Fanalytics Data Engine is a standalone Python-based ETL (Extract, Transform, Load) pipeline. Its purpose is to ingest historical and live Formula 1 data, engineer advanced analytical features, and export static JSON files for the Fanalytics React dashboard.

## 🏗️ Architecture & Data Flow

```text
[Historical DB Dump] + [Live API Updates (Jolpica/FastF1)]
          │
          ▼
1. INGESTION LAYER (data/raw/)
          │
          ▼
2. PRE-PROCESSING & FEATURE ENGINEERING
   ├─ Dynamic Power Ratings (Time-decayed performance)
   ├─ Track Fingerprinting (Pit-stop based tire deg, street vs. perm)
   └─ Driver/Track Mastery Matrix
          │
          ▼
3. ANALYTICS WAREHOUSE (pandas DataFrames)
          │
          ▼
4. EXPORT LAYER (data/export/)
   └─ drivers.json, teams.json, tracks.json, insights.json
          │
          ▼
5. GITHUB ACTIONS (Automated Cron Job)
   └─ Commits JSON files to repo -> Triggers Vercel UI deploy



** ## 📂 Directory Structure**

analytics_engine/
│
├── .github/workflows/
│   └── pipeline.yml          # GitHub Actions cron scheduler
│
├── data/
│   ├── raw/                  # Immutable raw CSVs (Bootstrapped data)
│   └── export/               # Final JSONs ready for the React frontend
│
├── src/
│   ├── ingestion/
│   │   ├── bootstrap.py      # One-time script to load historical DB dump
│   │   └── update.py         # Incremental API fetcher for recent races
│   │
│   ├── features/
│   │   ├── track_profile.py  # Calculates tire deg proxies & circuit types
│   │   ├── power_ratings.py  # Calculates time-decayed team/driver momentum
│   │   └── mastery.py        # Driver strengths/weaknesses per track
│   │
│   └── export/
│       └── json_builder.py   # Formats data to match React TypeScript interfaces
│
├── main.py                   # The Orchestrator (Runs the entire pipeline in order)
├── requirements.txt          # Python dependencies
└── README.md                 # Project documentation


## ⚙️ The GitHub Actions Workflow
The pipeline runs autonomously in the cloud via GitHub Actions.

Trigger: Scheduled for every Monday at 08:00 UTC.

Execution:

Spins up a temporary Ubuntu server.

Installs Python and requirements.txt.

Runs main.py to fetch the weekend's race and update features.

Generates new JSON files in data/export/.

Automatically commits and pushes the changes to the main branch.

## 📊 Engineered Features (Phase 1)
Tire Degradation Index: Calculated by averaging historical pit stops per race at a specific circuit under dry conditions.

Time-Decayed Form: Recent race results are weighted heavier than results from the beginning of the season to accurately track momentum.

Track Suitability: Cross-referencing driver averages on specific circuit profiles (e.g., Street/High-Deg vs. Permanent/Low-Deg).