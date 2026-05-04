# Methodology — Spain Water & Wastewater (Phase 1, Deliverable 1)

This document specifies **what** is catalogued, **how** it was gathered, and **how** to refresh it quarterly.

## 1. Market structure (why the list is manageable)

Spain's water and wastewater sector is constitutionally a **municipal competency** (Ley 7/1985 Reguladora de las Bases del Régimen Local, art. 25). Around 8,100 municipalities provide water services either:

| Delivery model | Description | Approx. share of connections |
|---|---|---|
| **Gestión directa** | Municipality operates service through its own body or a public company | ~45% |
| **Gestión indirecta** | Municipality contracts a private operator via concession | ~45% |
| **Mixed / joint venture** | Public-private partnership (empresa mixta) | ~10% |

Unlike the electrical sector, **Spain has no national economic water regulator** equivalent to Ofwat (UK) or ARERA (Italy). Tariff regulation sits entirely at municipal and regional level, which substantially reduces public data availability relative to the UK and Italy.

In practice, the market is dominated by five operator groups (three private, two public-origin) that collectively serve an estimated 35–38 million of Spain's ~47 million inhabitants through concession contracts or direct public provision.

| Group type | Number of operators | Estimated share of national connections |
|---|---|---|
| National private operators (concession) | ~5 groups | ~60% |
| Large public or publicly-owned operators | ~3–5 companies | ~25% |
| Small municipal operators (<50,000 connections) | ~thousands | ~15% |

## 2. Scope of this deliverable

**Included:** the 5 largest operators by connections served in Spain:

1. FCC Aqualia, S.A.U. (FCC Group)
2. Sociedad General de Aguas de Barcelona, S.A. (Agbar / SUEZ España)
3. Canal de Isabel II, S.A. (Comunidad de Madrid)
4. Acciona Agua, S.A.U. (Acciona Group)
5. Global Omnium Gestión en Infraestructuras, S.A. (Global Omnium Group)

**Excluded (for now):** hundreds of smaller municipal operators and empresa mixta arrangements. These exist across all regions but publish very uneven data. They can be layered in during Phase 3 if whitespace analysis requires municipal granularity.

**Geography:** Spain (peninsula, Balearic Islands, Canary Islands, Ceuta, Melilla). Offshore territories served by separate island water councils are excluded unless an operator above explicitly covers them.

**Vertical scope:** drinking water (abastecimiento) + wastewater (saneamiento/depuración) treated together. Operators are tagged by activity so views can be split. The 2024 revision of the Urban Wastewater Treatment Directive (UWWTD) is wastewater-specific and is the single largest regulatory capex trigger through 2045; excluding wastewater would gut the regulatory narrative.

## 3. Size tier definition

Tier assignment is based on `connections_total` (drinking water service connections — the closest operational analogue to supply points in the electrical sector):

| Tier | Connections | Rationale |
|---|---|---|
| **Tier 1 — Mega** | > 3 M | National-scale operators with multi-region footprint |
| **Tier 2 — Large** | 500k – 3 M | Major regional or multi-region operators |
| **Tier 3 — Mid** | 50k – 500k | City-level or multi-municipality operators |
| **Tier 4 — Small** | < 50k | Municipal-scale (out of scope for this deliverable) |

This tier definition is a working classification for Hexagon expansion analysis. It does **not** correspond to any Spanish legal threshold (none exists for water). The thresholds are calibrated to mirror the electrical sector's tier logic at the scale of the Spanish water market.

## 4. Data sources, priority order

Where two sources disagree, the priority below breaks the tie. Every numeric field records its actual source in `sources.csv`.

1. **INE** (Instituto Nacional de Estadística) — Encuesta sobre el suministro y saneamiento del agua. Published ~2 years in arrears; most recent cycle covers reference year 2021. Used for national aggregates (total connections, water produced, NRW) for QA reconciliation.
2. **MITERD** (Ministerio para la Transición Ecológica y el Reto Demográfico) — DSEAR programme data (Plan Nacional de Depuración, Saneamiento, Eficiencia, Ahorro y Reutilización) and Plan Hidrológico Nacional statistics.
3. **Canal de Isabel II annual report** — own audited annual report (Informe Anual / Memoria Anual), available for 2022 and 2023 at time of collection. Used as primary for Canal de Isabel II operator data.
4. **Listed parent group annual reports filed with CNMV / international stock exchanges** (FCC, Acciona) — for `revenue_eur`, `ebitda_eur`, `capex_eur`, `employees` at group or segment level.
5. **DSO / operator corporate website** — for network km, treatment plants, connections where the operator publishes this voluntarily (Canal de Isabel II, Agbar, Global Omnium websites).
6. **Spanish mercantile registry / einforma / empresia / BORME** — for `nif_cif`, legal form, incorporation date, registered address.
7. **EurEau, Europe's Water in Figures 2021** — for NRW benchmarks. Supplemented with the most recent national figure when the operator publishes its own NRW.
8. **Dun & Bradstreet / OpenCorporates** — for `duns_number` when publicly verifiable.

## 5. Data collection rules

