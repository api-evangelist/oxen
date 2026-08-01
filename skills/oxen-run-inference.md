---
name: Run inference against Oxen models
description: Discover available models on Oxen.ai and run OpenAI-compatible chat or image generation through the Hub AI API.
api: openapi/oxen-hub-api-openapi-original.json
operations:
  - OxenApiWeb.Controllers.ModelsController.index      # GET /api/ai/models
  - OxenApiWeb.Controllers.ModelController.get_model_response  # POST /api/ai/chat/completions
  - OxenApiWeb.Controllers.ModelController.generate_image      # POST /api/ai/images/generate
---

# Run inference against Oxen models

Oxen.ai exposes a unified, OpenAI-compatible inference API across 200+ text, image,
video, and audio models. All requests go to `https://hub.oxen.ai/api/ai` and
authenticate with a Bearer API key.

## Auth
Send `Authorization: Bearer $OXEN_API_KEY` on every request (see
`authentication/oxen-authentication.yml`).

## Steps

1. **List models** — `GET /api/ai/models` (`ModelsController.index`) to find a model
   `id`. The response is OpenAI-compatible; you can also filter with
   `GET /api/ai/models/search`.
2. **Chat completion** — `POST /api/ai/chat/completions`
   (`ModelController.get_model_response`) with `{ "model": "<id>", "messages": [...] }`.
   Because the endpoint is OpenAI-compatible, an OpenAI SDK pointed at
   `base_url=https://hub.oxen.ai/api/ai` works unchanged.
3. **Image generation (optional)** — `POST /api/ai/images/generate`
   (`ModelController.generate_image`) with a prompt. Long-running image/video jobs
   are enqueued; poll `GET /api/ai/queue/{generation_id}` until `status` is
   `completed`.

## Rules
- Errors come back as `application/json` with a `status`/`status_message` envelope
  (see `errors/oxen-problem-types.yml`); `401` means a bad/missing key.
- Requests are rate limited — back off on "Rate limit exceeded".
- No idempotency key is supported; do not blindly retry non-idempotent writes.
