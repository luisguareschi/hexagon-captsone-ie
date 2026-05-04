# Methodology — Spain Electrical Distribution (Phase 1, Deliverable 1)

This document specifies **what** is catalogued, **how** it was gathered, and **how** to refresh it quarterly.

## 1. Market structure (why the list is short)

Under Spanish law (Ley 24/2013 del Sector Eléctrico, art. 38) electricity distribution activity is split by customer count:

| Regulatory group | Threshold | Number of operators | Combined supply points |
|---|---|---|---|
| Large DSOs | > 100,000 connected customers | **5** | ~29 M (~99%) |
| Small / municipal DSOs | ≤ 100,000 customers | ~330 | ~0.3 M (~1%) |

Each DSO has a **legally exclusive concession** in the municipalities where it owns the network — Spain is not a competitive distribution market. Retail ("comercialización") is separate and competitive, but **out of scope** for this deliverable.

## 2. Scope of this deliverable

**Included:** the 5 large DSOs (>100,000 customers):

1. i-DE Redes Eléctricas Inteligentes, S.A. (Iberdrola)
2. E-distribución Redes Digitales, S.L.U. (Endesa / Enel)
3. UFD Distribución Electricidad, S.A. (Naturgy)
4. E-REDES Distribución Eléctrica, S.A.U. (EDP)
5. Viesgo Distribución Eléctrica, S.L. (Viesgo Infraestructuras Energéticas / TotalEnergies minority)

**Excluded (for now):** ~330 small distributors (< 100,000 customers). These are listed in CNMC's registry but publish very uneven data; including them would add ~1% of customers and disproportionate noise. They can be layered in during Phase 3 if whitespace analysis requires municipal granularity.

**Geography:** Spain (peninsula, Balearic Islands, Canary Islands, Ceuta). Melilla has its own non-distribution electricity regime and is excluded.

## 3. Size tier definition

Tier assignment is based on `supply_points_total` (the regulatory metric used by CNMC):

| Tier | Supply points | Rationale |
|---|---|---|
| **Tier 1 — Mega** | > 5 M | Dominant national players with consolidated group reporting |
| **Tier 2 — Large** | 1 M – 5 M | Multi-region DSOs, material but below the Big Two |
| **Tier 3 — Mid** | 100,000 – 1 M | Above the legal "large DSO" threshold but regional |
| **Tier 4 — Small** | < 100,000 | Municipal / cooperative (out of scope for this deliverable) |

This tier definition is a working classification for Hexagon expansion analysis — it does **not** replace the CNMC legal threshold of 100,000 customers, which is reported separately in `regulatory_group`.

## 4. Data sources, priority order

Where two sources disagree, the priority below breaks the tie. Every numeric field records its actual source in `sources.csv`.

1. **CNMC** (Comisión Nacional de los Mercados y la Competencia) — official regulator. Used for `supply_points_total`, `energy_distributed_gwh`, and national aggregates for QA reconciliation.
2. **AELEC** (Asociación de Empresas de Energía Eléctrica) — industry association reporting for i-DE, e-distribución, UFD.
3. **DSO own website / "Conócenos" / "Our grids" pages** — for network km, substations, transformer centers, smart meters.
4. **Parent group annual & integrated reports filed with CNMV** (Iberdrola, Endesa, Naturgy, EDP, TotalEnergies / Viesgo) — for `revenue_eur`, `ebitda_eur`, `capex_eur`, `employees`.
5. **Spanish mercantile registry / einforma / empresia / datoscif** — for `nif_cif`, legal form, incorporation date, registered address.
6. **Dun & Bradstreet / OpenCorporates** — for `duns_number`. DUNS is recorded only when publicly verifiable; otherwise the field is left blank and `notes` explains why.

## 5. Data collection rules

- **Reporting year:** latest full fiscal year for which a source has published data at collection time (2024 where available, otherwise 2023). `reporting_year` is stored per row.
- **Units:** lengths in kilometres (km), energy in gigawatt-hours (GWh), money in euros (EUR). Currency is always explicit.
- **Missing values:** `NA`. Do not guess — note the gap and raise in the next refresh cycle.
- **Confidence tag** (per row in `sources.csv`):
  - `High` — directly from regulator (CNMC) or audited annual report.
  - `Medium` — from company corporate site or industry association, not audited.
  - `Low` — press release, secondary aggregator, or analyst estimate.
- **Temporal versioning:** when a field is updated, **append** a new row to `sources.csv` rather than overwriting; the old row stays as history. `operators.csv` holds only the current ("latest") value.

## 6. QA reconciliation

Before considering the dataset "release-ready", the notebook `notebooks/01_spain_electrical_exploration.ipynb` must pass:

- Sum of `supply_points_total` across the 5 DSOs is within ±2% of the CNMC national aggregate (~30 M connected supply points in 2024).
- Sum of `energy_distributed_gwh` within ±3% of the CNMC aggregate for distribution.
- No DSO row has more than 3 `NA` cells in the must-have subset: `supply_points_total`, `network_km_total`, `revenue_eur`, `size_tier`, `nif_cif`, `primary_source_url`.

Any deviation above these bands is logged in the notebook's "Data quality" section with the likely driver (e.g. timing differences between CNMC and company publications).

## 7. Quarterly refresh playbook

Repeat each quarter (target: within 30 days of CNMC's quarterly supervisión publication):

1. **Pull** latest CNMC supervisión report (HTML + PDF) and the CNMC open-data portal supply-points download. Check if a more recent annual report is available for any parent group.
2. **Diff** new values vs. current `operators.csv`. For each changed field, append a new row to `sources.csv` (new `accessed_date`) and update the master row.
3. **Re-run** the notebook end-to-end. Verify QA checks still pass.
4. **Commit** with a message referencing the report release date, e.g. `Q2 2026 refresh — CNMC supervisión 2025-Q4`.
5. **Review**: PM signs off on the data-quality table and on any confidence downgrades.

## 8. Known limitations

- **Distribution-only financials are rarely broken out cleanly.** Parent groups report a "Networks" or "Redes" segment that often mixes Spain with other geographies (Iberdrola, EDP) or with transport (Red Eléctrica does not distribute, so no confusion there). Where only a group-level or mixed-geography figure is available, it is recorded with a note and `Medium` confidence.
- **Viesgo ownership history is complex.** E.ON → Macquarie consortium → current ownership; the *distribution* legal entity (Viesgo Distribución Eléctrica, S.L., CIF B62733159) has been stable, but the ultimate parent varies across years.
- **E-REDES footprint beyond Asturias** (small networks in Comunidad Valenciana, Cataluña, Aragón) stems from legacy HC Energía assets and is not always reported separately from Asturias totals.
- **Smart-meter counts** are frozen at the moment of reporting; the Spanish rollout is >99% complete so changes are marginal.

## 9. Out of scope (handled in later phases)

- Retail electricity commercialisation (comercializadoras) — Phase 3 if whitespace requires it.
- Transport (TSO) — Red Eléctrica de España is the sole TSO and is a separate vertical.
- Small distributors (< 100,000 customers) — optional Phase 3 add-on.
- Other verticals (water, gas, telecom, transit) — separate country-vertical slices following the same schema.
