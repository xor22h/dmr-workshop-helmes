# OpenAI-Compatible API Integration

To interact with models - you can use standart OpenAI compatible API's - meaning - you can easily use the same libraries you use to interact with cloud-providers in all programming languages. It's very easy to hook it up with Spring AI, node.js, or any other tooling.

## Understanding the API Endpoints

Docker Model Runner exposes OpenAI-compatible API endpoints, making it a drop-in replacement for OpenAI API calls in your applications.

### Base URLs

You can access OpenAI API's under these locations:

1. **From inside containers**: http://model-runner.docker.internal/
2. **From host (if TCP enabled):** http://localhost:12434/ (Note, you can change the port)

### Available OpenAI Endpoints

- `GET /engines/llama.cpp/v1/models` - List available models ([Host](http://localhost:12434/engines/llama.cpp/v1/models))
- `GET /engines/llama.cpp/v1/models/{namespace}/{name}` - Get specific model info ([Host](http://localhost:12434/engines/llama.cpp/v1/models/ai/gemma3))
- `POST /engines/llama.cpp/v1/chat/completions` - Chat completion (most common)
- `POST /engines/llama.cpp/v1/completions` - Text completion
- `POST /engines/llama.cpp/v1/embeddings` - Generate embeddings

**Note**: You can omit `llama.cpp` from the path (e.g., `/engines/v1/chat/completions`)

### Example: Chat Completions from Container

```bash
curl -s http://model-runner.docker.internal/engines/v1/chat/completions \
  -H "Content-Type: application/json" \
  -d '{
    "model": "ai/smollm2",
    "messages": [
      {
        "role": "system",
        "content": "You are a helpful assistant."
      },
      {
        "role": "user",
        "content": "Explain Docker in 100 words."
      }
    ]
  }' | jq
```

### Example: Chat Completions from Host

```bash
# Make sure TCP support is enabled first
curl -s http://localhost:12434/engines/v1/chat/completions \
  -H "Content-Type: application/json" \
  -d '{
    "model": "ai/smollm2",
    "messages": [
      {"role": "user", "content": "Baseball bat and a ball cost 1.10 eur. The ball is 1 eur more expensive than a baseball bat. How much the ball costs?"}
    ]
  }' | jq
## P.s. Expect hallucinations, as smollm2 is quite small
```

### Integrating with Applications

Since the API is OpenAI-compatible, you can use existing OpenAI SDKs by simply changing the base URL:

**JavaScript/Node.js Example:**

```javascript save-as=examples/node-js/main.js
import OpenAI from 'openai'; // npm install openai

const client = new OpenAI({
    baseURL: 'http://model-runner.docker.internal/engines/v1',
    // baseURL: 'http://localhost:12434/engines/v1',
    apiKey: 'not-needed'
});

const response = await client.chat.completions.create({
    model: 'ai/gemma3',
    messages: [
        { 
            role: 'user', 
            content: 'What are the benefits of containerization?' 
        }
    ]
});


console.log(response.choices[0].message.content);
```