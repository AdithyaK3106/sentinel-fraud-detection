# SENTINEL — Real-Time Fraud Response System

A hackathon-built intelligent fraud detection and investigation platform featuring real-time transaction analysis, dynamic case management, interactive fraud chain visualization, and an investigative action engine.

---

## Overview

SENTINEL processes live transaction streams, applies hybrid rule-based and ML-guided risk scoring, automatically links related transactions into investigation cases, builds interactive fraud chain graphs, and enables investigators to take rapid actions — freeze accounts, flag phone numbers, alert law enforcement.

### Screenshots

#### Case Management Dashboard
![Case Management Dashboard](Screenshot%202026-04-29%20090227.png)

#### Interactive Fraud Chain Visualization
![Interactive Fraud Chain Visualization](Screenshot%202026-04-29%20095843.png)

### Key Capabilities

- **Real-time Transaction Processing** — Ingest and score transactions within milliseconds
- **Hybrid Risk Scoring** — Rule-based + ML-guided scoring with explainable factor breakdowns
- **Dynamic Case Management** — Automatically link related fraud transactions into cases with golden window tracking
- **Interactive Graph Visualization** — Cytoscape.js-powered fraud chain visualization with node actions
- **Investigative Actions** — Freeze accounts, flag phone numbers, alert police, monitor accounts
- **Recovery Tracking** — Track recoverable amounts across accounts in fraud chains
- **Audit Trail** — Complete compliance-ready action logs
- **WebSocket Broadcasting** — Real-time event streaming to investigator dashboards
- **Transaction Simulator** — 11 configurable fraud scenarios for demos and testing

---

## Architecture

```
┌──────────────────────────────────────────────────────────┐
│                   TRANSACTION STREAM                      │
│            (Simulator / API / External feeds)             │
└───────────────────────┬──────────────────────────────────┘
                        │
                        ▼
              ┌───────────────────┐
              │  POST /transaction │
              │   (FastAPI Entry)  │
              └────────┬──────────┘
                       │
              ┌────────▼────────────────────────────────┐
              │        ORCHESTRATION PIPELINE            │
              │                                         │
              │  1. Account Initialization              │
              │  2. Risk Scoring (Rule + ML Hybrid)     │
              │  3. Case Linking & Creation              │
              │  4. Transaction Graph Building           │
              │  5. Recovery Amount Calculation           │
              └────────┬────────────────────────────────┘
                       │
              ┌────────▼──────────────┐
              │  In-Memory Data Store  │
              │  (Transactions, Cases, │
              │   Graphs, Accounts,    │
              │   Actions)             │
              └────────┬──────────────┘
                       │
              ┌────────▼──────────────┐
              │  WebSocket Broadcast   │
              │  (tx_scored,           │
              │   case_updated,        │
              │   action_taken)        │
              └────────┬──────────────┘
                       │
                       ▼
┌──────────────────────────────────────────────────────────┐
│                    REACT FRONTEND                         │
│                                                           │
│  ├── Live Feed (real-time transaction stream)             │
│  ├── Dashboard (cases, KPIs, charts)                      │
│  ├── Graph Module (interactive fraud chain visualization)  │
│  ├── Action Panel (freeze / flag / alert controls)        │
│  ├── Investigation Sidebar (risk reasoning, factors)      │
│  └── WebSocket Hook (useWebSocket.js)                     │
└──────────────────────────────────────────────────────────┘
```

### Data Flow

1. **Ingestion** — Simulator or API sends transaction to `/transaction`
2. **Scoring** — 4-factor rule scoring + ML feature importance + hybrid fusion
3. **Case Linking** — Transaction linked to existing case or new case created if score exceeds threshold
4. **Graph Building** — Account nodes and transaction edges added to case graph
5. **Recovery** — Recoverable amounts recalculated based on account status
6. **Broadcasting** — Events pushed via WebSocket to all connected frontends
7. **Actions** — Investigator takes action → mock API called → action logged → status updated

---

## Tech Stack

### Frontend

