# Capstone Report — Refresh / Content Opportunity Scoring

- **Author:** Samarth Dubbas
- **Lane:** Refresh / Content Opportunity Scoring
- **Repo:** https://github.com/SamarthD07/flyrank-ml-internship
- **Date:** 2026-08-26

## 0. Abstract

This project investigates whether pre-March search-performance signals can
support prioritization of content pages for SEO review and refresh decisions.
The analysis uses February 2026 page-level GSC signals as features and March
2026 GSC clicks as the outcome window. A transparent rule-based baseline is
compared with a machine-learning regression model using GSC impressions, GSC
clicks, and average position. Using a client-held-out evaluation with 40
training clients and 10 test clients, the model achieved an MAE of 3.1076 and
an R² of 0.3455, while its top-100 ranking had a mean March click count of
122.85 compared with 16.51 for the baseline. The resulting ranking is intended
as decision support for prioritizing human review, not as evidence that
refreshing a page will causally increase traffic.

## 1. Problem framing

This project supports the decision of which content pages should receive
priority for review when editorial or SEO review capacity is limited.

The unit of analysis is a client-content page aggregated across the February
2026 feature window. The output is a ranked page-level score and recommendation
queue. A FlyRank editor can use the ranking to decide which pages to inspect
first rather than treating the score as an automatic refresh decision.

A wrong call can waste editorial effort on a page that does not need attention,
or cause a genuinely valuable page to be overlooked.

Data and ML are useful because the warehouse contains large numbers of
client-content observations and historical search-performance signals that can
be combined consistently into a ranked decision-support queue.

## 2. Data safety

The analysis uses the FlyRank internship warehouse
`fact_content_daily_performance`.

February 2026 (2026-02-01 to 2026-02-28) is used as the feature window and
March 2026 (2026-03-01 to 2026-03-31) as the outcome window.

The verified February feature data contains 7,355,108 observations and the
March outcome data contains 9,841,378 observations before page-level
aggregation.

The main model features are:

- `gsc_impressions`
- `gsc_clicks`
- `gsc_avg_position`

Client and content identifiers are used for grouping, joining and traceability,
not as predictive features.

Traffic-source and AI-referral fields were excluded from the initial model
because they were not necessary for the first version of the analysis and may
contain sparse or zero-heavy signals.

Future March outcome information is not used as a model input. Product flags
are also excluded from the scoring rule.

The analysis uses pseudonymous client/content identifiers and does not include
client names, private queries, credentials, or other client-identifying
information in the work.

## 3. Baseline

The baseline is a transparent rule-based score using February GSC signals.

The score assigns:

- +2 points when GSC impressions are greater than zero
- +2 points when GSC clicks are greater than zero
- +1 point when average GSC position is 20 or better

The maximum baseline score is therefore 5.

This provides a simple and interpretable comparison because the score uses
pre-outcome information, has no fitted weights, and can be reproduced directly
from the documented conditions.

The baseline was applied to the February feature data and produced the
required ranked baseline queue.

The baseline output was generated as:

`work/outputs/baseline_action_score.csv`

The baseline is intentionally simple so that differences between the
machine-learning ranking and the baseline can be interpreted against an
explicit, human-readable rule.

## 4. Model / analysis

The capstone model uses a machine-learning regression approach to estimate
future March GSC click activity from pre-March February search-performance
signals.

The exact model features are:

- `gsc_impressions`
- `gsc_clicks`
- `gsc_avg_position`

The target is:

**March aggregate GSC clicks for the same client-content page.**

The model uses only February information as input, while March is reserved for
the outcome period.

The model is a `HistGradientBoostingRegressor` with a fixed random seed of 42.
It was selected as a practical nonlinear regression method for capturing
relationships between search visibility, clicks, and position.

Client and content IDs are used only for grouping, joining, and traceability;
they are not predictive features.

The model output is used to rank pages by estimated future March click
activity. The resulting ranking is treated as decision support for review
prioritization rather than as proof that a page should be refreshed or that a
refresh will cause improved performance.

## 5. Evaluation

