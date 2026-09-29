# CYBERDECEPTION AI
### Adaptive AI Honeypot and Threat Intelligence Platform

---

## 1. Project Overview
**CyberDeception AI** is an enterprise-grade defensive cybersecurity platform that deploys high-interaction, sandboxed honeypot decoys in a controlled private lab environment. The platform captures incoming reconnaissance and attack traffic, stores granular session telemetry, classifies network flows using machine learning, detects zero-day deviations with unsupervised anomaly detection, analyzes attacker behavior sequences, predicts future moves via empirical Markov transition models, calculates transparent and explainable threat scores, and synthesizes actionable Indicators of Compromise (IOCs) presented through a dark Security Operations Center (SOC) dashboard.

---

## 2. Problem Statement
Traditional perimeter intrusion detection systems (IDS) rely heavily on static signatures and struggle with novel adversarial reconnaissance, credential stuffing, and evasive multi-step lateral movement. Furthermore, standalone honeypots often function as isolated logging silos without integration into automated threat classification or behavioral state prediction. **CyberDeception AI** addresses this gap by integrating:
- Controlled deceptive decoy services.
- Real-time network and session telemetry logging.
- Dual-tier ML: Supervised attack classification and unsupervised anomaly detection.
- Empirical Markov chain behavioral sequence modeling.
- Explainable, transparent threat scoring (0–100).
- Automated threat intelligence and IOC generation.

---

## 3. Project Objectives
1. Provide controlled, safe decoy/honeypot services.
2. Allow authorized security test traffic to safely interact with decoys.
3. Capture network flow counters and session telemetry.
4. Store sessions, events, flows, attacks, threats, and reports in a structured database (MySQL with development fallback).
5. Classify network flows using machine learning trained on the benchmark CIC-IDS2017 dataset.
6. Detect statistical baseline divergence using Isolation Forest anomaly detection.
7. Model attacker behavioral transition sequences from honeypot session events.
8. Predict probable next behavior when sufficient empirical data exists (or return "Insufficient behavioral data").
9. Calculate explainable threat scores (0–100) with itemized contributing factors.
10. Generate structured defensive threat intelligence labeled strictly as *"Project-observed intelligence"*.
11. Present all telemetry and analytics through a professional React SOC dashboard.

---

## 4. Architecture & Data Flow
```
CONTROLLED TEST / LAB TRAFFIC
               │
               ▼
┌──────────────────────────────┐
│     HONEYPOT WEB DECOY       │ (Port 8080 - Sandboxed)
│  (Auth, Recon, Enum, Probes) │
└──────────────┬───────────────┘
               │ Structured Telemetry Stream
               ▼
┌──────────────────────────────┐
│     FASTAPI REST BACKEND     │ (Port 8000)
└──────┬───────────────┬───────┘
       │               │
       ▼               ▼
┌─────────────┐ ┌──────────────────────────────────────┐
│    MYSQL    │ │          MACHINE LEARNING            │
│  DATABASE   │ │                                      │
│  (Port 3306)│ │ • Supervised Classifier (Random Forest)
└─────────────┘ │ • Anomaly Detector (Isolation Forest)│
                │ • Markov Behavior Sequence Model     │
                │ • Explainable Tree Split Attribution │
                └──────────────────┬───────────────────┘
                                   │
                                   ▼
                ┌──────────────────────────────────────┐
                │        THREAT SCORING ENGINE         │
                │        (0–100 Severity Matrix)       │
                └──────────────────┬───────────────────┘
                                   │
                                   ▼
                ┌──────────────────────────────────────┐
                │     PROJECT THREAT INTELLIGENCE      │
                │      (Extracted IOCs & Patterns)     │
                └──────────────────┬───────────────────┘
                                   │
                                   ▼
                ┌──────────────────────────────────────┐
                │          REACT SOC DASHBOARD         │
                │  (Vite + React Router + Recharts)    │
                └──────────────────────────────────────┘
```

---

## 5. Technology Stack
- **Frontend**: React 18, Vite, React Router v6, Axios, Recharts, Lucide React, Modern Dark SOC CSS Design System.
- **Backend**: Python 3.13+, FastAPI, Uvicorn, Pydantic v2, SQLAlchemy 2.0, PyMySQL, Cryptography, python-dotenv.
- **Database**: MySQL 8.0+ (with automatic development fallback).
- **Machine Learning**: scikit-learn, pandas, numpy, joblib.
- **Testing**: pytest, pytest-asyncio, httpx.

---

