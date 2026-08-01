---
name: Fine-tune and deploy a custom model
description: Create a fine-tune job on an Oxen repository, run it, poll training status, and deploy a checkpoint for inference.
api: openapi/oxen-hub-api-openapi-original.json
operations:
  - OxenApiWeb.Controllers.FineTuneController.create   # POST /api/repos/{namespace}/{repo_name}/fine_tunes
  - OxenApiWeb.Controllers.FineTuneController.run       # POST /api/repos/{namespace}/{repo_name}/fine_tunes/{id}/actions/run
  - OxenApiWeb.Controllers.FineTuneController.get_train_status  # GET .../fine_tunes/{id}/train_status
  - OxenApiWeb.Controllers.FineTuneController.create_checkpoint_deployment  # POST .../fine_tunes/{id}/checkpoints/{step}/deploy
---

# Fine-tune and deploy a custom model

Fine-tune a model on data versioned in an Oxen repository, then deploy a checkpoint
so it can serve inference through the same Hub AI API. Base URL
`https://hub.oxen.ai`, Bearer API key auth.

## Steps

1. **Create the fine-tune job** — `POST /api/repos/{namespace}/{repo_name}/fine_tunes`
   (`FineTuneController.create`) referencing the base model and the training
   dataset/branch in the repo. The job starts in `created` state.
2. **(Optional) Tokenize** — `POST .../fine_tunes/{id}/tokenize`
   (`FineTuneController.tokenize`) if the job is in `created` state and needs
   tokenization first.
3. **Run it** — `POST .../fine_tunes/{id}/actions/run` (`FineTuneController.run`).
   The job must be in a runnable state (`created` or `tokenizing`).
4. **Poll training status** — `GET .../fine_tunes/{id}/train_status`
   (`FineTuneController.get_train_status`); list saved checkpoints with
   `GET .../fine_tunes/{id}/checkpoints`.
5. **Deploy a checkpoint** — `POST .../fine_tunes/{id}/checkpoints/{step}/deploy`
   (`FineTuneController.create_checkpoint_deployment`). Pass `retry=true` to tear
   down an existing deployment first. Once deployed the model `id` appears in
   `GET /api/ai/models` and serves via `/api/ai/chat/completions`.

## Rules
- Auth: `Authorization: Bearer $OXEN_API_KEY` (`authentication/oxen-authentication.yml`).
- Long operations are asynchronous — poll status rather than blocking.
- Stop a run with `POST .../fine_tunes/{id}/actions/stop` if needed.
- Errors: `application/json` `status`/`status_message` envelope
  (`errors/oxen-problem-types.yml`).
