# Using local models with vibe-code tools?

Yes, it's possible. In this demo, we going to hook it with `qwen` code cli.

Your mileage will vary based on the amount of RAM/VRAM you have available, and the model capabilities. For this demo, I will pick tiny version of IBM Granite.

## Download model

First, we need to find the model of our choice. We need GGUF file. As I've said, for this demo we are going to use [IBM Granite](https://huggingface.co/unsloth/granite-4.0-h-tiny-GGUF)

The download link of raw file: https://huggingface.co/unsloth/granite-4.0-h-tiny-GGUF/resolve/main/granite-4.0-h-tiny-Q8_0.gguf

## Packaging for Docker Model Runner

Once we have `.gguf` file - we just need to use `docker model package`

```bash
curl -LO https://huggingface.co/unsloth/granite-4.0-h-tiny-GGUF/resolve/main/granite-4.0-h-tiny-Q8_0.gguf
# Package with custom context-size -- otherwise it would be loaded with default 4096.
docker model package --context-size 262144 --gguf $(pwd)/granite-4.0-h-tiny-Q8_0.gguf xor22h/granite-256k:Q8_0
```

## Usage

For Qwen Code, we only need to set 3 env variables - API URL, Key & Model name

```bash
export OPENAI_MODEL=xor22h/granite-256k:Q8_0
export OPENAI_API_KEY="anything-will-work-here"
export OPENAI_BASE_URL="http://localhost:12434/engines/v1"
```

```bash
qwen
```
