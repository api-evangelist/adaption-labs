---
name: Estimate run cost and evaluate dataset quality
description: >-
  Quote the credit cost of an adaptation run before committing, then read the
  before/after quality metrics once a run has completed.
api: openapi/adaption-labs-datasets-openapi.yml
operations: [runDataset, getDatasetEvaluation, getDataset]
---

# Estimate run cost and evaluate dataset quality

Authenticate with `Authorization: Bearer pt_live_...`. Base URL
`https://api.prod.adaptionlabs.ai`.

## Estimate cost before running

1. Call `runDataset` with the `dataset_id` and `estimate: true`. No run starts.
2. Read `estimatedCreditsConsumed` and `estimatedMinutes` from the response to budget
   the job. If `multimodalPricingApplied` is true, image rows bill at 10 credits per
   100 output rows (`creditMultiplier`). `run_id` is null for estimate-only requests.
3. When ready, call `runDataset` again **without** `estimate` (or `estimate: false`) to
   start the real run.

## Evaluate quality after a run

1. Confirm the dataset finished with `getDataset` (or `getDatasetStatus`) — `status`
   should be `succeeded`.
2. Call `getDatasetEvaluation` with the `dataset_id`. Read `quality`:
   `grade_before` / `grade_after` (letter A-E), `improvement_percent`,
   `percentile_after`, `score_after` (0-10). `quality` is null until evaluation completes.

## Rules

- `estimate: true` validates column mapping and recipe config without consuming credits
  — use it to gate CI and budget jobs.
- Evaluation metrics are populated asynchronously; poll if `quality` is still null.
