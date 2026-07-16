# 🏎️ Fanalytics Data Engine

[![Pipeline Status](https://shields.io)](https://github.com)
[![Python](https://shields.io)](https://python.org)
[![Tech Stack](https://shields.io|%20FastF1%20|%20Jolpica-FF1801.svg?style=flat-square)](https://github.com)

A high-performance, standalone Python ETL (Extract, Transform, Load) pipeline. It ingests historical and live Formula 1 data, engineers advanced analytical features, and exports optimized static JSON assets to power the **Fanalytics React Dashboard**.

---

## 🏗️ Architecture & Data Flow

```mermaid
graph TD
    A[Historical DB Dump] & B[Live API Updates: Jolpica/FastF1] --> C[1. Ingestion Layer <br><i>data/raw/</i>]
    C --> D[2. Pre-Processing & Feature Engineering]
    D --> D1[Dynamic Power Ratings <br><i>Time-decayed performance</i>]
    D --> D2[Track Fingerprinting <br><i>Tire deg, street vs. permanent</i>]
    D --> D3[Mastery Matrix <br><i>Driver/Track affinity</i>]
    D1 & D2 & D3 --> E[3. Analytics Warehouse <br><i>pandas DataFrames</i>]
    E --> F[4. Export Layer <br><i>data/export/</i>]
    F --> F1[drivers.json, teams.json, tracks.json, insights.json]
    F1 --> G[5. GitHub Actions <br><i>Automated Cron Job</i>]
    G --> H[Commit JSONs to Repo] --> I[Trigger Vercel UI Deploy]

    style A fill:#2d3748,stroke:#4a5568,stroke-width:1px,color:#fff
    style B fill:#2d3748,stroke:#4a5568,stroke-width:1px,color:#fff
    style C fill:#1a365d,stroke:#2b6cb0,stroke-width:2px,color:#fff
    style D fill:#2c5282,stroke:#4299e1,stroke-width:2px,color:#fff
    style E fill:#234e52,stroke:#319795,stroke-width:2px,color:#fff
    style F fill:#744210,stroke:#b7791f,stroke-width:2px,color:#fff
    style G fill:#742a2a,stroke:#e53e3e,stroke-width:2px,color:#fff
```

---

## 📂 Directory Structure

```text
analytics_engine/
│
├── .github/workflows/
│   └── pipeline.yml          # GitHub Actions CI/CD cron scheduler
│
├── data/
│   ├── raw/                  # Immutable raw CSVs & bootstrapped assets
│   └── export/               # Production-ready JSONs for React frontend
│
├── src/
│   ├── ingestion/
│   │   ├── bootstrap.py      # One-time historical database seeder
│   │   └── update.py         # Incremental live API fetcher for recent GPs
│   │
│   ├── features/
│   │   ├── track_profile.py  # Tire degradation proxies & circuit typing
│   │   ├── power_ratings.py  # Time-decayed team & driver momentum engines
│   │   └── mastery.py        # Driver strengths/weaknesses matrix per track
│   │
│   └── export/
│       └── json_builder.py   # Formats payloads to match React TypeScript interfaces
│
├── main.py                   # Global Orchestrator (Executes full ETL sequence)
├── requirements.txt          # Python runtime dependencies
└── README.md                 # Project documentation
```

---

## 📊 Engineered Features (Phase 1)

*   **Tire Degradation Index** 
    Calculates dynamic degradation proxies by aggregating and filtering historical pit-stop windows exclusively under dry racing conditions.
*   **Time-Decayed Form Factor** 
    Applies an exponential time-decay algorithm to recent Grand Prix placements. Recent weekend forms are weighted heavier than early-season points to accurately map current team/driver momentum.
*   **Track Suitability Matrix** 
    Cross-references a driver's historical scoring metrics against specific structural track profiles (e.g., *High-Degradation Street Circuits* vs. *Low-Degradation Permanent Tracks*).

---

## ⚙️ Automated GitHub Actions Workflow

The engine operates entirely hands-free in the cloud via automated infrastructure.

```yaml
Trigger: 🕒 Every Monday at 08:00 UTC (Post-Race Weekend)
```

1.  **Environment Provisioning:** Spins up an isolated, ephemeral Ubuntu container.
2.  **Dependency Setup:** Installs Python environment and cached `requirements.txt` targets.
3.  **Pipeline Orchestration:** Executes `main.py` to target, fetch, and process the latest race weekend data.
4.  **Asset Generation:** Overwrites local feature states and rebuilds static data in `data/export/`.
5.  **Autonomous Deployment:** Automatically signs, commits, and pushes modified tracking JSONs back to `main`, instantly triggering your frontend Vercel deployment hook.
