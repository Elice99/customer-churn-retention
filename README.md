# Customer Churn Prediction & Retention Dashboard

**One telecom churn problem, two deliverables built on the same 7,043-customer dataset:** a scikit-learn model that scores every customer's churn probability and serves it through a FastAPI endpoint, and an Excel executive dashboard that shows where the risk lives and what it is worth.

![Status](https://img.shields.io/badge/status-complete-success)
![Model](https://img.shields.io/badge/model-logistic%20regression-blue)
![Serving](https://img.shields.io/badge/serving-FastAPI-009688)
![Dashboard](https://img.shields.io/badge/dashboard-Excel%20%7C%20PivotTables-217346)

---

## TL;DR

I took the classic telco churn dataset (7,043 customers, 21 attributes, 26.54% churn rate) and built the full path from question to action. The model ranks customers by churn probability at a test ROC-AUC of **0.858**, validated with 5-fold cross-validation at **0.842 ± 0.007** and served through a FastAPI `/predict` endpoint. The dashboard turns the same data into one screen for leadership: **1,869 customers at risk**, **$139,131 of monthly billing** and **$2.86M of lifetime billing** in that segment, with the risk concentrated in month-to-month contracts, electronic-check payments and low adoption of protective add-on services.

The two halves answer different questions about the same problem: *who is next to leave* (the model) and *where should the retention budget go* (the dashboard).

---

## Results

| Metric | Value | How it was validated |
|--------|-------|----------------------|
| Test ROC-AUC | **0.858** | Single scoring of a held-out 1,409-customer test set |
| Cross-validation AUC | **0.842 ± 0.007** | 5-fold CV on the 5,634-customer training set |
| Churn rate | **26.54%** (1,869 of 7,043) | Direct profile of the raw CSV |
| Strongest segment split | **42.7% vs 2.8%** churn | Month-to-month vs two-year contracts |
| Revenue at risk | **$139,131/month** | Sum of MonthlyCharges for the churned group |

The model pipeline is deterministic (fixed `random_state`), so every number in this README can be reproduced by running `model/Churn.py`.

## Repository structure

```
customer-churn-retention/
├── data/                        # Telco-Customer-Churn.csv (7,043 rows, 21 columns)
├── model/                       # the prediction half
│   ├── Churn.py                 # full training pipeline: cleaning, split, 5-fold CV, final fit, pickle
│   ├── predict.py               # FastAPI /predict endpoint (Swagger UI included)
│   ├── classification.ipynb     # exploratory notebook
│   ├── model_C=1.0.bin          # pickled DictVectorizer + LogisticRegression (ships together)
│   ├── customer.json            # sample request payload
│   └── requirements.txt         # pandas, numpy, scikit-learn, fastapi
├── dashboard/
│   └── Customer Retention.xlsx  # 5-sheet workbook: raw data, pivot layer, dashboard, risk analysis
├── docs/
│   ├── BUSINESS_README.md       # the business case, segments and use cases
│   ├── TECHNICAL_README.md      # pipeline, validation, serving, known limitations
│   └── QUICK_REFERENCE.md       # one-page cheat sheet
└── screenshots/
    ├── retention_dashboard.png
    └── api_swagger.png
```

## The two halves

### 1. Prediction model (`model/`)

Logistic regression (C=1.0, max_iter=5000) on 3 numeric + 16 one-hot encoded categorical features, with the DictVectorizer and model pickled together so serving cannot drift from training.

```bash
cd model
pip install -r requirements.txt
python Churn.py        # trains, cross-validates, writes model_C=1.0.bin
python predict.py      # serves on http://localhost:9696, docs at /docs
```

Sample request:

```bash
curl -X POST http://localhost:9696/predict \
  -H "Content-Type: application/json" \
  -d "{\"tenure\": 2, \"monthly_charges\": 34.56, \"total_charges\": 28.45, \"contract\": \"month-to-month\"}"
```

**Known limitation, stated plainly:** the demo schema above accepts 4 fields while the model trained on 19; missing features encode as zeros. Expand `CustomerInput` in `predict.py` to the full feature contract before real integration.

### 2. Executive dashboard (`dashboard/`)

The Excel workbook is built on a three-layer architecture: raw data sheet, a PivotTable layer that computes every count, share and total, and a dashboard sheet where every card and chart reads pivot output (nothing typed). Contract, gender and internet-service slicers filter the whole screen in one click.

One labelling note I keep on the record: the dashboard's "Yearly Charges" card ($2,862,927) sums the *cumulative lifetime* charges of the churned customers. The honest forward-looking figure is the $139,131 monthly run-rate (~$1.67M annualised).

## What the data says

- **Contract type is the ground truth**: month-to-month customers churn at 42.7%, one-year at 11.3%, two-year at 2.8%.
- **Payment friction is a churn signal**: electronic-check payers churn at 45.3% vs 15.2% for automatic credit card.
- **The premium tier churns hardest**: fiber optic 41.9% vs DSL 19.0%.
- **Churners pay more**: $74.44 average monthly bill vs $64.76 across the base.
- **The risk window is early**: 29.4% of all customers are inside their first 12 months.

## Documentation

- [`docs/BUSINESS_README.md`](docs/BUSINESS_README.md) — business problem, segments, use cases, financial scenarios
- [`docs/TECHNICAL_README.md`](docs/TECHNICAL_README.md) — pipeline, validation strategy, serving architecture, known issues
- [`docs/QUICK_REFERENCE.md`](docs/QUICK_REFERENCE.md) — one-page summary

## Author

Built by [Elice Bassey](https://github.com/Elice99) as part of the ML Journey. The same investigation is presented as two case studies on [my portfolio](https://elishabassey.netlify.app): Customer Churn Prediction and Customer Retention Dashboard.
