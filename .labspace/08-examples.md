# Real-World Integration Examples

## Hello GenAI

This repository provides an examples in GoLang, Rust, Node.JS & Python showcasing how you can easily integrate apps using open-source libraries with models, managed by Docker Model Runner.

```bash
# Clone the sample repository
git clone https://github.com/xor22h/hello-genai.git
cd hello-genai

# up!
docker compose up -d
```

## Node.js/TypeScript Integration

**Express API with AI:**

```typescript
import express from 'express';
import OpenAI from 'openai';

const app = express();
app.use(express.json());

const openai = new OpenAI({
  baseURL: process.env.LLM_URL || 'http://localhost:12434/engines/v1',
  apiKey: 'not-needed',
});

app.post('/api/chat', async (req, res) => {
  try {
    const { message } = req.body;
    
    const completion = await openai.chat.completions.create({
      model: process.env.LLM_MODEL || 'ai/smollm2',
      messages: [{ role: 'user', content: message }],
    });
    
    res.json({
      response: completion.choices[0].message.content,
    });
  } catch (error) {
    res.status(500).json({ error: 'AI request failed' });
  }
});

const PORT = process.env.PORT || 3000;
app.listen(PORT, () => {
  console.log(`Server running on port ${PORT}`);
});
```
