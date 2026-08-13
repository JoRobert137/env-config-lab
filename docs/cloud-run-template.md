# Cloud Run Mapping

Configuration Variables

- `DATABASE_URL`: Set via `--set-env-vars DATABASE_URL="..."`
- `LOG_LEVEL`: Set via `--set-env-vars LOG_LEVEL="info"`
- `PORT`: Set via `--set-env-vars PORT="3000"`

Secrets

- `API_KEY`: Stored in Google Secret Manager and referenced via `--set-secrets API_KEY=api-key-secret:latest`