| Technology | Purpose |
|-----------|---------|
| React 18 | UI component framework |
| Vite 5 | Build tool and dev server |
| TailwindCSS | Utility-first styling |
| Cytoscape.js | Graph / network visualization |
| Recharts | Charts (line, pie, bar) |
| React Router v6 | Client-side routing |

### Backend

| Technology | Purpose |
|-----------|---------|
| FastAPI | Async Python web framework |
| Uvicorn | ASGI server |
| Pydantic | Data validation |
| WebSockets | Real-time bidirectional communication |
| Python 3.10+ | Core language |

### Storage

In-memory Python dictionaries (hackathon-grade). Production would use Redis or PostgreSQL.

---

## Project Structure

```
├── backend/
│   ├── main.py                        # FastAPI entry point (routes, WebSocket, actions)
│   ├── requirements.txt
│   ├── app/
│   │   ├── engines/
│   │   │   ├── scoring_engine.py      # Hybrid rule + ML risk scoring
│   │   │   ├── case_manager.py        # Case creation, linking, lifecycle
│   │   │   ├── graph_engine.py        # Graph node/edge management
│   │   │   └── recovery_engine.py     # Recoverable amount calculation
│   │   ├── services/
│   │   │   ├── orchestrator.py        # Main pipeline (score → case → graph → recover)
│   │   │   ├── ml_risk_engine.py      # ML score emulator
│   │   │   ├── reasoning_engine.py    # Human-readable risk explanations
│   │   │   └── mock_apis.py           # Bank / Telecom / Police API stubs
│   │   ├── core/
│   │   │   ├── config.py              # Risk weights and thresholds
│   │   │   ├── constants.py           # Status enums
│   │   │   ├── data_store.py          # Global in-memory store
│   │   │   └── models/               # Pydantic models (Transaction, Case, Account, Action)
│   │   └── utils/
│   │       └── id_generator.py
│   ├── simulator/
│   │   └── simulator.py              # Transaction generator (11 fraud scenarios)
│   └── data/
│       └── transactions.json          # ML training data
│
├── frontend/
│   ├── src/
│   │   ├── App.jsx                    # Root component with routing
│   │   ├── pages/
│   │   │   ├── Feed.jsx               # Live transaction feed
│   │   │   ├── Dashboard.jsx          # Analytics dashboard with charts
│   │   │   ├── Cases.jsx              # Case management table
│   │   │   └── Graph.jsx              # Graph visualization page
│   │   ├── components/
│   │   │   ├── InvestigationSidebar.jsx  # Full investigation detail panel
│   │   │   ├── CaseCard.jsx           # Case display card with actions
│   │   │   ├── FactorBreakdown.jsx    # Risk factor visualization
│   │   │   ├── RiskBadge.jsx          # Color-coded risk indicator
│   │   │   ├── GoldenTimer.jsx        # Recovery countdown timer
│   │   │   ├── Login.jsx              # Role-based login
│   │   │   └── ...
│   │   ├── hooks/
│   │   │   └── useWebSocket.js        # Singleton store + WS + polling fallback
│   │   └── modules/
│   │       └── GraphModule/
│   │           ├── GraphCanvas.jsx    # Cytoscape.js rendering
│   │           ├── ActionPanel.jsx    # Freeze/Flag/Alert controls
│   │           ├── ActionLog.jsx      # Audit trail timeline
│   │           ├── RecoveryBar.jsx    # Recovery progress bar
│   │           ├── NodeActions.jsx    # Per-node context actions
│   │           └── Legend.jsx         # Node status legend
│   ├── package.json
│   ├── vite.config.js
│   └── tailwind.config.js
│
└── docs/                              # Dev notes and API contracts
```

---

## Getting Started

### Prerequisites

- Python 3.10+
- Node.js 16+ and npm
- Git

### Backend Setup

```bash
cd backend
python -m venv venv

# Windows
venv\Scripts\activate
# macOS/Linux
source venv/bin/activate

pip install -r requirements.txt
```

### Frontend Setup

```bash
cd frontend
npm install
```

---

## Running the System

Three terminal sessions:

### 1. Backend Server

