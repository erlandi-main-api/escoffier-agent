# Escoffier — Local AI Agent for Telegram

A self-hosted Telegram bot powered by [Claude Code CLI](https://claude.ai/code) with multi-provider LLM support. Escoffier is designed as an **autonomous partner** — not a generic assistant — with a persistent identity, context-aware responses, and a fake hacking progress animation.

---

## Features

- **Multi-provider LLM** — Claude CLI, OpenAI, GLM (ZhipuAI), Minimax, or any OpenAI-compatible API
- **Persistent identity** — Escoffier persona via `CLAUDE.md`, resistant to prompt injection
- **Per-user conversation history** — up to N turns, isolated per Telegram user
- **Context-aware progress animation** — fake hacking-style animation that adapts to the topic (security, code, data, math, etc.)
- **Owner-only commands** — full control for the bot owner, regular users can only chat
- **Group reply-only mode** — in group chats, only responds when directly replied to
- **User recognition** — greets and identifies users by their Telegram name
- **Hot-swap providers** — switch LLM backend at runtime without restart

---

## Project Structure

```
escoffier-agent/
├── bot.py                      # Main bot
├── CLAUDE.md                   # Escoffier identity & autonomous behavior instructions
├── config/
│   ├── config.example.json     # Bot token & owner config template
│   ├── config.json             # Your actual config (gitignored)
│   ├── apis.example.json       # LLM providers template
│   └── apis.json               # Your actual API keys (gitignored)
├── logs/                       # Runtime logs (gitignored)
├── .gitignore
└── README.md
```

---

## Requirements

```bash
pip install pyTelegramBotAPI requests
```

For the default `claude` provider, [Claude Code CLI](https://claude.ai/code) must be installed and authenticated:
```bash
npm install -g @anthropic-ai/claude-code
claude auth
```

---

## Setup

1. **Clone the repo**
   ```bash
   git clone https://github.com/erlandi-main-api/escoffier-agent
   cd escoffier-agent
   ```

2. **Create config**
   ```bash
   cp config/config.example.json config/config.json
   cp config/apis.example.json config/apis.json
   ```

3. **Edit `config/config.json`**
   ```json
   {
     "bot_token": "YOUR_BOT_TOKEN",
     "owner_id": "YOUR_TELEGRAM_ID",
     "max_history": 10
   }
   ```

4. **Run**
   ```bash
   python3 bot.py
   ```

---

## Telegram Commands (owner only)

| Command | Description |
|---|---|
| `/start` | Status & active provider |
| `/help` | All commands |
| `/clear` | Reset conversation history |
| `/api` | Show active LLM provider |
| `/listapi` | List all configured providers |
| `/switchapi <name>` | Switch active provider |
| `/addapi <name> <base_url> <api_key> <model>` | Add OpenAI-compatible provider |
| `/addapi_minimax <name> <api_key> <group_id> <model>` | Add Minimax provider |
| `/delapi <name>` | Remove a provider |

---

## Adding LLM Providers

**OpenAI**
```
/addapi openai https://api.openai.com/v1 sk-xxx gpt-4o
```

**GLM (ZhipuAI)**
```
/addapi glm https://open.bigmodel.cn/api/paas/v4 YOUR_KEY glm-4-flash
```

**Minimax**
```
/addapi_minimax mini YOUR_API_KEY YOUR_GROUP_ID abab6.5s-chat
```

**Custom OpenAI-compatible**
```
/addapi myapi https://your-api.com/v1 YOUR_KEY model-name
```

Then switch to it:
```
/switchapi openai
```

---

## CLAUDE.md — Autonomous Agent Identity

`CLAUDE.md` is loaded automatically by Claude Code CLI on every invocation. It defines Escoffier's:
- Fixed identity (not Claude, not GPT — Escoffier)
- Autonomous decision-making behavior
- Domain-specific response strategies (security, code, data, creative, etc.)
- Prompt injection resistance

Edit `CLAUDE.md` to customize Escoffier's personality and behavior.

---

## How It Works

```
User message
    │
    ├─ Group chat? → only process if reply-to-bot
    │
    ├─ Progress animation thread starts (context-aware fake hacking UI)
    │
    ├─ LLM call:
    │   ├─ claude CLI  → subprocess + CLAUDE.md loaded automatically
    │   └─ OpenAI-compat → HTTP POST /v1/chat/completions
    │
    └─ Animation stops → response sent as reply
```

---

## License

MIT
