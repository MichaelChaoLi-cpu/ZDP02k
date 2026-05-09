# ESG05h: Business Disclosure and Market Capitalisation

Mechanism study of how Business/Strategy ESG disclosure is associated with
market capitalisation among Japanese listed companies.

The project links fragment-level NLP scores from Japanese securities reports
with LSEG market capitalisation, fundamentals, GICS sector classifications, and
IBES analyst consensus data. The analysis follows the project SOP in
[docs/AnaSOP.md](docs/AnaSOP.md) and supports the manuscript draft
[docs/ZDP02k-v010.pdf](docs/ZDP02k-v010.pdf).

## Research Question

Through what channel does Business-related ESG disclosure affect market
capitalisation among Japanese listed companies?

This is a mechanism study rather than a pure application study. It tests whether
the valuation association operates through:

- information disclosure beyond observable firm quality;
- analyst attention and recommendation mediation;
- directional signal content, separating positive growth signals from risk
  acknowledgment;
- information-environment heterogeneity by firm quality, proxied by ROA
  quartile.

## Scope

- Universe: Japanese listed companies.
- Period: 2016-2025 annual panel.
- Panel size: 37,272 company-year observations before model-specific deletion.
- Main outcome: log annual market capitalisation.
- Main disclosure pillar: Business/Strategy topics.
- Estimator: two-way fixed effects panel OLS with company fixed effects, year
  fixed effects, and company-clustered standard errors.

The seven focal Business/Strategy topics are:

1. Competitiveness
2. Stakeholder Co-creation
3. Business Portfolio
4. Intellectual Capital
5. Digital Transformation
6. Financial Metrics
7. Corporate Value

## Data

Raw data are stored under `data/raw/` and the cleaned analysis panel is written
to `data/processed/panel.parquet`. The `data/` directory is Git-ignored and is
expected to be managed outside Git.

| Source | Files | Role |
| --- | --- | --- |
| EDINET securities reports + SAPT NLP pipeline | `match_scores.csv`, `sentiment_scores.csv`, `tendency_scores.csv` | Topic relatedness, document sentiment, and positive/negative tendency scores |
| LSEG Workspace | `Market_cap_annual.xlsx` | Annual market capitalisation |
| LSEG Workspace | `annual_financial_raw.xlsx` | ROA, leverage, R&D, revenue, assets, debt, equity |
| LSEG Workspace / IBES | `IBES_monthly_raw.xlsx` | Analyst EPS, revenue, recommendation, price target |
| MSCI/S&P via LSEG | `MSCI_category_Japan_listed_companies.xlsx` | GICS sector and industry classification |
| Japan interest rate data | `Japan_interest_rate_annual.xlsx` | Annual macro control/reference series |

## Variable Design

Each Business topic is estimated separately to avoid multicollinearity across
positively correlated topics. Disclosure variables are lagged by one year in the
baseline models; two-year lags are used for the reverse-causality robustness
check.

| Specification | Variables | Interpretation |
| --- | --- | --- |
| M1 | `match_{topic}_lag1` | Topic coverage volume |
| M2 | `match_{topic}_lag1` + `sentiment_mean_lag1` | Topic coverage plus overall document tone |
| M3 | `tend_{topic}_pos_mean_lag1` + `tend_{topic}_neg_mean_lag1` | Positive growth signal and negative/risk disclosure intensity |

Baseline model:

```text
logMC_it = company FE_i + year FE_t + f(disclosure_{i,t-1}) + error_it
```

Mechanism and robustness extensions add ROA/leverage controls, IBES
recommendation mediation, ROA-quartile splits, two-year disclosure lags,
alternative outcomes, COVID-year exclusion, balanced-panel restriction, and
winsorised growth outcomes.

## Main Findings

Current exported results show:

- M1 coverage is positive and statistically significant for six of seven
  Business topics; Digital Transformation has the largest baseline coefficient.
- Corporate Value coverage is negative in the baseline M1 model, but M3 shows a
  positive risk-transparency component and a negative positive-mean component.
