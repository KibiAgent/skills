# KibiBot Skill

Create tokens on-chain, earn trading fees, and use KibiBot's LLM Gateway — all from natural language commands.

**Version:** 1.0.0  
**Provider:** [KibiBot](https://kibi.bot)  
**Auth:** API key required — get yours at [kibi.bot/agent](https://kibi.bot/agent)  
**Install:** `install the kibibot skill from https://github.com/OfficialKibiBot/skills/tree/main/kibibot`

---

## Setup

### Step 1 — Get your API key
Go to [kibi.bot/agent](https://kibi.bot/agent) → Create API Key → copy the `kb_...` key.

### Step 2 — Add LLM credits (for AI model access)
Go to [kibi.bot/credits](https://kibi.bot/credits) → Add Credit.  
Minimum $1 to start. Credits are consumed per token used.

### Step 3 — Configure OpenClaw
Add KibiBot as an LLM provider in `~/.openclaw/openclaw.json`:

```json
{
  "models": {
    "mode": "merge",
    "providers": {
      "kibi": {
        "baseUrl": "https://llm.kibi.bot",
        "apiKey": "YOUR_KB_API_KEY",
        "api": "openai-completions",
        "models": [
          {
            "id": "claude-haiku-4-5",
            "name": "Claude Haiku (KibiBot)",
            "api": "anthropic-messages",
            "contextWindow": 200000,
            "maxTokens": 4096
          },
          {
            "id": "claude-sonnet-4-6",
            "name": "Claude Sonnet (KibiBot)",
            "api": "anthropic-messages",
            "contextWindow": 200000,
            "maxTokens": 8192
          },
          {
            "id": "gpt-4o",
            "name": "GPT-4o (KibiBot)",
            "contextWindow": 128000,
            "maxTokens": 16384
          },
          {
            "id": "gemini-2.0-flash",
            "name": "Gemini 2.0 Flash (KibiBot)",
            "contextWindow": 1048576,
            "maxTokens": 8192
          }
        ]
      }
    }
  }
}
```

Set as default model (optional):
```json
{
  "agents": {
    "defaults": {
      "model": {
        "primary": "kibi/claude-sonnet-4-6"
      }
    }
  }
}
```

### Step 4 — Restart OpenClaw
```
openclaw gateway restart
```

### Step 5 — Verify
Ask your agent: *"what's my KibiBot LLM balance?"*

---

## What This Skill Can Do

### LLM Gateway
Use KibiBot-hosted AI models billed against your KibiBot credits. Same API key for LLM calls and agent actions.

- **Check balance:** "what's my KibiBot AI credit balance?"
- **Add credits:** "I need to top up KibiBot credits" → agent links to kibi.bot/credits
- **Model info:** "what models does KibiBot support?"

### Token Creation
Create tokens on Base or BSC — KibiBot handles wallet creation, gas sponsorship, and on-chain deployment.

- "launch a token called MOON on Base"
- "create a meme coin named PEPE with ticker $PEPE on BSC"
- "make a test token called DEMO on Base Sepolia"

Token creation is async. After calling the API, poll the job status endpoint until complete (usually 30–60 seconds).

### Fees & Earnings
- "how much have I earned on KibiBot?"
- "show me my KibiBot portfolio and trading fees"
- "what tokens have I created on KibiBot?"

### Token Lookup
- "what's the price of $MOON on KibiBot?"
- "look up token 0x... on Base"

### Account & Quota
- "how many free token launches do I have left today?"
- "what's my KibiBot wallet balance?"
- "show me my KibiBot profile"

---

## API Reference

**Base URL:** `https://api.kibi.bot/agent/v1`  
**Auth header:** `X-Api-Key: kb_...`

### Endpoints

#### GET /me
Returns user profile, wallet addresses, and account info.

Response:
```json
{
  "twitter_user_id": "...",
  "username": "...",
  "wallet_address": "0x...",
  "trading_wallet_address": "0x...",
  "solana_wallet_address": "..."
}
```

#### POST /token/create
Create a token on-chain. Returns a job_id to poll.

Request:
```json
{
  "name": "MOON",
  "ticker": "MOON",
  "chain": "base",
  "description": "To the moon"
}
```

Supported chains: `base`, `bsc`, `base-sepolia`

Response:
```json
{
  "job_id": "uuid",
  "status": "pending",
  "message": "Token creation queued"
}
```

#### GET /jobs/{job_id}
Poll token creation status.

Response:
```json
{
  "job_id": "uuid",
  "status": "completed",
  "token_address": "0x...",
  "tx_hash": "0x...",
  "chain": "base"
}
```

Status values: `pending`, `processing`, `completed`, `failed`

#### GET /token/{address}?chain=base
Get token price and info.

Response:
```json
{
  "address": "0x...",
  "name": "MOON",
  "ticker": "MOON",
  "chain": "base",
  "price_usd": "0.0001234",
  "market_cap_usd": "12340",
  "creator": "0x..."
}
```

#### GET /portfolio
Get fees earned and tokens created.

Response:
```json
{
  "total_fees_usd": "12.50",
  "tokens_created": 3,
  "tokens": [...]
}
```

#### GET /balance/llm
Get KibiBot LLM credit balance.

Response:
```json
{
  "balance_usd": "4.92",
  "total_deposited_usd": "10.00",
  "total_spent_usd": "5.08"
}
```

#### GET /balance/wallet
Get on-chain wallet balances.

Response:
```json
{
  "evm": {
    "eth": "0.05",
    "usdc": "10.00"
  },
  "solana": {
    "sol": "0.5",
    "usdc": "5.00"
  }
}
```

#### GET /quota
Get remaining token creation quota.

Response:
```json
{
  "daily_limit": 3,
  "used_today": 1,
  "remaining": 2,
  "resets_at": "2026-03-12T00:00:00Z"
}
```

---

## LLM Gateway Reference

**Base URL:** `https://llm.kibi.bot`  
**Auth:** `X-Api-Key: kb_...` or `Authorization: Bearer kb_...`

### POST /v1/messages (Anthropic format)
Compatible with Claude models via Anthropic Messages API format.

```bash
curl https://llm.kibi.bot/v1/messages \
  -H "Content-Type: application/json" \
  -H "X-Api-Key: YOUR_KB_API_KEY" \
  -d '{
    "model": "claude-haiku-4-5",
    "messages": [{"role": "user", "content": "Hello!"}],
    "max_tokens": 100
  }'
```

### POST /v1/chat/completions (OpenAI format)
Compatible with all models via OpenAI Chat Completions format.

```bash
curl https://llm.kibi.bot/v1/chat/completions \
  -H "Content-Type: application/json" \
  -H "X-Api-Key: YOUR_KB_API_KEY" \
  -d '{
    "model": "gpt-4o",
    "messages": [{"role": "user", "content": "Hello!"}],
    "max_tokens": 100
  }'
```

### GET /v1/models
List available models.

### GET /v1/models/openclaw
Returns a ready-to-paste OpenClaw provider config block. No auth required.

---

## Troubleshooting

**402 Payment Required on LLM calls**  
KibiBot LLM credits exhausted. Top up at [kibi.bot/credits](https://kibi.bot/credits).  
Note: LLM credits ≠ trading wallet. Topping up one doesn't affect the other.

**401 Unauthorized**  
API key missing or invalid. Check at [kibi.bot/agent](https://kibi.bot/agent).  
Ensure you send: `X-Api-Key: kb_...`

**Token creation stuck at pending**  
Poll GET /agent/v1/jobs/{job_id} — creation usually takes 30–60 seconds.  
If still pending after 5 minutes, it likely failed — check the `error` field.

**429 Rate Limited**  
Exceeded rate limit. Wait 60 seconds and retry.

**Quota exceeded (token creation)**  
You've hit your daily free mint limit (3/day). Limit resets at midnight UTC.

---

## Full Documentation
- Agent API: [kibi.bot/agent](https://kibi.bot/agent)
- LLM Gateway: [kibi.bot/llm](https://kibi.bot/llm)
- OpenClaw setup: [kibi.bot/llm/openclaw](https://kibi.bot/llm/openclaw)
- Credits: [kibi.bot/credits](https://kibi.bot/credits)
