# Data Dictionary — Spain Water & Wastewater

Two CSVs form this deliverable. `operators.csv` holds one current record per operator; `sources.csv` logs every individual fact with its citation (supporting audit, temporal versioning and quarterly refresh).

---

## `data/spain/water/operators.csv`

One row per operator. Values represent the most recent published figure at `data_collection_date`.

### Identity

| Column | Type | Unit | Description | Typical source |
|---|---|---|---|---|
| `operator_id` | string | — | Stable slug, lowercase, kebab-case (e.g. `es-cyii`). Used as join key to `sources.csv`. | Assigned by this project |
| `legal_name` | string | — | Full registered legal name including corporate form (S.A., S.A.U., etc.). | Mercantile registry |
| `trade_name` | string | — | Brand / commercial name actually used in market. | Corporate website |
| `parent_group` | string | — | Ultimate parent group listed in its latest annual report. | Annual report |
| `country` | string (ISO-3166-1 alpha-2) | — | Fixed to `ES` for this file. | — |
| `nif_cif` | string | — | Spanish tax identification number (CIF/NIF). 9 chars, starts with a letter. | einforma / empresia / BORME |
| `duns_number` | string | — | Dun & Bradstreet D-U-N-S number (9 digits). Blank when not publicly verifiable. | D&B |
| `lei` | string | — | Legal Entity Identifier (20 chars), when the entity has one. | GLEIF |

### Geography

| Column | Type | Unit | Description |
|---|---|---|---|
| `regions_served` | string | — | Pipe-separated list of Comunidades Autónomas where the operator holds active concessions or operates networks. |
| `hq_city` | string | — | City of the registered HQ. |
| `hq_province` | string | — | Province of the registered HQ. |

### Network (infrastructure size)

| Column | Type | Unit | Description |
|---|---|---|---|
| `network_km_drinking` | integer | km | Total drinking-water pipe network length (potabilized water distribution). |
| `network_km_sewer` | integer | km | Total sewer / wastewater collection network length. |
| `treatment_plants_drinking` | integer | count | Drinking-water treatment / potabilization plants (ETAP — Estaciones de Tratamiento de Agua Potable). |
| `treatment_plants_wastewater` | integer | count | Wastewater treatment plants (EDAR — Estaciones Depuradoras de Aguas Residuales) operated or managed. |

### Customers & volume

| Column | Type | Unit | Description |
|---|---|---|---|
| `connections_total` | integer | count | Total drinking-water service connections (acometidas / contadores). Primary size metric, analogous to supply points in the electrical sector. |
| `population_served` | integer | count | Estimated inhabitants served with drinking water. Secondary size metric (not always disclosed; may be triangulated from municipal data). |
| `water_produced_hm3` | number | hm³ | Volume of potabilized water produced and injected into the distribution network per year. |
| `wastewater_treated_hm3` | number | hm³ | Volume of wastewater treated at EDAR plants per year. |
| `nrw_pct` | number | % | Non-revenue water (agua no registrada / pérdidas) as a percentage of water input to the system. Single most important network-performance indicator for Hexagon opportunity sizing. |

### Activities & management

| Column | Type | Unit | Description |
|---|---|---|---|
| `activities` | string | — | Service activities operated: `drinking_water`, `wastewater`, or `both`. |
| `management_type` | string | — | Delivery model: `private_concession` (private operator under municipal concession), `public_company` (publicly-owned commercial company, e.g. Canal de Isabel II), `public_direct` (municipality operating directly), or `mixed` (public-private empresa mixta). |

### Financials

All in EUR. Scope: the legal entity where possible; when only a parent-group or segment figure is available, it is noted in `sources.csv` with `Medium` or `Low` confidence.

| Column | Type | Unit | Description |
|---|---|---|---|
| `revenue_eur` | number | EUR | Operating revenue of the operator entity or its closest disclosed proxy. |
| `ebitda_eur` | number | EUR | EBITDA of the operator entity (or Water Spain segment, noted). |
| `capex_eur` | number | EUR | Capital investment in water/wastewater network in `reporting_year`. |
| `employees` | integer | count | FTE employees of the operating entity or Spain-specific headcount. |
| `reporting_year` | integer | year | Fiscal year the financial/operational figures correspond to. |

### Segmentation

| Column | Type | Unit | Description |
|---|---|---|---|
| `size_tier` | string | — | Working size tier: `Tier 1 - Mega` (>3M connections), `Tier 2 - Large` (500k–3M), `Tier 3 - Mid` (50k–500k), `Tier 4 - Small` (<50k). See `methodology_water.md`. |

### Metadata

| Column | Type | Unit | Description |
|---|---|---|---|
| `data_collection_date` | date (YYYY-MM-DD) | — | Date the row was last refreshed. |
| `confidence_overall` | string | — | Aggregate confidence: `High` / `Medium` / `Low`. Set to the lowest confidence among critical fields. |
| `primary_source_url` | URL | — | Single most authoritative URL underlying the row (usually the operator's annual report or corporate 'about' page). Full citations live in `sources.csv`. |
| `notes` | string | — | Free-text caveats (ownership changes, segment-reporting caveats, concession scope). |

---

## `data/spain/water/sources.csv`

Long-form citation table. One row **per (operator, field, value, source)** combination. Enables temporal versioning (add new rows, never overwrite) and supports quarterly refresh diffs.

| Column | Type | Description |
|---|---|---|
| `operator_id` | string | Matches `operators.csv`. Use `all` for national-aggregate rows not tied to a single operator. |
| `field_name` | string | Exact column name from `operators.csv` the citation backs. |
| `value` | string | The value as recorded at `accessed_date`. Stored as string to allow ranges / qualifiers. |
| `value_unit` | string | Unit of `value` (km, hm³, count, EUR, %, etc.). |
| `source_name` | string | Human-readable source name (e.g. `INE — Encuesta sobre el suministro y saneamiento del agua 2021`). |
| `source_url` | URL | Direct URL to the underlying document / page. |
| `source_type` | string | One of: `regulator`, `company_annual_report`, `company_website`, `industry_association`, `mercantile_registry`, `press`, `analyst`. |
| `publication_date` | date | Publication date of the source document (best known). |
| `accessed_date` | date | Date the source was accessed and the value extracted. |
| `confidence` | string | `High` / `Medium` / `Low` (see `methodology_water.md` §5). |
| `notes` | string | Caveats (e.g. "group-level figure, includes international operations"). |

### Convention for updates

When a field value changes (new INE survey, new annual report) **do not** delete the old citation row — append a new one with the new `accessed_date`. The historical row documents what was true last quarter and is essential for the Phase 1 deliverable requirement that the database be "updateable quarterly (not one-time snapshot)".
