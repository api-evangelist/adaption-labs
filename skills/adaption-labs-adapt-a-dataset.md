---
name: Adapt a dataset with Adaptive Data
description: >-
  Ingest a local file into Adaption, run an augmentation/adaptation pipeline with
  brand and quality controls, wait for it to finish, and download the processed
  rows. Uses the presigned-upload ingest path.
api: openapi/adaption-labs-datasets-openapi.yml
operations: [createDataset, completeUploadById, runDataset, getDatasetStatus, downloadDataset]
---

# Adapt a dataset with Adaptive Data

Authenticate every request with `Authorization: Bearer pt_live_...` (create a key in
the Adaption app under Settings -> API keys). Base URL `https://api.prod.adaptionlabs.ai`.

## Steps

1. **Create the dataset (file source).** Call `createDataset` with
   `source: { type: "file", file_format: "csv", name: "my-dataset" }`. The response
   returns `dataset_id` and `upload_instructions` (a presigned S3 `url`, `method`, `s3_key`).
2. **Upload the bytes.** PUT the file directly to the presigned `url` (this is an S3
   request, not an Adaption API call).
3. **Complete the upload.** Call `completeUploadById` with the `dataset_id`, the
   `file_size_bytes`, and optionally the `sha256` digest. This verifies the file and
   triggers preprocessing.
4. **Start the run.** Call `runDataset` with the `dataset_id`. Set a `column_mapping`
   (at minimum a `prompt` column) and optional `brand_controls`
   (`length`, `safety_categories`, `blueprint`, `hallucination_mitigation`). Optionally
   set `estimate: true` first to get a cost quote without starting the run.
5. **Wait for completion.** Poll `getDatasetStatus` until `status` is `succeeded`
   (or `failed`). Read `progress` for percent/rows. On failure inspect `error_data`
   (`code`, `level`, `message`).
6. **Download the output.** Call `downloadDataset` with the `dataset_id` and optional
   `fileFormat`. It streams the processed rows.

## Rules

- Runs and imports are **asynchronous** — never call `downloadDataset` before status
  is `succeeded` (it returns **422** if no run has ever started).
- Errors carry a stable `code` (e.g. `E0100`), a `level`, and a `message` on
  `error_data`. See `errors/adaption-labs-problem-types.yml`.
- No idempotency key is supported; do not blindly retry `runDataset` (it reserves credits).
