# Data Dictionary — Spain Telecommunications

Two CSVs form this deliverable. `operators.csv` holds one current record per telecom operator; `sources.csv` logs every individual fact with its citation (supporting audit, temporal versioning and quarterly refresh).

---

## `data/spain/telecommunications/operators.csv`

One row per operator. Values represent the most recent published figure at `data_collection_date`.

### Identity

| Column | Type | Unit | Description | Typical source |
|---|---|---|---|---|
| `operator_id` | string | — | Stable slug, lowercase, kebab-case (e.g. `es-telefonica`). Used as join key to `sources.csv`. | Assigned by this project |
| `legal_name` | string | — | Full registered legal name including corporate form (S.A., S.L.U., etc.). | Mercantile registry |
| `trade_name` | string | — | Brand / commercial name actually used in market. Pipe-separated when multiple brands belong to the same legal entity (e.g. `Orange\|Yoigo\|MásMóvil\|Jazztel`). | Corporate website |
| `parent_group` | string | — | Ultimate parent group listed in its latest annual report. For JV structures, ownership percentages are recorded (e.g. `Orange S.A. 50% / Lorca JVCo Limited 50%`). | Annual report |
| `country` | string (ISO-3166-1 alpha-2) | — | Fixed to `ES` for this file. | — |
| `nif_cif` | string | — | Spanish tax identification number (CIF/NIF). 9 chars, starts with a letter. Blank when the entity is not registered in Spain or when public verification is incomplete. | einforma / iberinform / empresite / BORME |
| `duns_number` | string | — | Dun & Bradstreet D-U-N-S number (9 digits). Blank when not publicly verifiable. | D&B |
| `lei` | string | — | Legal Entity Identifier (20 chars), when the entity has one. | GLEIF |

### Operator typology

| Column | Type | Unit | Description |
|---|---|---|---|
| `operator_type` | string | — | Network model: `MNO` (Mobile Network Operator with own radio), `MVNO` (Mobile Virtual Network Operator), `FNO` (Fixed Network Operator), `Wholesale` (B2B/wholesale-only). |
| `market_role` | string | — | Strategic role: `Incumbent`, `Challenger`, `Disruptor`, `Niche`. |
| `segments` | string | — | Pipe-separated services offered: `Mobile`, `Fixed`, `TV`, `Enterprise`, `Wholesale`, `IoT`. |
| `market_segments` | string | — | Pipe-separated customer segments: `B2C`, `B2B`, `B2B2C`, `Wholesale`. |

### Geography

| Column | Type | Unit | Description |
|---|---|---|---|
| `regions_served` | string | — | Geographic footprint. `Nacional` for nationwide operators; rural-specialist operators may use `Nacional rural y semi-urbano`; international ones `Nacional+Internacional`. |
| `hq_city` | string | — | City of the registered HQ. |
| `hq_province` | string | — | Province of the registered HQ. |

### Customers and lines

| Column | Type | Unit | Description |
|---|---|---|---|
| `mobile_lines_total` | integer | count | Total mobile lines including contract, prepaid and IoT/M2M. Some operators report excluding IoT — this is documented in `notes` per row. |
| `fixed_broadband_lines` | integer | count | Active fixed broadband subscriptions (FTTH + HFC + DSL). |
| `ftth_lines` | integer | count | Subset of `fixed_broadband_lines` that are fiber-to-the-home. |
| `pay_tv_customers` | integer | count | Pay-TV subscribers (e.g. Movistar+). Blank when service is not standalone-trackable. |

### Network (infrastructure)

| Column | Type | Unit | Description |
|---|---|---|---|
| `ftth_premises_passed` | integer | count | Unique residential/commercial premises passed by the operator's own or commercialised FTTH network. Differs from `ftth_lines` (active subscribers). |
| `mobile_sites_5g` | integer | count | Operational 5G base-station sites. Sum across operators exceeds CNMC's national figure due to multi-operator co-location. |
| `towers_or_sites` | integer | count | Owned tower / antenna sites. Mostly NaN for Top 1 operators after Telxius/Vantage divestments — they lease infrastructure from tower companies. |
| `network_km_total` | integer | km | Total fiber network length deployed, when reported by the operator. |
| `wholesale_only` | boolean | — | `true` if the operator has no retail business (e.g. Onivia). `false` otherwise. |

### Financials

All in EUR. Scope: the *Spanish operating entity*. When only a parent-group segment figure is available, this is noted in `sources.csv` with `Medium` confidence.

