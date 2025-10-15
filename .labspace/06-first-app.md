
# Building Your First AI-Powered App

## Complete Example: AI Chat Application

Let's build a simple but complete chat application using Docker Model Runner.

**Project structure:**
```
my-ai-app/
├── compose.yml
├── frontend/
│   ├── Dockerfile
│   ├── index.html
│   └── app.js
└── backend/
    ├── Dockerfile
    ├── main.go
    └── go.mod
    └── go.sum
```

**compose.yml:**
```yaml save-as=examples/my-ai-app/compose.yaml
services:
  # Frontend web server
  frontend:
    build: ./frontend
    ports:
      - "8080:80"
    depends_on:
      - backend

  # Backend API
  backend:
    build: ./backend
    ports:
      - "8081:8081"
    models:
      chat:
        endpoint_var: AI_URL
        model_var: AI_MODEL

models:
  chat:
    model: ai/smollm2:latest
    context_size: 2048
```

**backend/main.go:**
```go save-as=examples/my-ai-app/backend/main.go
package main

import (
	"context"
	"encoding/json"
	"log"
	"net/http"
	"os"

	"github.com/openai/openai-go"
	"github.com/openai/openai-go/option"
)

type ChatRequest struct {
	Messages []openai.ChatCompletionMessageParamUnion `json:"messages"`
}

type ChatResponse struct {
	Message string `json:"message"`
}

type ErrorResponse struct {
	Error string `json:"error"`
}

func corsMiddleware(next http.Handler) http.Handler {
	return http.HandlerFunc(func(w http.ResponseWriter, r *http.Request) {
		w.Header().Set("Access-Control-Allow-Origin", "*")
		w.Header().Set("Access-Control-Allow-Methods", "POST, OPTIONS")
		w.Header().Set("Access-Control-Allow-Headers", "Content-Type")

		if r.Method == "OPTIONS" {
			w.WriteHeader(http.StatusOK)
			return
		}

		next.ServeHTTP(w, r)
	})
}

func chatHandler(client *openai.Client) http.HandlerFunc {
	return func(w http.ResponseWriter, r *http.Request) {
		if r.Method != http.MethodPost {
			http.Error(w, "Method not allowed", http.StatusMethodNotAllowed)
			return
		}

		var req ChatRequest
		if err := json.NewDecoder(r.Body).Decode(&req); err != nil {
			w.Header().Set("Content-Type", "application/json")
			w.WriteHeader(http.StatusBadRequest)
			json.NewEncoder(w).Encode(ErrorResponse{Error: err.Error()})
			return
		}

		resp, err := client.Chat.Completions.New(context.Background(), openai.ChatCompletionNewParams{
			Model:    os.Getenv("AI_MODEL"),
			Messages: req.Messages,
		})

		if err != nil {
			w.Header().Set("Content-Type", "application/json")
			w.WriteHeader(http.StatusInternalServerError)
			json.NewEncoder(w).Encode(ErrorResponse{Error: err.Error()})
			return
		}

		w.Header().Set("Content-Type", "application/json")
		json.NewEncoder(w).Encode(ChatResponse{
			Message: resp.Choices[0].Message.Content,
		})
	}
}

func main() {
	client := openai.NewClient(
		option.WithBaseURL(os.Getenv("AI_URL")),
		option.WithAPIKey("not-needed"),
	)

	mux := http.NewServeMux()
	mux.HandleFunc("/api/chat", chatHandler(&client))

	handler := corsMiddleware(mux)

	log.Println("Server starting on :8081")
	if err := http.ListenAndServe(":8081", handler); err != nil {
		log.Fatal(err)
	}
}
```

**backend/go.mod:**
```go save-as=examples/my-ai-app/backend/go.mod
module github.com/xor22h/dmr-workshop-helmes

go 1.25.0

require github.com/openai/openai-go v1.12.0

require (
	github.com/tidwall/gjson v1.14.4 // indirect
	github.com/tidwall/match v1.1.1 // indirect
	github.com/tidwall/pretty v1.2.1 // indirect
	github.com/tidwall/sjson v1.2.5 // indirect
)
```