The evaluation uses a client-held-out validation design. Unique clients were
randomly divided using a fixed seed of 42, with 40 clients used for training
and 10 clients held out for testing. This prevents pages belonging to the same
client from appearing in both the training and test sets.

The model and transparent baseline are evaluated on the same held-out client
set.

### Model performance

- Training clients: **40**
- Test clients: **10**
- Training rows: **241,186**
- Test rows: **62,386**
- MAE: **3.1076**
- R²: **0.3455**

### Ranking comparison

For the top-100 comparison:

- K: **100**
- Baseline top-100 mean March clicks: **16.51**
- Model top-100 mean March clicks: **122.85**
- Baseline/model top-100 overlap: **0**
- Overlap rate: **0%**

The March positive-click base rate on the held-out evaluation set was:

- Pages with more than zero March clicks: **34.45%**

The model-selected top 100 therefore had a substantially higher observed mean
March click count than the baseline-selected top 100 in this held-out
evaluation. This is a ranking result, not a causal estimate of traffic
improvement.

The zero-page overlap also indicates that the model ranking was materially
different from the transparent baseline.

## 6. Interpretation

The analysis indicates that February search-performance signals contain useful
information about subsequent March search activity, although the predictive
performance is moderate rather than definitive.

The strongest ranked pages generally have substantial February search
visibility, existing clicks, and relatively strong average positions. This
means the model is identifying pages that already have measurable search
presence and may therefore be useful candidates for closer editorial or SEO
review.

The machine-learning ranking is not simply a copy of the transparent
baseline. In the held-out evaluation, the top-100 rankings had zero page
overlap. The model-selected top 100 had a mean of 122.85 March clicks,
compared with 16.51 for the baseline-selected top 100.

A key interpretation limit is that high predicted March click activity does
not necessarily mean that a page is stale or requires refreshing. Some highly
ranked pages may already be performing well. The ranking should therefore be
combined with editorial context, content quality review, search intent,
business importance, and other information not included in the model.

The analysis does not claim to predict Google's ranking algorithm or establish
that any specific content change caused an improvement.

## 7. Recommendation

The output should be used as a prioritized review queue rather than an
automatic action list.

Recommended workflow for a FlyRank editor:

1. Start with the highest-ranked pages from the model.
2. Inspect February impressions, clicks, and average position.
3. Check whether the page has an identifiable content or search opportunity.
4. Review search intent, content quality, freshness, and business relevance.
5. Only then decide whether a refresh, rewrite, consolidation, or no action is
   appropriate.
6. Treat lower-confidence or weak-signal pages as monitor/review candidates
   rather than guaranteed opportunities.

The generated recommendation artifact is:

`work/outputs/capstone_ranked_recommendations.csv`

The recommendation queue contains the model score, action label, reason code,
February search signals, and observed March clicks for evaluation.

Confidence should be considered moderate rather than definitive because the
analysis uses one February feature window and one March outcome window and does
not contain many factors that influence organic search performance.

## 8. Reproducibility

The main analysis is contained in:

`work/notebooks/capstone.ipynb`

The evaluation uses a fixed random seed of 42 for the client-held-out split
and the machine-learning model.

The analysis uses the FlyRank internship warehouse
`fact_content_daily_performance`, with February 2026 as the feature window and
March 2026 as the outcome window.

The generated analysis artifacts include:

- `work/outputs/baseline_action_score.csv`
- `work/outputs/capstone_ranked_recommendations.csv`
- `work/outputs/capstone_metrics.csv`

The notebook should be run from top to bottom in a fresh Colab runtime to
reproduce the analysis. The February feature data and March outcome data are
loaded from the internship warehouse using the configured Hugging Face
credential.

The evaluation uses a client-held-out split with 40 training clients and 10
held-out clients. The resulting evaluation metrics were:

- MAE: **3.1076**
- R²: **0.3455**
- Top-100 baseline mean March clicks: **16.51**
- Top-100 model mean March clicks: **122.85**
- Top-100 overlap: **0**
- March positive-click rate: **34.45%**

No future March outcome fields are used as model features.

## 9. Acknowledgments & data credit

Built on the FlyRank ML Internship dataset,
[FlyRank](https://flyrank.ai).