| Column | Type | Unit | Description |
|---|---|---|---|
| `revenue_eur` | number | EUR | Operating revenue of the Spanish entity (or "Spain segment" of the consolidated group). |
| `ebitda_eur` | number | EUR | EBITDA of the Spanish entity. Adjusted/recurring EBITDA where reported. |
| `capex_eur` | number | EUR | CapEx (excluding spectrum) for the Spanish entity. |
| `employees` | integer | count | FTE employees of the Spanish operating entity. Approximated post-ERE for operators with restructuring in progress. |

### Operational metrics (derived or reported)

| Column | Type | Unit | Description |
|---|---|---|---|
| `arpu_convergent_eur` | number | EUR/month | Average revenue per user on convergent (mobile + fixed + TV) packages. Mostly Telefónica-only — others do not publish. |
| `ebitda_margin_pct` | number | pct (0–100) | `ebitda_eur / revenue_eur × 100`. |
| `capex_intensity_pct` | number | pct (0–100) | `capex_eur / revenue_eur × 100`. Particularly relevant for operators in heavy deployment phase (e.g. Digi at 44.6%). |
| `churn_rate_pct` | number | pct | Monthly mobile contract churn (CNMC quarterly metric). **Caveat:** Finetwork's value is annual gross churn (29%) — not directly comparable. |
| `coverage_5g_pop_pct` | number | pct (0–100) | Population coverage with the operator's own 5G network. |
| `market_share_revenue_pct` | number | pct (0–100) | Operator's revenue / total Spanish telecom retail market revenue (~36 B€ per CNMC 2024). |

### Segmentation

| Column | Type | Unit | Description |
|---|---|---|---|
| `reporting_year` | integer | year | Fiscal year the financial/operational figures correspond to. |
| `size_tier` | string | — | Working size tier based on Spanish revenue: `Tier 1 - Mega` (>3B€), `Tier 2 - Large` (500M-3B€), `Tier 3 - Mid` (50-500M€), `Tier 4 - Small` (<50M€). See `methodology_telecom.md`. |
| `strategic_relevance` | string | — | Sponsor-perspective relevance: `Critical`, `High`, `Medium`, `Low`. |
| `ma_status_24m` | string | — | Free-text summary of M&A / restructuring movements in the past 24 months (mergers, spectrum divestitures, ownership changes, restructuring filings). |

### Metadata

| Column | Type | Unit | Description |
|---|---|---|---|
| `data_collection_date` | date (YYYY-MM-DD) | — | Date the row was last refreshed. |
| `confidence_overall` | string | — | Aggregate confidence: `High` / `Medium` / `Low`. Lowest confidence among critical fields. |
| `primary_source_url` | URL | — | Single most authoritative URL underlying the row (usually the operator's quarterly report or corporate page). Full citations live in `sources.csv`. |
| `notes` | string | — | Free-text caveats (legal-entity nuances, reporting perimeter, ERE impact, fiscal-year offset, etc.). |

---

## `data/spain/telecommunications/sources.csv`

Long-form citation table. One row **per (operator, field, value, source)** combination. Enables temporal versioning (add new rows, never overwrite) and supports quarterly refresh diffs.

| Column | Type | Description |
|---|---|---|
| `operator_id` | string | Matches `operators.csv`. |
| `field_name` | string | Exact column name from `operators.csv` the citation backs. The special value `notes` is allowed for sources that provide context rather than a specific field value. |
| `value` | string | The value as recorded at `accessed_date`. Stored as string to allow ranges / qualifiers. |
| `value_unit` | string | Unit of `value` (count, EUR, km, pct, etc.). |
| `source_name` | string | Human-readable source name (e.g. `Telefónica — Resultados 4T 2024`). |
| `source_url` | URL | Direct URL to the underlying document / page. |
| `source_type` | string | One of: `regulator` (CNMC), `company_annual_report`, `company_website`, `industry_association` (Nae, AOTEC, FTTH Council), `mercantile_registry` (einforma, iberinform, empresite), `press`, `analyst` (rating agencies). |
| `publication_date` | date | Publication date of the source document (best known). |
| `accessed_date` | date | Date the source was accessed and the value extracted. |
| `confidence` | string | `High` / `Medium` / `Low` (see `methodology_telecom.md` §5). |
| `notes` | string | Caveats (e.g. "estimate based on 2023 baseline + ERE 2024", "cross-validation"). |

### Convention for updates

When a field value changes (new CNMC report, new annual report) **do not** delete the old citation row — append a new one with the new `accessed_date`. The historical row documents what was true last quarter and is essential for the Phase 1 deliverable requirement that the database be "updateable quarterly (not one-time snapshot)".