- ROA and leverage controls do not explain away the M1 association. Attenuation
  ranges from 5.7% to 16.9%, and all seven controlled coefficients remain
  statistically significant.
- IBES mediation is modest. The match coefficient attenuation after adding
  analyst recommendation is generally small, from about 0.6% to 8.6% for the
  positive M1 topics.
- M3 separates the mechanism by topic: Business Portfolio, Intellectual Capital,
  and Financial Metrics follow a bilateral signalling pattern; Competitiveness,
  Stakeholder Co-creation, and Digital Transformation show growth-signal/risk
  penalty patterns; Corporate Value is classified as risk-transparency
  dominant.
- The two-year lag check preserves the baseline sign and statistical
  significance for all seven topics, strengthening the argument against simple
  contemporaneous reverse causality.

These estimates should be interpreted as within-company conditional
associations, not as definitive causal effects.

## Reproduction

Create an environment with Python 3.12 and install the project dependencies.
The current scripts require at least `numpy`, `pandas`, `openpyxl`, `pyarrow`,
`matplotlib`, `statsmodels`, and `pyyaml`.

```bash
python3 -m venv .venv
source .venv/bin/activate
python -m pip install --upgrade pip
python -m pip install -r requirements.txt pandas statsmodels pyyaml
```

Run the pipeline from the repository root:

```bash
# 1. Build the cleaned panel
python src/01_data_cleaning.py

# 2. Generate the analysis notebook skeleton
python src/03_analysis.py

# 3. Regenerate figures, tables, metadata, and Jiazi export artifacts
python src/90_export.py
```

Expected primary output:

```text
data/processed/panel.parquet
export/
  figures/
  tables/
  metadata/
  code/
  nbs/
  AnaSOP.md
  actionbrief.yaml
```

## Repository Structure

```text
ESG05h/
  src/
    01_data_cleaning.py       # raw data merge and panel construction
    02_data_inspection.py     # notebook generator for data inspection
    03_analysis.py            # notebook generator for mechanism analysis
    90_export.py              # full export pipeline
    vardict.py                # topic and variable dictionaries
  data/
    raw/                      # local raw data, Git-ignored
    processed/                # local processed panel, Git-ignored
  docs/
    AnaSOP.md                 # analysis standard operating procedure
    ZDP02k-v010.pdf           # manuscript draft
    LAWS/                     # export/actionbrief schemas
  export/
    figures/                  # final figures
    tables/                   # final regression/descriptive tables
    metadata/                 # dataset and variable dictionaries
    code/                     # exported reproducible scripts
    actionbrief.yaml          # Jiazi ingestion metadata
```

## Key Artifacts

- [docs/AnaSOP.md](docs/AnaSOP.md): research objective, theory, variable
  construction, estimands, identification strategy, and workflow.
- [export/actionbrief.yaml](export/actionbrief.yaml): machine-readable summary
  of datasets, variables, tables, figures, and analysis structure.
- [export/tables/Table2_CrossTopic_Summary.xlsx](export/tables/Table2_CrossTopic_Summary.xlsx):
  baseline M1/M2/M3 coefficient matrix.
- [export/tables/Table3_Mechanism_InfoChannel.xlsx](export/tables/Table3_Mechanism_InfoChannel.xlsx):
  fundamentals-control attenuation test.
- [export/tables/Table4_Mechanism_IBESMediation.xlsx](export/tables/Table4_Mechanism_IBESMediation.xlsx):
  analyst recommendation mediation test.
- [export/tables/Table5_Mechanism_SignalDirection.xlsx](export/tables/Table5_Mechanism_SignalDirection.xlsx):
  M3 positive/negative signal classification.
- [export/tables/Table7_IdentificationRobustness.xlsx](export/tables/Table7_IdentificationRobustness.xlsx):
  t-2 lag and standard robustness checks.

## Notes

- IBES covers roughly one third of the panel and is concentrated among larger
  firms with analyst coverage; mediation results should not be read as fully
  representative of the full sample.
- R&D coverage is limited, so R&D is used selectively rather than as a full
  baseline control.
- Utilities and Energy have small sector samples; sector heterogeneity results
  for those groups require caution.