## 6. Dataset Requirement & Preprocessing Pipeline
The network flow classification pipeline is designed for the official **CIC-IDS2017** dataset:
- **Location**: `backend/ml/data/raw/`
- **Discovery**: Dynamic file scanner detects all `.csv` files, row counts, and headers.
- **Leakage Prevention**: Strictly drops network identifiers (`Flow ID`, `Source IP`, `Source Port`, `Destination IP`, `Destination Port`, `Protocol`, `Timestamp`) to ensure the model learns traffic dynamics rather than memorizing IP addresses.
- **Label Normalization**:
  - `BENIGN` → `Benign`
  - `FTP-Patator`, `SSH-Patator` → `BruteForce`
  - `DoS Hulk`, `DoS GoldenEye`, `DoS slowloris`, `DoS Slowhttptest` → `DoS`
  - `DDoS` → `DDoS`
  - `PortScan` → `PortScan`
  - `Web Attack – Brute Force`, `Web Attack – XSS`, `Web Attack – Sql Injection` → `WebAttack`
  - `Bot` → `Botnet`
  - `Infiltration` → `Infiltration`
  - `Heartbleed` and unmapped labels → `OtherAttack`
- **Partitioning**: Stratified 80% train / 20% test split with StandardScaler fitted exclusively on training data.

---

## 7. Machine Learning Algorithms & Dual-Engine Architecture
### A. Supervised Attack Classifier
- **Model**: `RandomForestClassifier(n_estimators=200, random_state=42, class_weight='balanced', n_jobs=-1)`
- **Comparison Benchmarks**: Evaluated alongside Logistic Regression and Decision Trees.
- **Explainability**: Tree split feature importance and attribution factors.
- **Role**: Answers *"What known attack category does this flow resemble?"*

### B. Unsupervised Anomaly Detector
- **Model**: `IsolationForest(n_estimators=100, contamination=0.05, random_state=42)`
- **Training Baseline**: Trained exclusively on normal/benign network traffic.
- **Role**: Answers *"Does this flow diverge unusually from normal traffic baseline?"*

### C. Behavioral Markov Chain Sequence Model
- **Input**: Honeypot session event transitions (`connection` → `login_attempt` → `auth_failed` → `suspicious_request` → `restricted_access` → `session_end`).
- **Threshold Rule**: Strictly outputs `"Insufficient behavioral data"` when state observations fall below minimum sample thresholds.
- **Crucial Distinction**: CIC-IDS2017 is a network flow dataset, not a command sequence dataset. Behavioral models are trained exclusively on real honeypot session events.

---

## 8. Explainable Threat Scoring Engine
Threat scores range from **0 to 100**, categorized as:
- **0–24**: Low
- **25–49**: Medium
- **50–74**: High
- **75–100**: Critical

**Score Factors**:
- Base attack category weight (e.g. DoS: +75, DDoS: +85, WebAttack: +65, BruteForce: +55).
- Model confidence bonus (scaled up to +15).
- Anomaly detection flag (Isolation Forest deviation: +15).
- Repeated authentication failures (+5 per failure up to +20).
- Suspicious request & restricted resource probing (+10 each up to +25).
- Session persistence and sustained interaction (+5).

---

## 9. Honeypot Decoy Architecture & Safety Guardrails
- **Service**: `honeypots/web_decoy/app.py` running on `127.0.0.1:8080`.
- **Endpoints**:
  - `/`: Simulated Corporate SSO Portal.
  - `/login`: Realistic credential verification failure simulator.
  - `/admin`, `/.env`, `/wp-admin`, `/actuator`: Unauthorized reconnaissance capture.
  - `/api/v1/query`: Simulated injection probe capture.
  - `/logout`: Controlled session termination.
- **Safety Invariants**:
  - Enforces local/private bindings only (`127.0.0.1` or RFC1918 subnets).
  - Public IP bindings are rejected automatically by API validation.
  - Decoys are sandboxed; no host shell execution or real file reads occur.
  - No real credentials or secrets are accepted or logged.

---

## 10. Database Schema (MySQL 8.0+)
Defined in `database/schema.sql`:
1. `users`: Analysts and operators with role-based access.
2. `honeypots`: Registered decoy services, bound interfaces, and status.
3. `sessions`: Honeypot sessions tracking timestamps and threat scores.
4. `session_events`: Chronological events (`login_attempt`, `auth_failed`, etc.).
5. `network_flows`: Captured flow features compatible with CIC-IDS2017.
6. `attacks`: Classified attack incidents with model confidence.
7. `threats`: Correlated threat entries with itemized scoring reasons.
8. `threat_indicators`: Extracted indicators of compromise (IOCs).
9. `ai_predictions`: Inferences, top-k class distributions, and explanations.
10. `model_versions`: Auditable registry of weights, metrics, and lineage.
11. `reports`: Executive SOC and defensive audit reports.

---

