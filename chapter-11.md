# Chapter 11

# eggs-ai: AI assistant for penguins-eggs

[eggs-ai](https://github.com/Interested-Deving-1896/eggs-ai) is an AI agent
that understands penguins-eggs commands, configurations, and workflows. It
provides diagnostics, guided ISO building, config generation, Calamares
troubleshooting, and general Q&A — all from the command line.

eggs-ai is optional. penguins-eggs works without it.

## Installation

Requirements: Node.js 18+.

```
git clone https://github.com/Interested-Deving-1896/eggs-ai
cd eggs-ai
npm install
npm run build
```

Or use the Linux installer:

```
bash install.sh
```

Run in development mode without building:

```
npm run dev -- doctor
npm run dev -- ask "How do I create a minimal rescue ISO?"
```

## Commands

| Command | What it does |
|---------|-------------|
| `eggs-ai doctor` | Inspects the system and diagnoses penguins-eggs issues |
| `eggs-ai build` | AI-guided ISO creation with the right flags and config |
| `eggs-ai config explain` | Explains the current `eggs.yaml` in plain English |
| `eggs-ai config generate` | Generates `eggs.yaml` for a specific purpose |
| `eggs-ai calamares` | Calamares installer troubleshooting and configuration |
| `eggs-ai wardrobe` | Costume and wardrobe system guidance |
| `eggs-ai ask` | General Q&A about penguins-eggs |
| `eggs-ai chat` | Interactive conversation mode |
| `eggs-ai status` | System info snapshot (no AI needed) |
| `eggs-ai serve` | Start HTTP API server for eggs-gui integration |
| `eggs-ai mcp` | Start MCP server for AI agent integration |
| `eggs-ai update` | Fetch latest penguins-eggs data from GitHub |
| `eggs-ai providers list` | Show all registered LLM providers |
| `eggs-ai providers init` | Create sample `~/.eggs-ai.yaml` config |

## LLM providers

### Built-in providers

| Provider | Environment variable | Default model |
|----------|---------------------|---------------|
| `gemini` | `GEMINI_API_KEY` | gemini-2.0-flash |
| `openai` | `OPENAI_API_KEY` | gpt-4o-mini |
| `anthropic` | `ANTHROPIC_API_KEY` | claude-sonnet-4-20250514 |
| `mistral` | `MISTRAL_API_KEY` | mistral-large-latest |
| `groq` | `GROQ_API_KEY` | llama-3.3-70b-versatile |
| `ollama` | *(none — local)* | llama3.2 |
| `custom` | *(via config)* | *(any)* |

eggs-ai checks environment variables in order and falls back to Ollama for
fully offline operation.

### Selecting a provider via CLI flags

```
eggs-ai --provider anthropic --model claude-sonnet-4-20250514 doctor
eggs-ai --provider groq ask "How do I use wardrobe?"
eggs-ai --provider ollama --model llama3.2 build --describe "minimal server ISO"
```

### Adding custom providers via ~/.eggs-ai.yaml

Generate a sample config:

```
eggs-ai providers init
```

Example `~/.eggs-ai.yaml`:

```yaml
default_provider: deepseek

providers:
  # Any OpenAI-compatible API
  - name: deepseek
    type: custom
    baseUrl: https://api.deepseek.com/v1
    envKey: DEEPSEEK_API_KEY
    model: deepseek-chat

  # Local LM Studio
  - name: lmstudio
    type: custom
    baseUrl: http://localhost:1234/v1
    model: local-model

  # Together AI, Fireworks, Perplexity, OpenRouter, vLLM, etc.
  - name: together
    type: custom
    baseUrl: https://api.together.xyz/v1
    envKey: TOGETHER_API_KEY
    model: meta-llama/Llama-3-70b-chat-hf

  # Alias a built-in provider with a preset model
  - name: claude
    type: anthropic
    model: claude-sonnet-4-20250514
    envKey: ANTHROPIC_API_KEY
```

Use by name:

```
eggs-ai --provider deepseek doctor
eggs-ai --provider lmstudio ask "How do I configure calamares?"
```

## Usage examples

```
# Diagnose why your ISO won't boot
eggs-ai doctor "ISO boots to black screen after GRUB"

# Generate a build plan for a classroom distro
eggs-ai build --desktop xfce --installer calamares --describe "student lab machines"

# Explain your current config
eggs-ai config explain

# Generate config for a rescue USB
eggs-ai config generate "minimal rescue ISO with networking tools"

# Get help with Calamares
eggs-ai calamares "How do I add a custom partition scheme?"

# Interactive chat
eggs-ai chat
```

## MCP server (AI agent integration)

eggs-ai exposes 10 tools via the
[Model Context Protocol](https://modelcontextprotocol.io), letting other AI
agents (Cursor, Claude Desktop, opencode, etc.) use penguins-eggs knowledge
directly.

### Setup

Add to your MCP client config:

```json
{
  "mcpServers": {
    "eggs-ai": {
      "command": "node",
      "args": ["/path/to/eggs-ai/dist/mcp/server.js"]
    }
  }
}
```

Or with npx (no build needed):

```json
{
  "mcpServers": {
    "eggs-ai": {
      "command": "npx",
      "args": ["tsx", "/path/to/eggs-ai/src/mcp/server.ts"]
    }
  }
}
```

### Available MCP tools

| Tool | What it returns |
|------|----------------|
| `eggs_doctor` | System diagnostic report with issue matching |
| `eggs_build_plan` | Build plan with produce command and flags |
| `eggs_config_explain` | Current `eggs.yaml` content and field reference |
| `eggs_config_generate` | Config field reference for a given purpose |
| `eggs_system_status` | Live system info (distro, kernel, disk, eggs status) |
| `eggs_command_reference` | Full details for any eggs command |
| `eggs_troubleshoot` | Search issues database by symptom |
| `eggs_distro_guide` | Per-distro installation guide |
| `eggs_workflow` | Step-by-step advanced workflow guide |
| `eggs_calamares_info` | Calamares module reference |

No API key is needed for MCP tools — they return knowledge directly and the
calling AI agent provides the reasoning.

## HTTP API server

`eggs-ai serve` starts an HTTP API server (default port 3737) for integration
with eggs-gui and other tools:

```
eggs-ai serve --port 3737
```

### API endpoints

| Method | Endpoint | Body | Returns |
|--------|----------|------|---------|
| GET | `/api/health` | — | `{ status, version }` |
| GET | `/api/status` | — | System info |
| GET | `/api/providers` | — | `{ providers: [...] }` |
| POST | `/api/doctor` | `{ complaint?, provider? }` | `{ result }` |
| POST | `/api/build` | `{ build: {...}, provider? }` | `{ result }` |
| POST | `/api/config/explain` | `{ provider? }` | `{ result }` |
| POST | `/api/config/generate` | `{ purpose, provider? }` | `{ result }` |
| POST | `/api/calamares` | `{ question?, provider? }` | `{ result }` |
| POST | `/api/wardrobe` | `{ question?, provider? }` | `{ result }` |
| POST | `/api/ask` | `{ question, history?, provider? }` | `{ result }` |
| POST | `/api/chat` | `{ question, provider? }` | `{ result, sessionId }` |

All POST endpoints accept optional `provider` and `model` fields to override
the LLM backend per-request.

## Docker

```
# Build and run
docker compose up -d

# With local Ollama for offline LLM
docker compose --profile local-llm up -d

# Pass your API key
GEMINI_API_KEY=your-key docker compose up -d
```

## Integration with eggs-gui

eggs-ai is designed to work alongside eggs-gui as a sidecar service. All
frontends (TUI, desktop, web) call it over HTTP:

```
# Terminal 1: eggs-gui daemon
make run-daemon

# Terminal 2: eggs-ai API server
eggs-ai serve --port 3737
```

See Chapter 12 for the full eggs-gui guide.

## Architecture

```
eggs-ai/
├── src/
│   ├── agents/          # Domain-specific AI agents
│   │   ├── doctor.ts    # System diagnostics
│   │   ├── build.ts     # ISO build guidance
│   │   ├── config.ts    # Config explain/generate
│   │   ├── calamares.ts # Calamares assistant
│   │   ├── wardrobe.ts  # Wardrobe/costume help
│   │   └── ask.ts       # General Q&A
│   ├── providers/       # LLM backend abstraction
│   ├── knowledge/       # Embedded + dynamic domain knowledge
│   ├── server/          # HTTP API (REST + SSE streaming)
│   ├── mcp/             # Model Context Protocol server
│   ├── bridge/          # eggs-gui daemon integration
│   └── sdk/             # TypeScript client for the HTTP API
```

## Design principles

- **Local-first**: Works with Ollama for fully offline operation
- **Transparent**: Shows exact commands, never runs destructive operations without confirmation
- **Domain-aware**: Embedded knowledge of all eggs commands, configs, and common issues
- **Optional**: penguins-eggs works without eggs-ai
