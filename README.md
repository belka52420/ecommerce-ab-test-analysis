# E-commerce A/B test: should the new page launch?

An independent practice project evaluating a new landing page using conversion data. The analysis combines experiment-data validation, user-level metrics and a sensitivity check to support a rollout decision.

**Result:** the available data does not support launching the new page on the claim that it increases conversion. This is not evidence that the two versions are equivalent.

[Read the executed notebook](ecommerce_ab_test.ipynb) — analysis and explanations in Russian; summary below in English.

## Business question

Does the new page increase conversion? What can we conclude when group assignment and page exposure do not always match?

This is an educational dataset analysis, not a production experiment I ran at Yandex. No actual revenue impact or prevented loss is claimed.

## Data validation

- 294,480 source records in `ab_data.csv`.
- 3,893 rows where group assignment does not match the page shown.
- 1,895 users appearing in both groups **before** filtering mismatched rows.
- One identical duplicate in `countries.csv`, removed before a validated many-to-one join. All source users have a country match.
- User-level conversion is defined as at least one converted record among retained observations.

The primary specification reproduces the original cleaning order: remove mismatched group/page rows, then aggregate by user. A second specification excludes all users seen in both groups in the raw data before applying the page filter. Neither rule automatically recovers an unbiased causal estimate without the assignment and logging protocol.

## Results

| Specification | Users | Control | New page | Difference (percentage points) | One-sided p-value |
|---|---:|---:|---:|---:|---:|
| Original cleaning order | 290,585 | 12.039% | 11.881% | −0.158 | 0.905 |
| Exclude raw cross-group users first | 288,690 | 12.030% | 11.883% | −0.147 | 0.889 |

The approximate two-sided 95% confidence interval for the primary conversion difference is **−0.394 to +0.078 percentage points**. The sensitivity interval is **−0.384 to +0.089 percentage points**.

The increase test uses a pooled two-proportion z-test (H1: treatment > control, alpha = 0.05). Confidence intervals use an unpooled normal approximation and are two-sided. Country-level tests are exploratory; they do not establish zero or identical effects across countries. The retrospective MDE calculation is a sensitivity benchmark, not a claim about the original experiment's power.

## Recommendation

The analysis does not provide evidence of increased conversion under either cleaning specification. Before a new experiment:

1. Clarify random assignment, page-exposure logging and the conversion window.
2. Agree on the smallest commercially worthwhile improvement and guardrail metrics.
3. Set the sample size and stopping rule in advance; do not extend the test merely until significance appears.

The key analytical contribution is separating a decision supported by the data from claims the data cannot establish.

## Reproduce

Tested with Python 3.11. The notebook contains saved outputs and can be read without running it.

```bash
python3.11 -m venv .venv
source .venv/bin/activate
python -m pip install -r requirements.txt
python -m ipykernel install --user --name ecommerce-ab --display-name "E-commerce A/B"
jupyter notebook ecommerce_ab_test.ipynb
```

Select the **E-commerce A/B** kernel and run all cells from the repository directory. On Windows, activate the environment using `.venv\Scripts\activate`.

## Files and scope

- `ecommerce_ab_test.ipynb` — executed analysis, data checks, chart, tests and sensitivity comparison.
- `ab_data.csv`, `countries.csv` — supplied educational data; not internal company data. The original source URL and dataset license were not included with the local files; no ownership of the dataset is claimed.
- `requirements.txt` — package versions used to execute the analysis.

The randomization protocol, stopping rule, observation window and business threshold were not provided. These limitations constrain causal and rollout conclusions even with a large sample.
