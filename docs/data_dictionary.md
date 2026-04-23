# Data Dictionary — Spain Electrical Distribution

Two CSVs form this deliverable. `operators.csv` holds one current record per DSO; `sources.csv` logs every individual fact with its citation (supporting audit, temporal versioning and quarterly refresh).

---

## `data/spain/electrical_distribution/operators.csv`

One row per operator. Values represent the most recent published figure at `data_collection_date`.

### Identity

| Column | Type | Unit | Description | Typical source |
|---|---|---|---|---|
| `operator_id` | string | — | Stable slug, lowercase, kebab-case (e.g. `es-ide`). Used as join key to `sources.csv`. | Assigned by this project |
| `legal_name` | string | — | Full registered legal name including corporate form (S.A., S.L.U., etc.). | Mercantile registry |
| `trade_name` | string | — | Brand / commercial name actually used in market. | Corporate website |
| `parent_group` | string | — | Ultimate parent group listed in its latest annual report. | Annual report |
| `country` | string (ISO-3166-1 alpha-2) | — | Fixed to `ES` for this file. | — |
| `nif_cif` | string | — | Spanish tax identification number (CIF/NIF). 9 chars, starts with a letter. | einforma / empresia / BORME |
| `duns_number` | string | — | Dun & Bradstreet D-U-N-S number (9 digits). Blank when not publicly verifiable. | D&B |
| `lei` | string | — | Legal Entity Identifier (20 chars), when the entity has one. | GLEIF |

### Geography

| Column | Type | Unit | Description |
|---|---|---|---|
| `regions_served` | string | — | Pipe-separated list of Comunidades Autónomas where the DSO operates networks. |
| `hq_city` | string | — | City of the registered HQ. |
| `hq_province` | string | — | Province of the registered HQ. |

### Network (grid size)

| Column | Type | Unit | Description |
|---|---|---|---|
| `network_km_total` | integer | km | Total distribution line length (HT + MT + LT combined). |
| `network_km_ht` | integer | km | High-voltage distribution lines (≥ 36 kV, below transmission voltages). Blank when not broken out. |
| `network_km_mt` | integer | km | Medium-voltage lines (1 kV – 36 kV). |
| `network_km_bt` | integer | km | Low-voltage lines (< 1 kV). |
| `transformer_substations_count` | integer | count | Transformer centers (centros de transformación) in service. |
| `primary_substations_count` | integer | count | HV/MV substations (subestaciones). |
| `smart_meters_installed` | integer | count | Smart meters in service. Spain rollout is >99% complete. |

### Customers & energy

| Column | Type | Unit | Description |
|---|---|---|---|
| `supply_points_total` | integer | count | Total connected supply points (puntos de suministro). Primary size metric. |
| `energy_distributed_gwh` | number | GWh | Energy delivered through the DSO network in `reporting_year`. |

### Financials

All in EUR. Scope: the *legal distribution entity* where possible; when only a parent-group "Networks Spain" figure is available, it is noted in `sources.csv` with `Medium` confidence.

| Column | Type | Unit | Description |
|---|---|---|---|
| `revenue_eur` | number | EUR | Operating revenue of the distribution entity. |
| `regulated_remuneration_eur` | number | EUR | Regulated remuneration recognised for distribution activity under CNMC circular 6/2019. Blank when not separately disclosed. |
| `ebitda_eur` | number | EUR | EBITDA of the distribution entity (or Networks Spain segment, noted). |
| `capex_eur` | number | EUR | Investment in distribution network in `reporting_year`. |
| `employees` | integer | count | FTE employees of the distribution legal entity. |
| `reporting_year` | integer | year | Fiscal year the financial/operational figures correspond to. |

### Segmentation

| Column | Type | Unit | Description |
|---|---|---|---|
| `size_tier` | string | — | Working size tier: `Tier 1 - Mega` (>5M), `Tier 2 - Large` (1-5M), `Tier 3 - Mid` (100k-1M), `Tier 4 - Small` (<100k). See `methodology.md`. |
| `regulatory_group` | string | — | Legal classification: `large_dso` (>100k customers under Ley 24/2013) or `small_dso` (≤100k). |

### Metadata

| Column | Type | Unit | Description |
|---|---|---|---|
| `data_collection_date` | date (YYYY-MM-DD) | — | Date the row was last refreshed. |
| `confidence_overall` | string | — | Aggregate confidence: `High` / `Medium` / `Low`. Lowest confidence among critical fields. |
| `primary_source_url` | URL | — | Single most authoritative URL underlying the row (usually CNMC or the DSO corporate page). Full citations live in `sources.csv`. |
| `notes` | string | — | Free-text caveats (ownership changes, segment-reporting caveats). |

---

## `data/spain/electrical_distribution/sources.csv`

Long-form citation table. One row **per (operator, field, value, source)** combination. Enables temporal versioning (add new rows, never overwrite) and supports quarterly refresh diffs.

| Column | Type | Description |
|---|---|---|
| `operator_id` | string | Matches `operators.csv`. |
| `field_name` | string | Exact column name from `operators.csv` the citation backs. |
| `value` | string | The value as recorded at `accessed_date`. Stored as string to allow ranges / qualifiers. |
| `value_unit` | string | Unit of `value` (km, GWh, count, EUR, etc.). |
| `source_name` | string | Human-readable source name (e.g. `CNMC — Informe de supervisión 2024`). |
| `source_url` | URL | Direct URL to the underlying document / page. |
| `source_type` | string | One of: `regulator`, `company_annual_report`, `company_website`, `industry_association`, `mercantile_registry`, `press`, `analyst`. |
| `publication_date` | date | Publication date of the source document (best known). |
| `accessed_date` | date | Date the source was accessed and the value extracted. |
| `confidence` | string | `High` / `Medium` / `Low` (see `methodology.md` §5). |
| `notes` | string | Caveats (e.g. "group-level figure, includes UK and US networks"). |

### Convention for updates

When a field value changes (new CNMC report, new annual report) **do not** delete the old citation row — append a new one with the new `accessed_date`. The historical row documents what was true last quarter and is essential for the Phase 1 deliverable requirement that the database be "updateable quarterly (not one-time snapshot)".
