# Market Intelligence Analytics — IE Capstone 2026

**Sponsor:** Hexagon | **Team:** 6 IE Business School students | **Duration:** Apr 7 – Jul 14, 2026

This repository hosts the Market Intelligence system that answers two questions for sponsor leadership:

1. Where is our biggest opportunity?
2. Where should we focus / expand?

The project runs in three phases (see `IE Capstone Brief v2.pdf` for the full scope):

- **Phase 1 (Apr 7 – May 12):** Outside-in market map from public data → TAM model → expansion targets.
- **Phase 2 (May 13 – Jun 16):** Inside-out analysis with internal customer / revenue data (pending Finance approval).
- **Phase 3 (Jun 17 – Jul 14):** Integration → market share → whitespace → expansion roadmap.

## Current status — Phase 1, Deliverable 1

First country × vertical slice: **Spain — Electrical Distribution (DSO) operators**.

```
data/spain/electrical_distribution/
├── operators.csv      # one row per DSO, grid / customers / revenue / tier / D&B ID
└── sources.csv        # long-form citations (one row per field-value-source)
docs/
├── data_dictionary.md # every column defined, units, source type, confidence
└── methodology.md     # scope, tier rules, refresh playbook
notebooks/
└── 01_spain_electrical_exploration.ipynb  # summary stats + charts + QA
```

## How to refresh the data (quarterly playbook)

1. Download the latest **CNMC** "Informe de supervisión de la distribución de energía eléctrica" and extract supply points + energy distributed per DSO.
2. For each of the 5 large DSOs pull the parent group's most recent annual / sustainability report for network-km and distribution-segment financials.
3. Update `operators.csv` values and append a new citation row per edited field in `sources.csv` (keeping the old row preserves quarterly history).
4. Re-run `notebooks/01_spain_electrical_exploration.ipynb` to refresh charts and the QA reconciliation.
5. Bump `data_collection_date` on every row touched.

Full refresh details are in [`docs/methodology.md`](docs/methodology.md).

## Tech stack (Phase 1)

CSV + Python (pandas) + Jupyter. The schema is intentionally flat and SQL-ready — it can be lifted to SQLite, PostgreSQL or Snowflake via `pandas.to_sql()` without structural changes when the stack decision is finalised.

## Local setup

```bash
python3 -m venv .venv
source .venv/bin/activate
pip install -r requirements.txt
jupyter notebook notebooks/01_spain_electrical_exploration.ipynb
```

## Contacts

- Scope / strategy: Tobias Pforr
- Technical / methodology: IE Academic Mentor
- Logistics: Daniela Rivas (Project Manager)
