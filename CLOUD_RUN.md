# Cloud Run Deployment and Environment Configuration

This document outlines how environment variables and secrets defined in this application are mapped from a local Docker environment to Google Cloud Run.

## Overview & Security Best Practices

- **Never hardcode secrets** in `Dockerfile`, source code, `.env` files committed to version control, or Docker image layers.
- **Non-secret application configuration** (e.g. database connection URLs, log levels, server ports) is supplied as environment variables.
- **Secrets & sensitive credentials** (e.g. API keys, passwords, private tokens) must be stored in **Google Secret Manager** and mounted or injected at runtime into Cloud Run services.

---

## Variable Classification

| Environment Variable | Type | Purpose | Local Mechanism | Cloud Run Equivalent |
|---|---|---|---|---|
| `DATABASE_URL` | Non-Secret Config | Database Connection String | `.env` file (`--env-file .env`) | `--set-env-vars DATABASE_URL="..."` |
| `LOG_LEVEL` | Non-Secret Config | Logging Verbosity (`info`, `debug`, `error`) | `.env` file (`--env-file .env`) | `--set-env-vars LOG_LEVEL="..."` |
| `PORT` | Non-Secret Config | HTTP Server Listening Port | `.env` file or default (`3000`) | `--set-env-vars PORT="3000"` (or Cloud Run default `8080`) |
| `API_KEY` | Secret | Sensitive API Access Key | Runtime flag (`-e API_KEY="..."`) | `--set-secrets API_KEY=api-key-secret:latest` (Secret Manager) |

---

## Local Docker Setup vs. Cloud Run Mapping

### 1. Local Docker Execution
Non-secret configurations are read from the local, git-ignored `.env` file, while secrets are injected directly at runtime via CLI parameters:

```bash
docker run \
  --name app-fixed \
  --env-file .env \
  -e API_KEY="$API_KEY" \
  -p 3000:3000 \
  envlab
```

### 2. Cloud Run Deployment Command
When deploying to Cloud Run, non-secret environment variables are set using `--set-env-vars`, and sensitive credentials stored in Secret Manager are attached using `--set-secrets`:

```bash
# Step 1: Create the secret in Google Secret Manager (one-time setup)
echo -n "$API_KEY" | gcloud secrets create api-key-secret --data-file=-

# Step 2: Deploy to Cloud Run using --set-env-vars and --set-secrets
gcloud run deploy env-config-lab \
  --image gcr.io/YOUR_PROJECT_ID/envlab:latest \
  --platform managed \
  --region us-central1 \
  --allow-unauthenticated \
  --set-env-vars DATABASE_URL="mongodb://production-db:27017/mydb",LOG_LEVEL="info",PORT="8080" \
  --set-secrets API_KEY=api-key-secret:latest
```

---

## Summary of Cloud Run Options

- **`--set-env-vars`**: Sets standard non-sensitive runtime environment variables for the service container.
- **`--set-secrets`**: Securely binds secrets from Secret Manager into environment variables without exposing sensitive values in deployment manifests, build scripts, or container images.
