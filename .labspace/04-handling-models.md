# Models

## Where to get models?

- [Docker Hub](https://hub.docker.com/u/ai) AI models: https://hub.docker.com/u/ai
- [Hugging Face](https://huggingface.co/models?apps=docker-model-runner&sort=trending) search by either `Docker Model Runner` to find already OCI Packaged models, or pick any `.gguf` file, download it, and package it yourself

## Why Package Models as OCI Artifacts?

- **Distribution**: Share models using existing container registries (Docker Hub, Harbor, Azure Container Registry, others...)
- **Versioning**: Use tags to version your models like container images
- **Reproducibility**: Ensure everyone uses the exact same model version
- **CI/CD Integration**: Treat models like any other artifact in your pipeline

## Packaging a GGUF Model

[GGUF](https://github.com/ggml-org/ggml/blob/master/docs/gguf.md) is a file format for storing models optimized for inference.

**Basic packaging example:**

```bash
curl -L -o model.gguf \
  https://huggingface.co/TheBloke/Mistral-7B-v0.1-GGUF/resolve/main/mistral-7b-v0.1.Q4_K_M.gguf
```

### Package as OCI artifact

```bash
docker model package \
  --gguf "$(pwd)/model.gguf" \
  xor22h/mistral:7b
```

**Package and push directly:**

```bash
docker model package \
  --gguf "$(pwd)/model.gguf" \
  --push \
  xor22h/mistral:7b
```

**Package with license and custom context size:**

```bash
docker model package \
  --gguf "$(pwd)/model.gguf" \
  --license "$(pwd)/LICENSE.txt" \
  --context-size 8192 \
  --push \
  helmes/mistral:7b
```

## Working with Packaged Models

**Tag existing models:**

```bash
# Pull a model from Docker Hub
docker model pull ai/smollm2

# Tag it under your namespace
docker model tag ai/smollm2 myorg/smollm2:production

# Push to your registry
docker model push myorg/smollm2:production
```

## Sharded Models

For large models split into multiple files:
```bash
# Point to the first shard; all shards must be in the same directory
# Files should be named: model-00001-of-00015.gguf, model-00002-of-00015.gguf, etc.
docker model package \
  --gguf "$(pwd)/model-00001-of-00015.gguf" \
  --push \
  myorg/large-model:70B
```
