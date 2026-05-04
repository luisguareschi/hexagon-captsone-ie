# Methodology — Spain Telecommunications (Phase 1, Deliverable 1)

This document specifies **what** is catalogued, **how** it was gathered, and **how** to refresh it quarterly. It mirrors the structure of `methodology.md` in the electrical-distribution slice for cross-vertical consistency.

## 1. Market structure (why these operators were selected)

The Spanish telecommunications market is **competitive and concentrated**, regulated by CNMC (Comisión Nacional de los Mercados y la Competencia) under Ley 9/2014 General de Telecomunicaciones. Unlike electricity distribution (legal monopoly per concession), telecom operators compete in retail and share infrastructure via wholesale agreements and FiberCo joint ventures.

The market has three structural layers:

| Layer | Role | Examples in this dataset |
|---|---|---|
| **Mobile Network Operators (MNO)** with own radio + fixed network | Build and operate physical infrastructure | Telefónica, MasOrange, Vodafone |
| **Fixed Network Operators (FNO) / FTTH-only** | Own fiber, no own mobile radio | Adamo, Onivia (wholesale-only), Avatel |
| **MVNOs / wholesale-buyers** | Lease capacity from MNOs/FNOs | Digi (in transition to MNO), Finetwork, Aire Networks |

There is no formal regulatory threshold equivalent to electricity's "100,000 customers" line, but CNMC publishes operator data quarterly and tracks ~1,400 registered operators. The Top 4 (Telefónica, MasOrange, Vodafone, Digi) hold ~85% of mobile and fixed-broadband market share.

| Market layer | Number of operators | Combined revenue share |
|---|---|---|
| Tier 1 — Mega (>3 B€) | **3** | ~66% |
| Tier 2 — Large (500 M€ – 3 B€) | **2** | ~3% |
| Tier 3 — Mid (50 M€ – 500 M€) | **4** | ~2% |
| Tier 4 — Small (<50 M€) | ~1,400 | ~29% (very fragmented MVNOs and local operators) |

## 2. Scope of this deliverable

**Included:** 9 operators covering the strategic archetypes of the Spanish market:

1. i-DE-equivalent in scale: **es-telefonica** — Incumbent integrated MNO (Movistar, O2)
2. **es-masorange** — Challenger fusion JV (Orange + MásMóvil, March 2024)
3. **es-vodafone** — 3rd MNO acquired by Zegona Communications (May 2024)
4. **es-digi** — Romanian disruptor in MVNO→MNO transition
5. **es-finetwork** — Residential MVNO in restructuring (Vodafone takeover Sept 2025)
6. **es-avatel** — Rural FTTH consolidator under sale process
7. **es-adamo** — Rural FTTH premium owned by Ardian Infrastructure
8. **es-onivia** — Largest neutral wholesale FTTH platform (post-Digi acquisition)
9. **es-aire** — B2B wholesale + cloud + datacenters (Grupo Aire)

These 9 operators represent ~85-90% of market value and **all relevant operating archetypes** for sponsor analysis. The selection is intentional: cover the spectrum from incumbent to disruptor to niche-rural to wholesale-only without duplicating sub-brands (Pepephone, Lebara, Yoigo, Lowi, Jazztel, etc., are consolidated within their parent groups).

**Excluded (for now):**