```bash
cd backend
python main.py
```

Backend runs at `http://localhost:8000`

### 2. Frontend Dev Server

```bash
cd frontend
npm run dev
```

Frontend runs at `http://localhost:5173`

### 3. Transaction Simulator

```bash
cd backend
python simulator/simulator.py
```

Generates fraud transactions at 6 tx/min across 11 scenarios.

### Access

Open `http://localhost:5173` — login with `admin/admin123` (full access) or `viewer/viewer123` (read-only).

---

## Risk Scoring

### Formula

```
RISK_SCORE = (new_receiver × 0.35) + (amount_deviation × 0.30) +
             (time_anomaly × 0.20) + (call_flag × 0.15) +
             [dynamic boosts] + [ML contribution]
```

### Factors

| Factor | Weight | Trigger |
|--------|--------|---------|
| New Receiver | 35% | First-time recipient |
| Amount Deviation | 30% | Amount > 1.05x monthly average |
| Time Anomaly | 20% | Transaction between 10 PM – 6 AM |
| Call Flag | 15% | Active call during transaction |

### Dynamic Boosts

Cross-border transfers, device/location changes, crypto transactions, velocity attacks, bulk transfers, SIM swap patterns, remote access activity.

### Thresholds

- **HIGH_RISK** (>= 60) — Immediate case creation
- **MEDIUM** (>= 40) — Case consideration
- **LOW** (< 40) — No case

---

## API Reference

| Method | Endpoint | Description |
|--------|----------|-------------|
| GET | `/health` | Health check |
| POST | `/transaction` | Process and score a transaction |
| GET | `/cases` | List all investigation cases |
| POST | `/action/freeze` | Freeze account + downstream accounts |
| POST | `/action/flag` | Flag phone number via telecom |
| POST | `/action/alert` | Alert law enforcement |
| POST | `/action/monitor` | Place account under surveillance |
| POST | `/action/close` | Close investigation |
| POST | `/action/close_fp` | Close as false positive |
| POST | `/attack-mode` | Inject 5 high-risk attack transactions |
| GET | `/export/sentinel_audit.csv` | Export full audit trail |
| WS | `/ws` | WebSocket for real-time events |

### WebSocket Events

- `tx_scored` — Transaction processed and scored
- `case_updated` — Case status or data changed
- `action_taken` — Investigator performed an action

---

## Simulation Scenarios

| ID | Scenario | Risk | Amount Range |
|----|----------|------|-------------|
| SC-01 | Mule Chain | HIGH | 2L – 5L |
| SC-05 | Cross-Border Fraud | HIGH | 1.5L – 4L |
| SC-06 | Account Takeover | HIGH | 1L – 3L |
| SC-08 | Crypto Drain | HIGH | Variable |
| SC-02 | SIM Swap | MEDIUM | 25K – 95K |
| SC-11 | Aggregation/Mule | MEDIUM | 50K – 1.5L |
| SC-03 | Routine Transaction | LOW | 100 – 9K |
| SC-07 | Small Payment | LOW | 50 – 5K |
| SC-04 | Velocity Attack | LOW | 10 – 50 each |

---

## Case Lifecycle

```
NEW → HIGH_RISK → ACTIONED → MONITORING → CLOSED
                                        → CLOSED_FP
```

- **NEW** — Created on first high-risk transaction
- **HIGH_RISK** — Multiple suspicious transactions confirmed
- **ACTIONED** — Investigator took action (freeze, flag, alert)
- **MONITORING** — Account under surveillance
- **CLOSED** — Investigation resolved
- **CLOSED_FP** — Marked as false positive

---

## Configuration

Edit `backend/app/core/config.py`:

```python
W_NEW_RECEIVER = 0.35
W_AMOUNT_DEV = 0.30
W_TIME_ANOMALY = 0.20
W_CALL_FLAG = 0.15

HIGH_RISK_THRESHOLD = 60
MEDIUM_THRESHOLD = 40
DECAY_FACTOR = 0.85
GOLDEN_WINDOW_MINUTES = 20
```

---

## License

MIT
