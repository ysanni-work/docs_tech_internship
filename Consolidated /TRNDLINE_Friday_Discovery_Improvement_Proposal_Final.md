# TRNDLINE Insight — Friday Discovery & Improvement Proposal

**Branch / commit inspected:** `main` at `54c4b236`


# 1.  Summary

Friday's task was to move from:

> **How does Insight work?**

to:

> **What is actually preventing Insight from becoming materially better?**


My strongest conclusion is that the current evidence does **not** justify replacing logistic regression with GBT or LightGBM as the primary recommendation. Before model sophistication can be compared fairly, there are several upstream problems in data causality, evaluation, artifact measurement, and serving that can make performance numbers misleading.

When I staeted investigating I was of the opinion that the model had to be changed... but I am still trying to see what decisive direction to take

The most important finding is a **confirmed historical higher-timeframe look-ahead vulnerability** in the systematic dataset builder. I changed only future HTF candles while keeping the historical base data fixed. All 8/8 historical observations changed. This demonstrates that the current offline historical feature path can use information that was not available at the observation time.

I also found a second point-in-time concern in calibration: historical `asOf` filtering limits the setup timestamp but does not prove that the outcome had matured by that time.

Beyond data causality, there are several evaluation/serving inconsistencies:

- GBT artifacts are stored with metrics calculated from logistic predictions rather than GBT predictions.
- Python LightGBM training produces booster text that the verified Node scoring path does not consume.
- Global retraining calculates release metrics from rows that omit publication/EV context required by several release gates.
- Global retraining and ablation do not preserve the canonical partition identity as an immutable untouched holdout.
- Missing base inputs are often converted to normal-looking zero/default values before missingness is recorded.
- Family training writes shadow models, but I did not find an automatic promotion writer in the verified path.
- The router can silently leave the global model in control when exact-family serving fails.
- The local database did not contain enough observations/models to answer real dataset balance, segment performance, or best-model questions.

My current direction is therefore:

> **First make the historical data, evaluation cohort, model-specific metrics, and serving lineage trustworthy. Only then decide whether features or the model class should change.**

The highest expected-impact first implementation is a **causal evaluation foundation**:

1. enforce per-anchor base and HTF boundaries;
2. enforce calibration outcome maturity at historical `asOf`;
3. preserve immutable cohort/partition identities;
4. score each model with its own scorer on the same untouched cohort;
5. make serving/artifact compatibility explicit and observable.

Only after this should the team compare global logistic, family logistic, GBT, and LightGBM as actual challengers.
