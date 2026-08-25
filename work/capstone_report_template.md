# Capstone Report — Refresh / Content Opportunity Scoring

- **Author:** Samarth Dubbas
- **Lane:** Refresh / Content Opportunity Scoring
- **Repo:** https://github.com/SamarthD07/flyrank-ml-internship
- **Date:** 2026-08-26

## 0. Abstract

<!-- Write this last after Sections 1–9 are complete. -->

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

The baseline was applied to 7,355,108 February observations and produced the
required ranked baseline queue.

The baseline output was saved as:

`work/outputs/baseline_action_score.csv`

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

The approach was selected because the goal is to produce a page-level ranking
that can support prioritization while remaining interpretable enough for
decision-support use.

Client and content IDs are not predictive features.
