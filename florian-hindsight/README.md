# Hindsight on Umbrel

Install **Hindsight** from this community app store. In the Umbrel app settings,
set `HINDSIGHT_API_LLM_PROVIDER`, `HINDSIGHT_API_LLM_MODEL` and, for
hosted providers, `HINDSIGHT_API_LLM_API_KEY`. Restart the app after changes.

The Control Plane opens from the app tile on port 9999. Sign in with the
password shown on the Umbrel app page. The API is at
`http://umbrel.local:8888`, and MCP at
`http://umbrel.local:8888/mcp/<bank_id>/`. API clients must provide the
Umbrel app seed as a Bearer token; retrieve the `APP_SEED` value from the app's container environment on the
Umbrel host. Keep it private.
Do not publish port 8888 directly to the internet; use HTTPS and access controls
when connecting remotely.

For local Ollama, set provider `ollama`, a model already pulled in Ollama,
and base URL `http://host.docker.internal:11434/v1`. Ollama must listen on an
address reachable by Docker containers. No LLM API key is needed.

The embedded PostgreSQL data is persisted under
`~/umbrel/app-data/florian-hindsight/data/`. Back up this directory with the
app stopped. The container runs as UID 1000, so its directory must be writable
by that user. The image runs local embeddings and reranking on CPU and needs
around 2 GB RAM plus 1 GB shared memory. The first start may download models.

Upstream documentation: https://hindsight.vectorize.io/developer/installation
and https://hindsight.vectorize.io/developer/configuration
