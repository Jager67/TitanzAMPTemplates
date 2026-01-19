# Titanz AI Agent - Setup Guide

## What is Titanz AI Agent?

A lightweight Node.js server that connects your AMP game server panel to local AI models (Ollama, LM Studio, etc.) for automation, log analysis, and intelligent workflows.

## Features

- **AI-Powered Automation**: Connect game servers to local LLMs
- **n8n-Compatible Workflows**: Use simple JSON workflow definitions
- **RESTful API**: Standard HTTP endpoints for easy integration
- **Webhook Support**: Trigger AI workflows via webhooks
- **Rate Limiting**: Built-in protection against abuse
- **Flexible Authentication**: Optional bearer token security
- **Context Management**: Maintains conversation history

## Prerequisites

1. **Node.js** (v16 or higher)
2. **Local AI Model Server** (one of):
   - [Ollama](https://ollama.ai/) - Recommended
   - [LM Studio](https://lmstudio.ai/)
   - [LocalAI](https://localai.io/)
   - Any OpenAI-compatible API

## Quick Start

### 1. Install Local AI (Ollama Example)

```bash
# Download Ollama from https://ollama.ai/
# Then pull a model:
ollama pull llama3.2
```

### 2. Configure in AMP

1. Create a new **Generic Module** instance in AMP
2. Select the **Titanz AI Agent** template
3. Configure settings:
   - **AI Model Endpoint**: `http://localhost:11434/api/generate` (for Ollama)
   - **Default AI Model**: `llama3.2` (or your preferred model)
   - **Server Port**: `5678` (or any available port)
   - **API Token**: (optional) Set a secure token for authentication

### 3. Start the Server

Click **Start** in AMP. You should see:
```
Server listening on port 5678
```

## API Endpoints

### Health Check
```bash
curl http://localhost:5678/health
```

### Chat Completion
```bash
curl -X POST http://localhost:5678/api/chat \
  -H "Content-Type: application/json" \
  -d '{
    "prompt": "Analyze this server error: Connection timeout on port 25565",
    "systemPrompt": "You are a Minecraft server administrator. Provide concise technical solutions."
  }'
```

### Webhook Trigger
```bash
curl -X POST http://localhost:5678/webhook/server-status \
  -H "Content-Type: application/json" \
  -d '{
    "cpu": 85,
    "memory": 72,
    "players": 42,
    "tps": 19.8
  }'
```

## Creating Workflows

Workflows are JSON files placed in the `workflows/` directory. They trigger when the matching webhook is called.

**Example: `workflows/server-status.json`**
```json
{
  "name": "Server Status Alert",
  "description": "Analyzes server metrics",
  "prompt": "Server metrics: {{$json}}. Provide status summary and recommendations."
}
```

Call with:
```bash
curl -X POST http://localhost:5678/webhook/server-status \
  -d '{"cpu": 95, "ram": 88, "tps": 15}'
```

## Integration Examples

### AMP Task Scheduler
Create scheduled tasks in AMP that call the AI agent:

```bash
# Daily log analysis
curl -X POST http://localhost:5678/api/chat \
  -H "Content-Type: application/json" \
  -d @server-logs.json
```

### Discord Bot Integration
```javascript
// Send player reports to AI for triage
const response = await fetch('http://localhost:5678/api/chat', {
  method: 'POST',
  headers: { 'Content-Type': 'application/json' },
  body: JSON.stringify({
    prompt: `Player report: ${reportText}`,
    systemPrompt: 'Categorize and prioritize this player report.'
  })
});
```

### External Monitoring
```bash
# Cron job: Analyze metrics every hour
0 * * * * curl -X POST http://localhost:5678/webhook/hourly-check \
  -d "$(amp-cli get-server-stats)"
```

## Security Best Practices

1. **Enable Authentication**: Set `API Authentication Token` in AMP settings
2. **Limit Origins**: Configure `Allowed Origins` for CORS protection
3. **Rate Limiting**: Adjust `Rate Limit` based on your needs
4. **Network Security**: Bind to localhost only if not exposing externally
5. **Use HTTPS**: Place behind reverse proxy (nginx/Caddy) for production

## Troubleshooting

### "Connection refused" error
- Ensure Ollama/LM Studio is running
- Verify the AI endpoint URL is correct
- Check firewall rules

### Rate limit exceeded
- Increase `Rate Limit (requests/min)` in settings
- Or set to `0` for unlimited (not recommended)

### AI responses are slow
- Reduce `Max Tokens` (try 1024 or 512)
- Use a smaller/faster model (e.g., `llama3.2:1b`)
- Check AI server resources (CPU/GPU usage)

### Webhooks not working
- Ensure `Enable Webhooks` is checked
- Verify workflow JSON files are in `workflows/` directory
- Check workflow file naming matches webhook URL

## Advanced Configuration

### Custom Headers for AI Requests
```json
{
  "User-Agent": "TitanzAI/1.0",
  "X-Custom-Header": "value"
}
```

### Context Window Management
Higher `Context Window Size` = more memory, better continuity
Lower = faster, less memory usage

### Temperature Settings
- `0.1-0.3`: Focused, deterministic responses
- `0.7`: Balanced (default)
- `1.0-2.0`: Creative, varied responses

## Support

For issues or questions:
- **Titanz.tv Support**: [your support channel]
- **AMP Documentation**: https://github.com/CubeCoders/AMP/wiki
- **Ollama Docs**: https://github.com/ollama/ollama

---
**Titanz.tv** - Intelligent Game Server Automation
