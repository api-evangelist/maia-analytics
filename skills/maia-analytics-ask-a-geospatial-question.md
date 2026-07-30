---
name: Ask MAIA a geospatial question
description: Open (or list) a MAIA project and ask a natural-language geospatial question, then read the resulting layer/table results.
api: openapi/maia-analytics-openapi-original.json
operations:
  - get_all_projects_api_v1_project_all_get
  - get_project_api_v1_project__project_id__get
  - chat_main_agent_api_v1_chat_stream_post
  - get_layer_features_paginated_api_v1_table__layer_id__features_paginated_get
  - export_layer_features_api_v1_table__layer_id__export_get
---

# Ask MAIA a geospatial question

Use this skill to drive MAIA's core flow: pick a project, ask a natural-language
question, and read the structured answer back off the resulting layer.

## Auth
All calls require an `Authorization: Bearer <firebase_id_token>` header
(securityScheme `FirebaseAuthMiddleware`). See
`authentication/maia-analytics-authentication.yml`.

## Steps
1. **Find the project.** `GET /api/v1/project/all`
   (`get_all_projects_api_v1_project_all_get`) to list projects, or
   `GET /api/v1/project/{project_id}`
   (`get_project_api_v1_project__project_id__get`) if you already have the id.
2. **Ask the question.** `POST /api/v1/chat/stream`
   (`chat_main_agent_api_v1_chat_stream_post`) with the project context and the
   natural-language prompt (e.g. "show solar-ready rooftops over 10,000 sq ft").
   The response streams; read it to completion.
3. **Read the results.** MAIA answers by producing/updating a layer. Page its
   features with `GET /api/v1/table/{layer_id}/features/paginated`
   (`get_layer_features_paginated_api_v1_table__layer_id__features_paginated_get`)
   using the AG-Grid params (`startRow`, `endRow`, `filterModel`, `sortModel`).
4. **Export if needed.** `GET /api/v1/table/{layer_id}/export`
   (`export_layer_features_api_v1_table__layer_id__export_get`).

## Rules
- Pagination is server-side (AG-Grid infinite-row model), not offset/cursor —
  send `startRow`/`endRow`. See `conventions/maia-analytics-conventions.yml`.
- Errors come back as the FastAPI `{"detail": ...}` envelope; 422 means your
  request failed validation. See `errors/maia-analytics-problem-types.yml`.