## 11. REST API Specification
| Method | Endpoint | Description |
|---|---|---|
| `GET` | `/api/health` | System health, DB connection, and model readiness status |
| `GET` | `/api/dashboard/summary` | Real-time SOC metrics, velocity charts, and attack stream |
| `GET` | `/api/honeypots` | List registered honeypot decoys with session counts |
| `POST`| `/api/honeypots` | Register new controlled honeypot decoy |
| `PUT` | `/api/honeypots/{id}` | Update honeypot status (activate/pause) |
| `DELETE` | `/api/honeypots/{id}` | Delete honeypot decoy |
| `GET` | `/api/sessions` | List decoy sessions with duration and threat scores |
| `GET` | `/api/sessions/{id}` | Full session detail with event history |
| `GET` | `/api/sessions/{id}/events` | Chronological session event timeline |
| `POST`| `/api/sessions/{id}/events` | Telemetry intake from decoy services |
| `GET` | `/api/attacks` | Filtered attack stream (by type, severity, decoy) |
| `GET` | `/api/attacks/{id}` | In-depth attack investigation, explanations, and actions |
| `POST`| `/api/attacks` | Record classified attack event |
| `GET` | `/api/threats` | Correlated threat incidents |
| `GET` | `/api/threats/intelligence` | Structured defensive threat intelligence feed |
| `GET` | `/api/ml/status` | Model operational readiness and feature counts |
| `GET` | `/api/ml/model-info` | Verified model metrics, hyperparameters, and lineage |
| `POST`| `/api/ml/predict` | Supervised flow classification with explanation |
| `POST`| `/api/ml/anomaly` | Isolation Forest unsupervised anomaly evaluation |
| `GET` | `/api/ml/explain/{id}` | Tree split feature importance and explanation |
| `GET` | `/api/behavior/{session_id}` | Session event frequencies and repeated action analysis |
| `GET` | `/api/behavior/{session_id}/next`| Markov next-state behavior prediction |
| `GET` | `/api/reports` | List generated incident and security audit reports |
| `POST`| `/api/reports` | Compile executive audit report |

---

## 12. Frontend Pages (React SOC Dashboard)
1. **SOC Dashboard (`/`)**: High-level velocity charts, threat distributions, active decoy count, and live attack stream.
2. **Decoy Services (`/honeypots`)**: Management of local decoys with safety status indicators and registration modal.
3. **Attack Monitoring (`/attacks`)**: Searchable and filterable attack log with deep-dive investigation modal.
4. **Decoy Sessions (`/sessions`)**: Interactive chronological timeline of attacker probe steps and Markov transition probabilities.
5. **Threat Intelligence (`/threats`)**: Project-observed indicators of compromise (IOCs), affected assets, and actionable recommendations.
6. **AI Analysis (`/ai-analysis`)**: Dual-engine metrics, feature importances, and an interactive real-time inference sandbox.
7. **Audit Reports (`/reports`)**: Incident report compiler with printable formatting.
8. **Settings (`/settings`)**: Health diagnostics, active dialect status, and sandboxing invariant verifications.

---

## 13. Installation & Setup

### Prerequisites
- Python 3.10+ (Verified on Python 3.13)
- Node.js 18+ (Verified on Node.js v24)
- MySQL 8.0+ (Optional: SQLite fallback is active automatically)

### Step 1: Clone & Configure Environment
```bash
cp .env.example .env
```
Edit `.env` to supply your MySQL password if connecting to local MySQL:
```ini
MYSQL_HOST=localhost
MYSQL_PORT=3306
MYSQL_USER=root
MYSQL_PASSWORD=YOUR_PASSWORD_HERE
MYSQL_DATABASE=cyberdeception
```

### Step 2: Database Setup (MySQL)
Run the SQL schema in your MySQL instance:
```bash
mysql -u root -p < database/schema.sql
mysql -u root -p < database/seed.sql
```

### Step 3: Run FastAPI Backend
```bash
cd backend
.\venv\Scripts\uvicorn.exe app.main:app --host 127.0.0.1 --port 8000 --reload
```
Swagger API documentation will be available at: `http://127.0.0.1:8000/docs`.

### Step 4: Run React SOC Dashboard
```bash
cd frontend
npm run dev
```
Open `http://localhost:5173` in your browser.

### Step 5: Run Honeypot Web Decoy
```bash
cd honeypots/web_decoy
& "..\..\backend\venv\Scripts\python.exe" app.py
```
Decoy login portal will be available at: `http://127.0.0.1:8080`.

---

## 14. Running Machine Learning Training
To train the supervised classifier and Isolation Forest on the raw CIC-IDS2017 dataset:
1. Place the official CIC-IDS2017 CSV files in `backend/ml/data/raw/`.
2. Run the training script:
```bash
cd backend
.\venv\Scripts\python.exe ml\train.py
```
3. The script automatically benchmarks Logistic Regression, Decision Trees, and Random Forests, trains the Isolation Forest baseline, and saves the verified model artifacts to `backend/ml/models/`.

---

## 15. Running Automated Tests
Run all unit and regression tests:
```bash
cd backend
.\venv\Scripts\pytest.exe -v
```

Run the end-to-end controlled lab integration test:
```bash
& "backend\venv\Scripts\python.exe" tests\integration_test.py
```

---

## 16. Security Restrictions & Ethical Notice
- This platform is designed exclusively for authorized defensive cybersecurity research, academic demonstration, and educational lab simulations.
- Do not deploy decoys to public internet IP ranges without authorization.
- Do not connect decoys to real internal production networks or confidential databases.