- **Sub-brands and low-cost MVNOs** within the four large groups (Pepephone, Simyo, Lebara, Lowi, Yoigo, Jazztel, Llamaya, Virgin Telco). Their KPIs are reported only at parent-group level.
- **Regional operators absorbed into MasOrange** (Euskaltel, R, Telecable, Embou).
- **Wholesale specialists not relevant for Hexagon/Octave product fit** (Lyntia dark-fiber, Bluevía Telefónica's tower co, etc.).
- **TV-only operators and small local FTTH players** (~330 active per CNMC). They can be layered in Phase 3 if whitespace requires regional granularity.

**Geography:** Spain (peninsula, Balearic Islands, Canary Islands). All financials and KPIs are at the **Spanish market level**, not at international-group level. Where parent groups report only consolidated figures, this is flagged in `notes` and `confidence_overall` is downgraded.

## 3. Size tier definition

Tier assignment is based on `revenue_eur` of the Spanish operating entity. This differs from the electrical-distribution slice (which uses supply points) because telecom revenue is more comparable across diverse business models (MNO, MVNO, wholesale-only, B2B).

| Tier | Revenue (Spain) | Operators in this dataset |
|---|---|---|
| **Tier 1 — Mega** | > 3,000 M€ | Telefónica, MasOrange, Vodafone |
| **Tier 2 — Large** | 500 M€ – 3,000 M€ | Digi, Onivia |
| **Tier 3 — Mid** | 50 M€ – 500 M€ | Finetwork, Avatel, Adamo, Aire Networks |
| **Tier 4 — Small** | < 50 M€ | (out of scope) |

Tier classification is a working bucket for sponsor expansion analysis. The value-chain position (`operator_type`, `market_role`) carries equal analytical weight — a Tier 2 wholesale-only operator like Onivia may be a higher-priority sponsor target than a Tier 1 MNO, depending on Hexagon/Octave product fit.

## 4. Data sources, priority order

Where two sources disagree, the priority below breaks the tie. Every numeric field records its actual source in `sources.csv`.

1. **CNMC** (Comisión Nacional de los Mercados y la Competencia) — official regulator. Used for sector aggregates, churn, sites 5G distribution, and market-share validation. Quarterly Sector Report and annual *Informe Económico Sectorial*.
2. **Operator quarterly/annual results** filed with stock exchanges (Telefónica with CNMV, Digi Communications with Bucharest SE, Zegona with London SE) — for revenue, EBITDA, CapEx, customer counts of the listed groups.
3. **Operator corporate communications** (sala de prensa, blog corporativo) — for KPIs, deployment milestones, network reach.
4. **Industry-association reports** (Nae BarómetroTelco, AOTEC studies) — for cross-operator comparisons of 5G sites, churn, market share. Especially valuable when CNMC data lags.
5. **Spanish mercantile registry / einforma / iberinform / empresite** — for `nif_cif`, legal form, registered address, employees of private companies.
6. **Specialised financial press** (The Objective, Cinco Días, Expansión, El Economista, Revista Cloud, Mobile World Live) — for M&A activity, restructuring details, and private-company figures derived from cuentas anuales depositadas.
7. **Rating agency reports** (EthiFinance, S&P, Moody's) — for private companies with rated debt (e.g. Avatel's BB- rating from EthiFinance).

## 5. Data collection rules

- **Reporting year:** latest full fiscal year for which a source has published data at collection time. **Critical:** Vodafone España uses fiscal year April–March (FY2024-25 ended 31-March-2025). For consistency with calendar-year operators it is reported as `reporting_year=2024`, but this is documented in `notes`. Avatel and Aire have `reporting_year=2023` because no FY2024 audited accounts were available at collection time.
- **Units:** counts as integers, money in euros (EUR, no thousands separators), percentages as decimals (e.g. `0.99` for monthly mobile churn or `91` for population coverage), distances in kilometres.
- **Missing values:** empty cell. Pandas reads as `NaN`. **Do not guess.** If a field requires estimation, document the calculation in `notes` and downgrade `confidence_overall`.
- **Confidence tag** (per row in `sources.csv`):
  - `High` — directly from regulator (CNMC) or audited annual report (parent group filings, depositadas en Registro Mercantil).
  - `Medium` — from corporate website, press release with corporate spokesperson, or industry association.
  - `Low` — secondary aggregator, analyst estimate without disclosed methodology, or own estimate.
- **Multi-brand handling:** when an operator runs multiple brands (Telefónica → Movistar+O2, MasOrange → Orange+Yoigo+Jazztel+MásMóvil, etc.), `trade_name` uses pipe-separated values (`Orange|Yoigo|MásMóvil|Jazztel`) and KPIs are the sum across all brands of the legal entity.
- **Temporal versioning:** when a field is updated, **append** a new row to `sources.csv` rather than overwriting; the old row stays as history. `operators.csv` holds only the current ("latest") value.

## 6. QA reconciliation

Before considering the dataset "release-ready", the notebook `notebooks/02_spain_telecom_exploration.ipynb` must pass:

- Sum of `mobile_lines_total` across the 4 MNOs (Telefónica, MasOrange, Vodafone, Digi) within ±5% of CNMC national aggregate (~61.3 M lines in 2024). The 5% margin (vs ±2-3% in electric) reflects MVNO line-counting overlap.
- Sum of `ftth_premises_passed` across the 5 FTTH operators (Telefónica, MasOrange, Onivia, Digi, Adamo) is a meaningful lower bound on national FTTH footprint — duplicated coverage is real and expected (overbuild).
- Sum of `market_share_revenue_pct` across all 9 operators should be ~70% (the rest is fragmented MVNOs / local operators outside the dataset).
- No critical row (Tier 1 or Tier 2) has more than 3 NaN cells in: `revenue_eur`, `mobile_lines_total` OR `ftth_premises_passed`, `size_tier`, `nif_cif`, `primary_source_url`.
- Wholesale-only operators (`wholesale_only=true`) may have many NaN retail KPIs — this is expected and not a quality issue.

Any deviation above these bands is logged in the notebook's "Data quality" section.

## 7. Quarterly refresh playbook

Repeat each quarter (target: within 30 days of CNMC's quarterly Sector Report publication):

1. **Pull** the latest CNMC quarterly Sector Report and annual *Informe Económico Sectorial* (typically published mid-year). Cross-check operator quarterly/annual results filings.
2. **Diff** new values vs. current `operators.csv`. For each changed field, append a new row to `sources.csv` (new `accessed_date`) and update the master row.
3. **Re-run** `notebooks/02_spain_telecom_exploration.ipynb` end-to-end. Verify QA checks still pass.
4. **Commit** with a message referencing the report release date, e.g. `Q2 2026 refresh — CNMC sector report 2025-Q4`.
5. **Review:** PM signs off on the data-quality table and on any confidence downgrades.

## 8. Known limitations

- **Vodafone fiscal-year offset.** Vodafone España (Zegona) uses April–March fiscal years; FY2024-25 ends 31-March-2025. All other operators use calendar years. The `reporting_year=2024` tag for Vodafone is a working compromise that needs to be flagged in any temporal analysis.
- **MasOrange Q1 2024 is not consolidated.** The merger closed 26-March-2024. Q1 2024 reports the sum of separate Orange España + MásMóvil pre-merger entities; Q2-Q4 are consolidated. Year-on-year growth comparisons must use the proforma sum of 2023 as baseline.
- **Avatel and Aire FY2024 not audited.** Both operators' last publicly available audited accounts are FY2023. Their `reporting_year=2023` while the four large operators are 2024 — temporal comparisons require care.
- **Finetwork in restructuring.** As of September 2025 Vodafone took control via debt capitalisation. The FY2024 figures captured here (157 M€ revenue, 6 M€ EBITDA) reflect the operator's last independent year. Subsequent quarters will reclassify the entity as a Vodafone subsidiary.
- **Onivia and Aire are private companies** without public results disclosure. Revenue, EBITDA, CapEx and headcount are partially estimated from registered accounts (cuentas anuales depositadas) and are flagged with `Medium` confidence.
- **B2B revenue split is rarely disclosed.** Operators report B2B at parent-group level (Telefónica Tech, Vodafone Business) but rarely break it out for the Spanish entity. The `b2b_revenue_share_pct` field was excluded from the schema for this reason — the data simply does not exist publicly for most operators.
- **Mobile sites count differs from CNMC-reported sites.** CNMC reports total *deployed* 5G sites (~31,000 in 2024), while individual operator figures (Telefónica 14,098, MasOrange 12,188, Vodafone ~8,500) sum to higher values due to sites with multiple operators co-located. Operator-reported sites are used because they reflect real operational footprint per operator.
- **Telxius / tower-co divestments.** Telefónica sold Telxius to American Tower in 2021; Vodafone sold towers to Vantage/Cellnex previously. `towers_or_sites` is therefore mostly NaN for Tier 1 operators because they no longer own the physical sites — they lease them.

## 9. Out of scope (handled in later phases or other slices)

- **TV-only / streaming services** — Movistar+ is captured as `pay_tv_customers` of Telefónica, but standalone OTT players (Filmin, Atresplayer, etc.) are excluded.
- **Tower companies / FiberCos** — Cellnex, American Tower (ex-Telxius), Vantage Towers, Bluevía are infrastructure-only and out of scope for the operator slice. May warrant a separate "infrastructure" slice in Phase 3.
- **Sub-brands within the Top 4** (Pepephone, Simyo, Lebara, Yoigo, Jazztel, Lowi, Virgin Telco) — consolidated within their parent groups.
- **Local rural FTTH operators below 50 M€** (~330 listed by CNMC) — long tail, optional Phase 3 add-on.
- **Other verticals** (water, gas, electrical distribution, transit) — separate country-vertical slices following the same schema.