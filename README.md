# China Prestige Skincare TAM Sizing — Commercial Due Diligence Simulator

**Business problem:** A PE fund evaluating an acquisition of a mid-size prestige
skincare brand entering China needs a defensible Total Addressable Market (TAM)
estimate that survives partner scrutiny — built bottom-up from population, income,
and category-spend data, rather than a single number cited from a market-research
report. This project simulates that diligence exercise end-to-end.

## Key Findings

- **Base-case TAM is roughly two orders of magnitude larger than base-case SOM
  (Serviceable Obtainable Market)** — the number a PE fund should actually underwrite
  is the SOM figure, not the market-size headline that typically anchors early deal
  conversations.
- **Monte Carlo simulation (10,000 runs) puts the underwritable SOM range at the P10–P90
  confidence band reported in `outputs/cdd_summary_onepager.html`**, not a single point
  estimate — the range itself is the deliverable, not a false-precision midpoint.
- **Tornado sensitivity analysis identifies which single assumption drives the most
  swing in outcome** (see `outputs/tornado_sensitivity.html`) — this is the variable a
  real diligence engagement should prioritize stress-testing first, rather than treating
  all five underlying assumptions as equally uncertain.

## Business Recommendation

Anchor the acquisition thesis on the SOM range, not the TAM headline. Prioritize
diligence resources on validating the single most sensitive assumption identified by
the tornado analysis — if that assumption can be tightened with primary research
(management interviews, expert calls, category data licensing), the SOM range
compresses meaningfully and the deal becomes easier to underwrite with confidence.

## Approach

| Step | Method | Why |
|---|---|---|
| Macro data | World Bank Open Data API (population, GDP per capita, urbanization, consumption) | Free, reproducible, no manual entry |
| Serviceable geography | Top 10 provinces by 2020 National Population Census | Covers ~65% of national population, skewed toward coastal/urban prestige-skincare demand |
| Category assumptions | Low/base/high ranges with explicit sourcing notes | Stands in for licensed Euromonitor/Statista data a real engagement would use; transparency about the substitution is the point |
| TAM → SAM → SOM (Notebook 02) | Three discrete scenarios, all inputs moved together | Simple, defensible first pass |
| Sensitivity (Notebook 03) | Monte Carlo (10,000 sims), each input varied independently via triangular distribution | More realistic than scenario-based ranges; produces a proper distribution and identifies the dominant driver via tornado chart |

## Model Decisions

- **Triangular distribution for Monte Carlo sampling**, not normal — chosen because
  each assumption's low/base/high already defines a natural triangular shape (with
  "base" as the mode), and it avoids the normal distribution's implicit assumption of
  symmetric uncertainty, which none of these five inputs actually have.
- **All five assumptions treated as independent in the simulation** — a simplification.
  In reality, penetration rate and spend-per-consumer likely correlate. See limitations
  below.
- **`fig.write_html()` used for all charts**, not static image export — `kaleido`-based
  PNG export is unreliable in Codespaces; HTML output also makes the tornado and
  distribution charts interactive when opened directly, which is a better artifact for
  a portfolio review than a static image.

## Repository Structure
china-prestige-skincare-tam-sizing/
├── data/
│   ├── raw/                                    # (gitignored — no raw caching needed; API-sourced)
│   └── processed/
│       ├── china_macro_indicators.csv
│       ├── province_population_reference.csv
│       ├── category_assumptions.csv
│       ├── tam_sam_som_scenarios.csv
│       ├── monte_carlo_simulations.csv
│       └── tornado_sensitivity.csv
├── notebooks/
│   ├── 01_data_collection_and_assumptions.ipynb
│   ├── 02_bottom_up_tam_triangulation.ipynb
│   └── 03_sensitivity_and_scenario_output.ipynb
├── outputs/
│   ├── province_population_ranking.html
│   ├── tam_sam_som_funnel.html
│   ├── som_distribution.html
│   ├── tornado_sensitivity.html
│   └── cdd_summary_onepager.html               # standalone executive report
├── src/
├── requirements.txt
└── README.md

## Data Source

- World Bank Open Data API: `https://api.worldbank.org/v2/country/CHN/indicator` — no
  API key required, indicators: `SP.POP.TOTL`, `NY.GDP.PCAP.CD`, `SP.URB.TOTL.IN.ZS`,
  `NE.CON.PRVT.PC.KD`
- China's 7th National Population Census (2020), National Bureau of Statistics —
  provincial population figures, entered as a static reference table (census data
  doesn't update annually; NBS's own site isn't reliably scrapable)
- Category-specific assumptions (penetration rate, spend per consumer, category growth,
  addressable channel share, achievable market share) are directional estimates, **not**
  licensed data — explicitly stated as such throughout, with low/base/high ranges rather
  than point figures

## What I Would Do With More Time

- **License actual category data** (Euromonitor or Statista prestige beauty category
  reports) to replace the directional assumption ranges with sourced figures — the
  current ranges are defensible as a methodology demonstration but wouldn't stand up
  in an actual client engagement without real data behind them.
- **Model correlation between assumptions** rather than treating all five as
  independent in the Monte Carlo — penetration rate and spend-per-consumer plausibly
  move together, and a correlated simulation would likely narrow the plausible range
  compared to the current (probably too-wide) independent-variable version.
- **Consolidate all five assumptions into one source-of-truth table.** Two of the five
  (`addressable_channel_share`, `achievable_share`) were defined inline in Notebook 02
  rather than saved to `category_assumptions.csv` alongside the other three — a minor
  inconsistency that should be refactored so every assumption lives in one place and
  every notebook reads from it identically.
- **Extend the provincial breakdown below the top-10 aggregate** — the current model
  treats all 10 provinces as one pooled serviceable geography; a more granular version
  would model province-level penetration and spend differences (Guangdong and Shanghai
  plausibly have meaningfully higher prestige-skincare penetration than, say, Henan),
  which would sharpen both the TAM estimate and the market-entry sequencing recommendation.
