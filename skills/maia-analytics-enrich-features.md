---
name: Enrich MAIA features and add to a layer
description: Create a data-enrichment job for a project, run it asynchronously, poll to completion, and add the enriched results to a layer.
api: openapi/maia-analytics-openapi-original.json
operations:
  - create_enrichment_api_v1_enrichment_create_post
  - enrich_row_async_api_v1_enrichment_enrich_async__enrichment_id__post
  - get_enrichment_status_api_v1_enrichment_workflow_status__workflow_id__get
  - add_enrichment_to_layer_api_v1_enrichment_add_to_layer_post
---

# Enrich MAIA features and add to a layer

Use this skill to enrich a MAIA project's features with additional data and fold
the results back into a layer.

## Auth
All calls require `Authorization: Bearer <firebase_id_token>`
(`FirebaseAuthMiddleware`). See `authentication/maia-analytics-authentication.yml`.

## Steps
1. **Create the enrichment.** `POST /api/v1/enrichment/create`
   (`create_enrichment_api_v1_enrichment_create_post`) — returns an enrichment id.
2. **Run it asynchronously.** `POST /api/v1/enrichment/enrich_async/{enrichment_id}`
   (`enrich_row_async_api_v1_enrichment_enrich_async__enrichment_id__post`) —
   returns a `workflow_id`.
3. **Poll status.** `GET /api/v1/enrichment/workflow/status/{workflow_id}`
   (`get_enrichment_status_api_v1_enrichment_workflow_status__workflow_id__get`)
   until the workflow reports completion.
4. **Add to a layer.** `POST /api/v1/enrichment/add_to_layer`
   (`add_enrichment_to_layer_api_v1_enrichment_add_to_layer_post`) to merge the
   enriched values into the project's layer.

## Rules
- Enrichment consumes workspace credits; check `GET /api/v1/enrichment/credits`
  before large runs and honor `GET /api/v1/enrichment/limits`.
- Errors use the FastAPI `{"detail": ...}` envelope (422 = validation). See
  `errors/maia-analytics-problem-types.yml`.
- Dial write operations (contacts, campaigns) support an `Idempotency-Key`
  header for safe retries; enrichment jobs are tracked by `workflow_id` instead.
  See `conventions/maia-analytics-conventions.yml`.
