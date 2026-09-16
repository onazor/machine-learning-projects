# ABC Supermarket Campaign Analysis

Can ABC Supermarket find more likely gold-membership buyers while calling fewer customers?

This project analyzes the results of a previous membership campaign and builds a classification-based customer prioritization strategy. The final model ranks customers by their likelihood of responding so the campaign team can match its calling list to available capacity.

[View the analysis notebook](main.ipynb) | [View the presentation](presentation/abc-campaign-analysis.pdf) | [View the problem statement](data/Problem%20Statement.pdf)

## Business problem

ABC plans to offer an annual gold membership for $499, discounted from its normal $999 price. Calling the entire customer base would be costly, particularly when only a small share of customers historically accepted the offer.

The analysis addresses three questions:

1. Which customer characteristics are associated with campaign response?
2. Which model best distinguishes and ranks likely responders?
3. How can the model support a practical calling strategy?

## Dataset

The source workbook contains 2,240 customer records and 22 columns. The target, `Response`, indicates whether a customer accepted the previous campaign offer.

Key data characteristics:

- 334 customers responded, for a historical response rate of 14.9%.
- 24 income values are missing and are imputed inside the modeling pipeline.
- Three implausible age records and one extreme income record are excluded, leaving 2,236 rows for modeling.
- The final split contains 1,788 training rows and 448 test rows, stratified by response.

## Approach

The notebook follows an end-to-end classification workflow:

1. Inspect data quality and explore response patterns.
2. Engineer customer age and tenure features.
3. Apply median imputation and scaling to numeric fields, plus most-frequent imputation and one-hot encoding to categorical fields.
4. Compare logistic regression, decision tree, random forest, K-nearest neighbors, gradient boosting, and a dummy baseline.
5. Tune the strongest candidates with five-fold stratified cross-validation using average precision as the primary search metric.
6. Compare Extra Trees, soft voting, and stacking ensembles.
7. Rank customers by model score and evaluate different campaign capacities using capture, response rate, and lift.

Preprocessing is fitted only on the training data through scikit-learn pipelines to reduce leakage risk.

## Results

The soft-voting ensemble, which combines logistic regression, random forest, and gradient boosting, provided the best overall balance.

| Metric | Test result |
| --- | ---: |
| Recall | 0.761 |
| F1 score | 0.630 |
| ROC AUC | 0.907 |
| Average precision | 0.624 |
| Cross-validated average precision | 0.549 |

At the default 0.50 threshold, the model identified 51 of 67 responders and produced 44 false-positive calls.

Ranking customers is more useful than relying on a single threshold:

| Share contacted | Customers called | Responders found | Responder capture | Response rate | Lift |
| ---: | ---: | ---: | ---: | ---: | ---: |
| 10% | 45 | 27 | 40.30% | 60.00% | 4.01x |
| 20% | 90 | 49 | 73.13% | 54.44% | 3.64x |
| 30% | 134 | 59 | 88.06% | 44.03% | 2.94x |

## Recommendation

Start with the top 20% of customers ranked by the ensemble score. In the held-out sample, this group contained 73.1% of all observed responders. If additional calling capacity is available, expanding to the top 30% raises responder capture to about 88%, although response rate and lift decline.

This policy should be tested through a controlled pilot. A model-selected group and a randomly selected group should receive the same offer, script, budget, and observation window. The final decision should use incremental conversions and net contribution, not gross membership revenue alone.

## Project structure

```text
campaign-analysis/
├── data/
│   ├── marketing_data.xlsx
│   └── Problem Statement.pdf
├── presentation/
│   └── abc-campaign-analysis.pdf
├── main.ipynb
└── README.md
```

## Run the notebook

From this project directory:

```bash
python -m venv .venv
source .venv/bin/activate
python -m pip install jupyter pandas matplotlib seaborn scikit-learn openpyxl
jupyter notebook main.ipynb
```

Run the notebook from the project directory so the relative path `data/marketing_data.xlsx` resolves correctly.

## Limitations

- Results come from one historical campaign and need validation on fresh data.
- Associations in the exploratory analysis do not establish causation.
- Repeated inspection of the test set makes the current results exploratory.
- Model scores should be calibrated before being treated as literal purchase probabilities.
- Feature timing, drift, contact costs, discount costs, and customer-level net value should be validated before deployment.