**backend/go.sum:**
```go save-as=examples/my-ai-app/backend/go.sum
github.com/openai/openai-go v1.12.0 h1:NBQCnXzqOTv5wsgNC36PrFEiskGfO5wccfCWDo9S1U0=
github.com/openai/openai-go v1.12.0/go.mod h1:g461MYGXEXBVdV5SaR/5tNzNbSfwTBBefwc+LlDCK0Y=
github.com/tidwall/gjson v1.14.2/go.mod h1:/wbyibRr2FHMks5tjHJ5F8dMZh3AcwJEMf5vlfC0lxk=
github.com/tidwall/gjson v1.14.4 h1:uo0p8EbA09J7RQaflQ1aBRffTR7xedD2bcIVSYxLnkM=
github.com/tidwall/gjson v1.14.4/go.mod h1:/wbyibRr2FHMks5tjHJ5F8dMZh3AcwJEMf5vlfC0lxk=
github.com/tidwall/match v1.1.1 h1:+Ho715JplO36QYgwN9PGYNhgZvoUSc9X2c80KVTi+GA=
github.com/tidwall/match v1.1.1/go.mod h1:eRSPERbgtNPcGhD8UCthc6PmLEQXEWd3PRB5JTxsfmM=
github.com/tidwall/pretty v1.2.0/go.mod h1:ITEVvHYasfjBbM0u2Pg8T2nJnzm8xPwvNhhsoaGGjNU=
github.com/tidwall/pretty v1.2.1 h1:qjsOFOWWQl+N3RsoF5/ssm1pHmJJwhjlSbZ51I6wMl4=
github.com/tidwall/pretty v1.2.1/go.mod h1:ITEVvHYasfjBbM0u2Pg8T2nJnzm8xPwvNhhsoaGGjNU=
github.com/tidwall/sjson v1.2.5 h1:kLy8mja+1c9jlljvWTlSazM7cKDRfJuR/bOJhcY5NcY=
github.com/tidwall/sjson v1.2.5/go.mod h1:Fvgq9kS/6ociJEDnK0Fk1cpYF4FIW6ZF7LAe+6jwd28=
```

**backend/Dockerfile:**
```dockerfile save-as=examples/my-ai-app/backend/Dockerfile
# syntax=docker/dockerfile:1

ARG GO_VERSION=1.25.0
FROM --platform=$BUILDPLATFORM golang:${GO_VERSION} AS build
WORKDIR /src

RUN --mount=type=cache,target=/go/pkg/mod/ \
    --mount=type=bind,source=go.sum,target=go.sum \
    --mount=type=bind,source=go.mod,target=go.mod \
    go mod download -x

ARG TARGETARCH

# Build the application.
RUN --mount=type=cache,target=/go/pkg/mod/ \
    --mount=type=bind,target=. \
    CGO_ENABLED=0 GOARCH=$TARGETARCH go build -o /bin/server .

FROM alpine:latest AS final

RUN --mount=type=cache,target=/var/cache/apk \
    apk --update add \
        ca-certificates \
        tzdata \
        && \
        update-ca-certificates

ARG UID=10001
RUN adduser \
    --disabled-password \
    --gecos "" \
    --home "/nonexistent" \
    --shell "/sbin/nologin" \
    --no-create-home \
    --uid "${UID}" \
    appuser
USER appuser

COPY --from=build /bin/server /bin/
EXPOSE 8081
ENTRYPOINT [ "/bin/server" ]
```

**frontend/index.html:**
```html save-as=examples/my-ai-app/frontend/index.html
<!DOCTYPE html>
<html>
<head>
    <title>AI Chat</title>
    <style>
        body { font-family: Arial; max-width: 800px; margin: 50px auto; }
        #messages { height: 400px; overflow-y: scroll; border: 1px solid #ccc; padding: 10px; }
        #input { width: 80%; padding: 10px; }
        button { padding: 10px 20px; }
    </style>
</head>
<body>
    <h1>AI Chat with Docker Model Runner</h1>
    <div id="messages"></div>
    <input id="input" type="text" placeholder="Type your message...">
    <button onclick="sendMessage()">Send</button>
    <script src="app.js"></script>
</body>
</html>
```

**frontend/app.js:**
```javascript save-as=examples/my-ai-app/frontend/app.js
let messages = [];

async function sendMessage() {
    const input = document.getElementById('input');
    const userMessage = input.value;
    if (!userMessage) return;

    messages.push({ role: 'user', content: userMessage });
    displayMessage('You', userMessage);
    input.value = '';

    try {
        const response = await fetch('http://localhost:8081/api/chat', {
            method: 'POST',
            headers: { 'Content-Type': 'application/json' },
            body: JSON.stringify({ messages })
        });

        const data = await response.json();
        messages.push({ role: 'assistant', content: data.message });
        displayMessage('AI', data.message);
    } catch (error) {
        displayMessage('Error', 'Failed to get response');
    }
}

function displayMessage(sender, text) {
    const div = document.getElementById('messages');
    div.innerHTML += `<p><strong>${sender}:</strong> ${text}</p>`;
    div.scrollTop = div.scrollHeight;
}
```

**frontend/Dockerfile:**

```dockerfile save-as=examples/my-ai-app/frontend/Dockerfile
FROM nginx:alpine
COPY --chmod=644 index.html /usr/share/nginx/html/
COPY --chmod=644 app.js /usr/share/nginx/html/
```

**Run the application:**
```bash
# Start everything
docker compose up
```

::tabLink[localhost:8080]{href="http://localhost:8080" title="My First App"}
