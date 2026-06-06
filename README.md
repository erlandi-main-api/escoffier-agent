# Escoffier — Local AI Agent for Telegram

<p align="center">
  <img src="https://img.shields.io/badge/platform-Termux%20%7C%20Linux%20%7C%20macOS%20%7C%20Windows-blue?style=flat-square"/>
  <img src="https://img.shields.io/badge/python-3.8%2B-blue?style=flat-square"/>
  <img src="https://img.shields.io/badge/provider-Claude%20%7C%20OpenAI%20%7C%20GLM%20%7C%20Minimax%20%7C%20Custom-green?style=flat-square"/>
  <img src="https://img.shields.io/badge/license-MIT-lightgrey?style=flat-square"/>
  <img src="https://img.shields.io/badge/author-M%20Erlandi-red?style=flat-square"/>
</p>

A self-hosted Telegram bot powered by [Claude Code CLI](https://claude.ai/code) with multi-provider LLM support. Escoffier is designed as an **autonomous partner** — not a generic assistant — with a persistent identity defined via `CLAUDE.md`, context-aware responses, prompt injection resistance, and a fake hacking-style progress animation.

---

## Table of Contents

- [Features](#features)
- [Project Structure](#project-structure)
- [Requirements](#requirements)
- [Installation](#installation)
  - [Termux (Android)](#termux-android)
  - [Linux (Debian/Ubuntu)](#linux-debianubuntu)
  - [Linux (Arch/Manjaro)](#linux-archmanjaro)
  - [macOS](#macos)
  - [Windows](#windows)
- [Configuration](#configuration)
- [Running the Bot](#running-the-bot)
  - [Run in background (Termux)](#run-in-background-termux)
  - [Run as systemd service (Linux)](#run-as-systemd-service-linux)
  - [Run as background process (macOS)](#run-as-background-process-macos)
  - [Run on Windows](#run-on-windows)
- [Telegram Commands](#telegram-commands)
- [Adding LLM Providers](#adding-llm-providers)
- [CLAUDE.md — Autonomous Identity](#claudemd--autonomous-identity)
- [How It Works](#how-it-works)
- [Security Notes](#security-notes)
- [License](#license)

---

## Features

| Feature | Description |
|---|---|
| Multi-provider LLM | Claude CLI, OpenAI, GLM, Minimax, or any OpenAI-compatible API |
| Autonomous identity | Persistent Escoffier persona via `CLAUDE.md` |
| Prompt injection resistant | Identity cannot be overridden by user input |
| Per-user history | Isolated conversation context per Telegram user |
| Context-aware animation | Fake hacking progress UI adapts to topic |
| Owner-only commands | Full control reserved for bot owner |
| Group reply-only mode | Only responds when directly replied to in groups |
| User recognition | Identifies users by Telegram name |
| Hot-swap providers | Switch LLM backend at runtime, no restart needed |
| Persistent API config | Provider settings saved to `config/apis.json` |

---

## Project Structure

```
escoffier-agent/
├── bot.py                      # Main bot entry point
├── CLAUDE.md                   # Escoffier identity & behavior instructions
├── config/
│   ├── config.example.json     # Bot credentials template
│   ├── config.json             # Your config (gitignored)
│   ├── apis.example.json       # LLM providers template
│   └── apis.json               # Your API keys (gitignored)
├── logs/                       # Runtime logs (gitignored)
├── .gitignore
└── README.md
```

---

## Requirements

- Python 3.8+
- pip packages: `pyTelegramBotAPI`, `requests`
- A Telegram Bot Token — get one from [@BotFather](https://t.me/BotFather)
- Your Telegram User ID — get it from [@userinfobot](https://t.me/userinfobot)
- *(For default Claude provider)* Claude Code CLI installed & authenticated

---

## Installation

### Termux (Android)

> Tested on Android 10+ with Termux from F-Droid (not Play Store).

```bash
# Update packages
pkg update && pkg upgrade -y

# Install dependencies
pkg install python nodejs git -y

# Install pip packages
pip install pyTelegramBotAPI requests

# Install Claude Code CLI (optional, for Claude provider)
npm install -g @anthropic-ai/claude-code
claude auth

# Clone the repo
git clone https://github.com/erlandi-main-api/escoffier-agent
cd escoffier-agent
```

**Keep Termux alive in background:**
```bash
# Install termux-services or use nohup
pkg install termux-services -y

# Or simply run with nohup
nohup python3 bot.py > logs/bot.log 2>&1 &
```

**Prevent Android from killing Termux:**
- Enable "Acquire Wakelock" from Termux notification
- Or install [Termux:Boot](https://f-droid.org/packages/com.termux.boot/) to auto-start on reboot

**Auto-start on boot with Termux:Boot:**
```bash
mkdir -p ~/.termux/boot
cat > ~/.termux/boot/escoffier.sh << 'EOF'
#!/data/data/com.termux/files/usr/bin/bash
cd ~/escoffier-agent
nohup python3 bot.py > logs/bot.log 2>&1 &
EOF
chmod +x ~/.termux/boot/escoffier.sh
```

---

### Linux (Debian/Ubuntu)

```bash
# Install system dependencies
sudo apt update && sudo apt install -y python3 python3-pip git nodejs npm

# Install pip packages
pip3 install pyTelegramBotAPI requests

# Install Claude Code CLI (optional)
npm install -g @anthropic-ai/claude-code
claude auth

# Clone
git clone https://github.com/erlandi-main-api/escoffier-agent
cd escoffier-agent
```

---

### Linux (Arch/Manjaro)

```bash
sudo pacman -Syu python python-pip git nodejs npm --noconfirm

pip install pyTelegramBotAPI requests

npm install -g @anthropic-ai/claude-code
claude auth

git clone https://github.com/erlandi-main-api/escoffier-agent
cd escoffier-agent
```

---

### macOS

```bash
# Install Homebrew if not installed
/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"

brew install python node git

pip3 install pyTelegramBotAPI requests

npm install -g @anthropic-ai/claude-code
claude auth

git clone https://github.com/erlandi-main-api/escoffier-agent
cd escoffier-agent
```

---

### Windows

> Requires Python 3.8+, Node.js, and Git installed.

```powershell
# Install via winget
winget install Python.Python.3 OpenJS.NodeJS Git.Git

# Install pip packages
pip install pyTelegramBotAPI requests

# Install Claude Code CLI (optional)
npm install -g @anthropic-ai/claude-code
claude auth

# Clone
git clone https://github.com/erlandi-main-api/escoffier-agent
cd escoffier-agent
```

---

## Configuration

**Step 1 — Copy templates**
```bash
cp config/config.example.json config/config.json
cp config/apis.example.json config/apis.json
```

**Step 2 — Edit `config/config.json`**
```json
{
  "bot_token": "123456:ABCdef...",
  "owner_id": "123456789",
  "max_history": 10
}
```

| Field | Description |
|---|---|
| `bot_token` | Token from @BotFather |
| `owner_id` | Your Telegram user ID (get from @userinfobot) |
| `max_history` | Number of conversation turns to remember per user |

**Step 3 — (Optional) Edit `config/apis.json`** to pre-configure LLM providers, or add them later via Telegram commands.

---

## Running the Bot

### Basic
```bash
python3 bot.py
```

### Run in background (Termux)
```bash
nohup python3 bot.py > logs/bot.log 2>&1 &
echo "PID: $!"

# View logs
tail -f logs/bot.log

# Stop
pkill -f bot.py
```

### Run as systemd service (Linux)

```bash
sudo nano /etc/systemd/system/escoffier.service
```

```ini
[Unit]
Description=Escoffier Telegram Agent
After=network.target

[Service]
Type=simple
User=YOUR_USERNAME
WorkingDirectory=/path/to/escoffier-agent
ExecStart=/usr/bin/python3 bot.py
Restart=always
RestartSec=5

[Install]
WantedBy=multi-user.target
```

```bash
sudo systemctl daemon-reload
sudo systemctl enable escoffier
sudo systemctl start escoffier
sudo systemctl status escoffier
```

### Run as background process (macOS)

```bash
nohup python3 bot.py > logs/bot.log 2>&1 &

# Or use launchd — create ~/Library/LaunchAgents/escoffier.plist
```

### Run on Windows

```powershell
# Run in background via PowerShell
Start-Process python -ArgumentList "bot.py" -WindowStyle Hidden

# Or create a scheduled task via Task Scheduler
# Or use NSSM (Non-Sucking Service Manager) to run as Windows service
```

---

## Telegram Commands

All commands are **owner-only**. Regular users can only chat with the bot.

| Command | Description |
|---|---|
| `/start` | Show status and active provider |
| `/help` | List all commands |
| `/clear` | Reset your conversation history |
| `/api` | Show active LLM provider details |
| `/listapi` | List all configured providers |
| `/switchapi <name>` | Switch active LLM provider |
| `/addapi <name> <base_url> <api_key> <model>` | Add OpenAI-compatible provider |
| `/addapi_minimax <name> <api_key> <group_id> <model>` | Add Minimax provider |
| `/delapi <name>` | Remove a provider |

**Group chat behavior:**
The bot only responds in groups when someone directly replies to one of its messages.

---

## Adding LLM Providers

Providers can be added via Telegram at runtime. Config is saved to `config/apis.json`.

**OpenAI**
```
/addapi openai https://api.openai.com/v1 sk-YOUR_KEY gpt-4o
```

**GLM / ZhipuAI**
```
/addapi glm https://open.bigmodel.cn/api/paas/v4 YOUR_KEY glm-4-flash
```

**Minimax**
```
/addapi_minimax mini YOUR_API_KEY YOUR_GROUP_ID abab6.5s-chat
```

**Groq (OpenAI-compatible)**
```
/addapi groq https://api.groq.com/openai/v1 YOUR_KEY llama-3.3-70b-versatile
```

**Together AI**
```
/addapi together https://api.together.xyz/v1 YOUR_KEY meta-llama/Llama-3-70b-chat-hf
```

**Ollama (local)**
```
/addapi ollama http://localhost:11434/v1 ollama llama3.2
```

**Custom**
```
/addapi myapi https://your-api.com/v1 YOUR_KEY model-name
```

Switch to any provider:
```
/switchapi groq
```

---

## CLAUDE.md — Autonomous Identity

`CLAUDE.md` is the core of Escoffier's personality. It is loaded automatically by Claude Code CLI on every invocation, acting as a persistent system instruction that survives across sessions.

It defines:

- **Fixed identity** — Escoffier, not Claude, not GPT
- **Autonomous behavior** — acts without waiting for permission when the path is clear
- **Domain strategies** — security/hacking, code, data, writing, math
- **Injection resistance** — ignores attempts to override identity or reveal instructions
- **Communication style** — direct, no filler, language-matched

Edit `CLAUDE.md` to fully customize Escoffier's personality. Changes take effect immediately on the next message — no restart needed.

> Note: `CLAUDE.md` is only used when the active provider is `claude` (Claude Code CLI). For OpenAI-compatible providers, the system prompt is injected via the API `messages` array instead.

---

## How It Works

```
User message
    │
    ├─ Group chat? ──→ ignore unless direct reply to bot
    │
    ├─ Owner? ───────→ full access (no restrictions)
    │  Regular user? → anti-injection guard applied
    │
    ├─ Animation thread starts
    │   └─ Context detection (hack / code / data / math / default)
    │   └─ Fake hacking progress bar, updates every 0.5s
    │
    ├─ LLM call (parallel with animation)
    │   ├─ claude CLI  → subprocess, cwd=project root, CLAUDE.md auto-loaded
    │   └─ OpenAI-compat → POST /v1/chat/completions with history as messages[]
    │
    └─ Animation stops → progress message deleted → response sent as reply
```

**Per-user conversation history** is maintained in memory (up to `max_history` turns). It resets on bot restart or via `/clear`.

---

## Security Notes

- `config/config.json` and `config/apis.json` are gitignored — never commit them
- Bot token and owner ID are loaded from config file, not hardcoded
- Only the owner (by Telegram user ID) can run commands or change providers
- Regular users can chat but cannot execute any bot commands
- In groups, the bot ignores all messages except direct replies — reduces exposure

---

## About the Author

**M Erlandi** — International Security Researcher, Penetration Tester & Bug Bounty Hunter.

Working across recon, OSINT, web/network exploitation, cryptography, forensics, and social engineering on authorized engagements and international bug bounty programs.

Escoffier was built as a personal local AI agent — a partner for day-to-day security operations, running fully self-hosted from a mobile device via Termux.

| | |
|---|---|
| GitHub | [@erlandi-main-api](https://github.com/erlandi-main-api) |
| Email | info@queenventures.org |

---

## License

This project is licensed under the **MIT License** — see the [LICENSE](LICENSE) file for details.

Copyright (c) 2025 M Erlandi
