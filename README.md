# Bank ALM Master Model — BNP Paribas (Excel)

> A publication-grade Asset-Liability Management model built on BNP Paribas Q4 2025 actuals.
> Designed to mirror the analytical framework used by Treasury and ALM desks at Tier-1 EU banks.

---

## Overview

This workbook implements a full-cycle ALM engine covering interest rate risk, liquidity risk, capital adequacy, and income simulation — calibrated to regulatory standards and real balance sheet data.

The model is structured around four interconnected sheets, driven by a single scenario selector that propagates through every output simultaneously. All outputs are formula-linked; no hardcoded values exist in the calculation chain.

---

## Architecture

| Sheet | Role |
|---|---|
| **Dashboard** | Executive ALCO view — scenario selector, KPI cards, NII waterfall, EVE sensitivity, LCR/NSFR trend, CET1 bridge |
| **Engine** | Central control layer — rate curve, scenario inputs, CHOOSE-based propagation logic |
| **Model** | Full analytical engine — 15 modular sections (see below) |
| **Master Balance Sheet** | Source-of-truth balance sheet — actuals + three forecast years, structured for CSV export |

---

## Model Sections (Model Sheet)

1. **Balance Sheet Summary** — Assets, liabilities, equity; actuals-to-forecast bridge
2. **NII Simulation** — Monthly NII with repricing schedule, margin decomposition
3. **NIM Sensitivity** — NIM trend analysis across scenarios; 2024A anchor
4. **Repricing Gap** — Static gap table by maturity bucket; cumulative gap position
5. **PVBP / DV01** — Price value of a basis point by asset class; portfolio-level rate sensitivity
6. **EVE Analysis** — Economic Value of Equity under rate shocks; hedge offset calibration (55% factor)
7. **FTP Curve** — Internal funds transfer pricing curve with worked examples; spread attribution
8. **Maturity Ladder** — Contractual cash flow ladder; survival horizon analysis
9. **LCR** — Liquidity Coverage Ratio per CRR2 / Delegated Regulation EU 2022/786; HQLA haircuts, outflow rates
10. **NSFR** — Net Stable Funding Ratio per CRR Art. 428a–428at; ASF/RSF weights, trading book RSF
11. **CET1 / RWA** — Common Equity Tier 1 ratio bridge; RWA by risk type; capital buffer analysis
12. **Scenario Manager** — Four scenarios (Base / Rate Hike / Rate Cut / Stress) with rate, growth, and credit shock parameters
13. **IRRBB Summary** — IRRBB outlier test; EVE/Tier 1 and NII/Tier 1 limits per EBA/GL/2022/14
14. **Stress Testing** — Combined rate + credit + liquidity stress; capital adequacy under combined shock
15. **ALCO Pack** — Formatted summary for ALCO-style reporting; scenario comparison table

---

## Scenario Framework

Four scenarios are controlled from a single dropdown on the Dashboard. The selection writes to `Engine!C7`, which all downstream CHOOSE formulas reference. No sheet has an independent selector.

| Scenario | Description |
|---|---|
| Base | Stable rate environment with moderate loan and deposit growth |
| Rate Hike | +200bps shock with slower growth, higher deposit beta, and slightly weaker credit quality |
| Rate Cut | -200bps shock with stronger growth, lower deposit beta, and lower credit cost |
| Stress | +300bps stress with weak loan growth, flat deposits, high deposit beta, and materially higher credit cost |

---

## Regulatory Framework

| Metric | Standard | Reference |
|---|---|---|
| LCR | ≥ 100% | CRR2 / Delegated Regulation EU 2022/786 |
| NSFR | ≥ 100% | CRR Art. 428a–428at |
| IRRBB EVE | < 15% of Tier 1 | EBA/GL/2022/14 |
| IRRBB NII | Monitored | EBA/GL/2022/14 |
| CET1 | ≥ 8% (minimum) + buffers | CRR2 / Basel III |

---

## Key Design Decisions

**Single-source-of-truth control chain** — one scenario selector, one reference cell (`Engine!C7`), no independent dropdowns on downstream sheets. Changing the scenario cascades through NII, EVE, LCR, NSFR, and CET1 simultaneously.

**EVE hedge offset calibration** — a 55% hedge offset factor is applied to bring EVE/Tier 1 into a realistic range for a bank with BNP's hedging posture. Raw unhedged EVE produces outlier readings (~29%) that misrepresent actual exposure.

**NSFR trading book RSF** — trading assets are included with a blended weight (~20%) based on actual trading book composition. Omitting them understates RSF and inflates NSFR.

**Circular reference resolution** — the cash plug creates a circular dependency through NII → retained earnings → equity → total L+E → cash. Resolved by using the prior-year cash balance in the NII formula, eliminating the circularity without a workaround.

**Formula color conventions** — blue font for hardcoded inputs, black for formula cells, green (#006B42 / #00915A) for headers and cross-sheet references. The only acceptable hardcoded value is the 2024A NII actual in the NIM trend section.

---
## Model Validation

To ensure internal consistency and robustness, the model was tested across all scenarios with the following checks:

- Balance sheet identity verified across all forecast periods (Assets = Liabilities + Equity)
- Scenario selector tested across all 4 scenarios with full propagation through NII, EVE, LCR, NSFR, and CET1
- Key regulatory ratios recalculated after each scenario switch to confirm stability
- Monthly NII reconciled to annual NII within acceptable variance
- No hardcoded values in calculation chains; all outputs are formula-driven

## Data Source

Balance sheet actuals sourced from **BNP Paribas Q4 2025 Results Press Release** (verified public disclosure). All actuals are clearly labeled; forecast years are model-generated.

---

## Technical Specs

- **Platform:** Microsoft Excel (.xlsx)
- **Formula count:** 1,000+
- **Error count:** 0 (verified post-build)
- **External dependencies:** None (fully self-contained)
- **Recalculation:** Verified via LibreOffice after all programmatic writes

---

## Python Extensions (In Development)

This model is the foundation for a series of Python-based ALM tools that consume the Master Balance Sheet CSV as their primary data input:

| Project | Description | Depends on Model |
|---|---|---|
| IRRBB Stress Testing Framework | Python implementation of EBA/GL/2022/14 stress tests | Yes |
| Basel III LCR/NSFR Calculator | Regulatory calculator per EU 2022/786 and CRR | Optional |
| Yield Curve & NII Scenario Tool | Dynamic rate curve modeling and NII simulation engine | Yes |
| Full ALM Dashboard | Interactive Python/Dash ALM reporting interface | Yes |
| Bond Pricing & Duration Calculator | PVBP, DV01, convexity, modified duration | No |
| ECB Policy & Banking Impact Analyzer | Macro scenario overlay and policy transmission model | No |
| SQL Banking Sector Analysis | Sector benchmarking using public bank disclosures | No |

---

## Author

**Hossam Eltarrass**
ALM & Treasury Risk Analyst
[LinkedIn](https://www.linkedin.com/in/) | [GitHub](https://github.com/)

---

*Model built to bank-level standard for portfolio purposes. All regulatory references are to publicly available EBA guidelines and EU regulations.*
