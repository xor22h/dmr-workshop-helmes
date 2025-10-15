# Orchestrating with Docker Compose

## Why Use Docker Compose for AI?

- **Declarative configuration**: Define models and services in one file
- **Reproducibility**: Same configuration works across dev, staging, and production
- **Service integration**: Connect your app to AI models easily
- **Portability**: Run locally or in the cloud with the same config

## Basic Compose Setup

**Simple application with AI model:**

```yaml save-as=compose.yaml
services:
  myapp:
    image: ghcr.io/open-webui/open-webui:dev-slim
    ports:
      - "8080:8080"
    environment:
      - OPENAI_API_KEY=demo
      - WEBUI_AUTH=False
    models:
      llm:
        endpoint_var: OPENAI_API_BASE_URL

models:
  llm:
    model: ai/gemma3
    context_size: 10000
```

This configuration:
- Defines a `myapp` service that uses a model
- Declares the `llm` model using `ai/gemma3` from Docker Hub
- Docker automatically injects environment variables into the service:
  - `LLM_URL` - API endpoint for the model
  - `LLM_MODEL` - Model identifier

### Model Configuration Options

**With custom settings:**

```yaml
models:
  llm:
    model: ai/smollm2
    context_size: 4096
    runtime_flags:
      - "--no-prefill-assistant"
      - "--threads"
      - "8"
```

**Configuration options:**

- `model`: OCI artifact identifier (required)
- `context_size`: Maximum token context size
- `runtime_flags`: Command-line flags for llama.cpp inference engine

### Service Model Binding

**Short syntax (automatic env vars):**

```yaml
services:
  app:
    image: my-app
    models:
      - llm
      - embedding-model

models:
  llm:
    model: ai/smollm2
  embedding-model:
    model: ai/all-minilm
```

Auto-generated environment variables:
- `LLM_URL`, `LLM_MODEL`
- `EMBEDDING_MODEL_URL`, `EMBEDDING_MODEL_MODEL`

**Long syntax (custom env vars):**

```yaml
services:
  app:
    image: my-app
    models:
      llm:
        endpoint_var: AI_MODEL_URL
        model_var: AI_MODEL_NAME

models:
  llm:
    model: ai/smollm2
```

Your app receives: `AI_MODEL_URL` and `AI_MODEL_NAME`


### Complete Application Example

```yaml
services:
  # Frontend service
  web:
    image: my-web-app
    ports:
      - "3000:3000"
    depends_on:
      - api
    environment:
      - API_URL=http://api:8080

  # Backend API service
  api:
    image: my-api
    ports:
      - "8080:8080"
    models:
      llm:
        endpoint_var: LLM_URL
        model_var: LLM_MODEL
    environment:
      - NODE_ENV=production

  # Database service
  db:
    image: postgres:15
    environment:
      - POSTGRES_PASSWORD=secret
    volumes:
      - db-data:/var/lib/postgresql/data

models:
  llm:
    model: ai/smollm2
    context_size: 4096

volumes:
  db-data:
```

### Running Your Compose Application

```bash no-run-button
# Start all services and models
docker compose up

# Start in detached mode
docker compose up -d

# View logs
docker compose logs -f

# Stop everything
docker compose down
```
