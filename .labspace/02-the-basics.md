# Getting Started - Your First Model

## Enable Docker Model Runner

**For Docker Desktop:**

1. Go to Settings → Beta features tab

![Docker Settings](./.labspace/images/settings.png)

2. Enable "Docker Model Runner"
3. (Optional) Enable TCP support and set a port (default: 12434)
4. Configure CORS origins if using with web apps (e.g., `http://localhost:3131`)

**For Docker Engine (Debian):**

```bash
# Install the plugin
sudo apt-get update
sudo apt-get install docker-model-plugin
```
**For Docker Engine (RHEL):**

```bash
# Or for Fedora/RHEL
sudo dnf update
sudo dnf install docker-model-plugin
```

```bash
# Verify installation
docker model version
```

### Pull Your First Model

**Using CLI:**

```bash
# Pull from Docker Hub
docker model pull ai/smollm2

# Or pull directly from HuggingFace
docker model pull hf.co/bartowski/Llama-3.2-1B-Instruct-GGUF
```

**Using Docker Desktop:**

- Navigate to Models tab
- Select Docker Hub tab
- Find and pull your desired model

### Run and Interact with Models

**1. Pull a small model**

```bash
docker model pull ai/smollm2
```

**2. List your models**

```bash
docker model ls
```

**3. Run it interactively**

```bash
docker model run ai/smollm2
```

**4. Try these prompts:**

```plaintext
Explain Docker in one sentence
```

```plaintext
Why number 42 is so special?
```

This opens an interactive chat session. Type `/bye` to exit.

**Key Concept**: Models load on-demand. You don't need to explicitly run them before making API calls - they load automatically when requested and unload when idle to save resources.

