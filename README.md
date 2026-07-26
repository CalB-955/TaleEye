# TaleEye
TaleEye is a story analysis application created for the IBM Bob AI Builder's Challenge.

---

## Project Structure

```
taleeye/
├── backend/
│   ├── main.py                   # FastAPI application & API endpoints
│   ├── requirements.txt
│   ├── engine/
│   │   ├── __init__.py           # Exports run_analysis()
│   │   ├── analyze.py            # Orchestrator — runs full pipeline
│   │   ├── parser.py             # Text → characters, events, segments
│   │   ├── scorer.py             # NE scoring + Final Pacing Score
│   │   ├── tropes.py             # Trope detection (trope_taxonomy.json)
│   │   ├── continuity.py         # Continuity & retcon detection
│   │   └── uniqueness.py         # Genre-based uniqueness scoring
│   ├── schemas/                  # Copied from project schemas/
│   │   ├── character_schema.json
│   │   ├── event_schema.json
│   │   ├── segment_schema.json
│   │   ├── continuity_rules.json
│   │   └── trope_taxonomy.json
│   ├── config/                   # Scoring config
│   │   ├── scoring_weights.json
│   │   ├── project_settings.json
│   │   └── uniqueness_baselines.json
│   └── output/
│       └── projects/             # Saved project JSON files
└── frontend/
    ├── index.html
    ├── package.json
    ├── vite.config.js
    └── src/
        ├── main.jsx
        ├── App.jsx               # App shell + sidebar navigation
        ├── styles.css
        ├── hooks/
        │   ├── useAnalysis.js    # Analysis API hook
        │   └── useProjects.js    # Project CRUD hook
        ├── components/
        │   ├── PacingGraph.jsx   # NE line chart + component breakdown
        │   ├── PacingReport.jsx  # Final score, radar chart, diagnostics
        │   ├── EventTimeline.jsx # Event timeline with NE details
        │   ├── CharacterPanel.jsx# Character states over time
        │   ├── ContinuityPanel.jsx # Flags & retcon alerts
        │   └── TropeMap.jsx      # Trope chips with occurrences
        └── pages/
            ├── AnalyzerPage.jsx  # Story Analyzer mode
            └── ProjectsPage.jsx  # Projects list
```

---

## Quick Start

### Backend

```bash
cd taleeye/backend
pip install -r requirements.txt
uvicorn main:app --reload --port 8000
```

API available at `http://localhost:8000`  
Interactive docs at `http://localhost:8000/docs`

### Frontend

```bash
cd taleeye/frontend
npm install
npm run dev
```

UI available at `http://localhost:5173`

---

## API Endpoints

| Method | Endpoint | Description |
|--------|----------|-------------|
| `GET`  | `/health` | Service health check |
| `POST` | `/analyze` | Analyze story text |
| `GET`  | `/projects` | List saved projects |
| `POST` | `/projects` | Save a project |
| `GET`  | `/projects/{id}` | Load a project |
| `DELETE` | `/projects/{id}` | Delete a project |

### POST /analyze

Request body:
```json
{
  "text": "Scene 1 — ...",
  "medium": "movie_scene",
  "project_name": "My Story (optional)"
}
```

Response: Full analysis object containing `characters`, `events`, `segments`,
`continuity_report`, `pacing_report`, `trope_map`.

---

## Analysis Pipeline

```
Raw Text
   │
   ▼
parser.py  ──► characters[], events[], segments[]
   │
   ▼
scorer.py  ──► NE_components (E,I,M,S,C per segment)
             NE_total, IMR, classification
             Final Pacing Score (0-100)
   │
   ▼
tropes.py  ──► trope_map (tagged events + segments)
   │
   ▼
continuity.py ──► continuity_report (flags, retcons)
   │
   ▼
uniqueness.py ──► uniqueness score vs. genre baselines
```

---

## Scoring Details

### Narrative Energy (NE) Components (0-2 each)

| Component | Description |
|-----------|-------------|
| **E** Emotional Charge | How strongly the segment evokes emotion |
| **I** Information Density | How much new story info is delivered |
| **M** Momentum | How much the plot moves forward |
| **S** Stakes Pressure | Risk/consequence level present |
| **C** Conflict Activity | Activity of internal/external conflict |

**NE_total = E + I + M + S + C** (0–10)

### Segment Classifications

| NE Range | Classification |
|----------|----------------|
| 0–2 | Calm/Setup/Reflection |
| 3–5 | Development/Moderate movement |
| 6–8 | High tension/Major turns |
| 9–10 | Peak intensity/Climax/Revelation |

### Final Pacing Score Formula

```
Score = 10×(avg_NE/10) + 20×variance_health + 20×climax_build_quality
        + 20×rhythm_balance + 30×stakes_growth
```

Weights sourced from `scoring_weights.json`.

### Pacing Diagnostics

Detects: Slow Opening · Sagging Middle · Rushed Climax · Repetitive Rhythm ·
Emotional Flatline · Info Dumping

### Continuity Rules

Sourced from `continuity_rules.json`:
- Character state tracking (alive/dead, location, relationships)
- Dead character reappearance detection
- Location change flagging
- Identity/trait reversal detection via keyword markers

### Trope Detection

Keyword matching against `trope_taxonomy.json`:
- Hidden Truth Revealed
- Betrayal by Trusted Ally
- Sudden Power Gain
- Loss of Key Ability
- Public Humiliation

### Uniqueness Scoring

Compares story NE pattern (normalized to 5 points) against genre baselines
in `uniqueness_baselines.json` using cosine similarity.
Uniqueness = 100 − max_genre_similarity

---

## Supported Input Formats

| Format | Detection |
|--------|-----------|
| Storyboard | `=== Scene N ===` headers with `ID:`, `Location:`, `Characters:` |
| Script | `Scene N — Title` headers with narrative body |
| Freeform | Any other text (treated as single segment) |

---

## Output Files

When saving a project, all data is stored in `backend/output/projects/{id}.json`:
- `characters.json` structure
- `events.json` structure  
- `segments.json` structure
- `continuity_report.json` structure
- `pacing_report.json` structure
- `trope_map.json` structure

---

*Created for the IBM Bob AI Builder's Challenge. Original work by Calvin Bennett.*