- **Reporting year:** latest full fiscal year for which a source has published data at collection time (2023 where available, otherwise 2022). `reporting_year` is stored per row.
- **Units:** lengths in kilometres (km), water volumes in cubic hectometres (hm³), money in euros (EUR). Currency is always explicit.
- **Missing values:** `NA`. Do not guess — note the gap and raise in the next refresh cycle.
- **Confidence tag** (per row in `sources.csv`):
  - `High` — directly from INE, MITERD, or operator's audited annual report.
  - `Medium` — from company corporate site, industry association, or parent group report (not Spain-specific).
  - `Low` — press release, secondary aggregator, analyst estimate, or triangulation from proxy data.
- **Temporal versioning:** when a field is updated, **append** a new row to `sources.csv` rather than overwriting; the old row stays as history. `operators.csv` holds only the current ("latest") value.

## 6. QA reconciliation

Before considering the dataset "release-ready", the notebook `notebooks/01_spain_water_exploration.ipynb` must pass:

- Sum of `connections_total` across the 5 operators is within ±10% of the INE national aggregate (~13.3 M drinking-water connections in 2021). The higher tolerance (vs. ±2% for electrical) reflects the absence of a real-time regulatory database and the partial overlap of operator reporting periods.
- Sum of `water_produced_hm3` within ±15% of the INE national total (~4,500 hm³/year).
- No operator row has more than 3 `NA` cells in the must-have subset: `connections_total`, `nif_cif`, `size_tier`, `primary_source_url`, `activities`.

Any deviation above these bands is logged in the notebook's "Data quality" section with the likely driver (e.g. timing differences, INE vs. operator reporting scope).

## 7. Quarterly refresh playbook

Repeat each quarter (target: within 30 days of INE publishing its annual water survey results):

1. **Pull** the latest INE Encuesta sobre el suministro y saneamiento del agua cycle. Check if Canal de Isabel II's new annual report is available (published ~Q2 each year for the prior fiscal year). Check FCC and Acciona annual reports (published ~Q1 for prior year).
2. **Diff** new values vs. current `operators.csv`. For each changed field, append a new row to `sources.csv` (new `accessed_date`) and update the master row.
3. **Re-run** the notebook end-to-end. Verify QA checks still pass.
4. **Commit** with a message referencing the data release date, e.g. `Q2 2026 refresh — INE Encuesta Agua 2022`.
5. **Review:** PM signs off on the data-quality table and on any confidence downgrades.

## 8. Known limitations

- **No national economic regulator means no compulsory public disclosure.** Unlike the UK (Ofwat annual return) or Italy (ARERA-submitted data), Spanish water operators are under no legal obligation to publish comparable operational or financial data at national level. Financial data for private operators (Aqualia Spain, Agbar Spain, Acciona Agua Spain) is largely unavailable without paid access to Orbis/SABI. Canal de Isabel II is the main exception as a public company subject to transparency requirements.
- **INE data lags by ~2 years.** The most recent published survey at collection time (reference year 2021) is used as the QA anchor. Operator-reported figures may differ from INE due to coverage scope and timing.
- **Concession contract fragmentation.** Each of the private operators (Aqualia, Agbar, Acciona Agua) manages hundreds of separate municipal concession contracts, each with its own tariff, reporting obligation, and contract term. The `regions_served` field aggregates across concessions; individual concession detail is out of scope for Phase 1.
- **Agbar/SUEZ España ownership structure is complex.** Agbar (Sociedad General de Aguas de Barcelona, S.A.) was ~75% owned by SUEZ prior to the Veolia/SUEZ restructuring (2021–2022). After restructuring, Agbar remains part of the reconstituted SUEZ group (not absorbed into Veolia). The exact current ownership split should be verified before Phase 2.
- **Canal de Isabel II financials are the most complete** but cover Madrid region only. They are not representative of private operator economics.
- **NRW figures** for private operators are not publicly disclosed at operator level. Values in this dataset are either operator-published (Canal de Isabel II) or estimated from INE regional aggregates.

## 9. Key pressure drivers (Spain-specific)

| Driver | Intensity | Notes |
|---|---|---|
| **Drought** | Very high | Spain is the driest country in Western Europe; 2022–2024 consecutive drought years drove water-use restrictions in Catalonia, Andalusia, and the Canaries. |
| **UWWTD 2024 revision** | Medium–high | Spain has outstanding gaps in secondary and tertiary treatment at small agglomerations; MITERD's DSEAR programme (2022–2027) is partly a response. |
| **NRW / network ageing** | Medium–high | National average NRW ~20–25% (EurEau); much of Spain's pipe network dates from 1960s–1980s. |
| **Tariff reform pressure** | Medium | Municipalities keep tariffs below cost-reflective levels in many areas; national affordability debate is ongoing. |
| **Digital / smart water** | Medium | Adoption of AMI (smart meters), pressure management, and hydraulic modelling is advancing but patchy. |

## 10. Out of scope (handled in later phases)

- Retail water commercialisation (gestión de clientes) — out of scope for Phase 1.
- Irrigation water management (comunidades de regantes) — separate vertical.
- Flood management and river basin authorities (Confederaciones Hidrográficas) — upstream of the water service operators, separate vertical.
- Other verticals (electrical, gas, telecom, transit) — separate country-vertical slices following the same schema.
- Small municipal operators (< 50,000 connections) — optional Phase 3 add-on.
