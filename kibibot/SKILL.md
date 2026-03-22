---
name: kibibot
description: Create tokens on-chain, check Kibi Credit balance, trigger agent credit reload, and interact with KibiBot's Agent API and Kibi LLM Gateway. Use when asked to create a token via KibiBot, check KibiBot Kibi Credit balance, check daily token creation quota, reload credits from trading wallet, or make LLM calls through KibiBot's gateway.
---

# KibiBot Skill

Create tokens on-chain, earn trading fees, and use KibiBot's Kibi LLM Gateway — all from natural language commands.

**Version:** 1.4.0  
**Provider:** [KibiBot](https://kibi.bot)  
**Auth:** API key required — get yours at [kibi.bot/settings/api-keys](https://kibi.bot/settings/api-keys)  
**Install:** `install the kibibot skill from https://github.com/KibiAgent/skills/tree/main/kibibot`

---

## Setup

### Step 1 — Get your API key
Go to [kibi.bot/settings/api-keys](https://kibi.bot/settings/api-keys) → Create API Key → copy the `kb_...` key.

> **Permissions:** Base API is always included. Enable **Kibi LLM Gateway** if you want to use AI models. Enable **Agent Reload** if you want the agent to top up your Kibi Credits automatically from your trading wallet.

### Step 2 — Add Kibi Credits (for AI model access)
Go to [kibi.bot/credits](https://kibi.bot/credits) → Add Credit.  
Minimum $1 to start. Credits are consumed per token used.

### Step 3 — Set up Kibi LLM Gateway *(optional)*
> This step registers KibiBot as your agent's AI model provider, so your agent *thinks* using Claude/GPT/Gemini billed to your Kibi Credits — instead of paying Anthropic/OpenAI directly. It's separate from the Agent API skill.
>
> **OpenClaw users:** follow the config below. If you're using LangChain, CrewAI, or any OpenAI-compatible framework, point your `base_url` to `https://llm.kibi.bot/v1` with your `kb_...` API key instead.
>
> **OpenClaw users** — add KibiBot as an LLM provider in `~/.openclaw/openclaw.json`:

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
            "name": "Kibi Haiku",
            "api": "anthropic-messages",
            "contextWindow": 200000,
            "maxTokens": 4096
          },
          {
            "id": "claude-sonnet-4-6",
            "name": "Kibi Sonnet",
            "api": "anthropic-messages",
            "contextWindow": 200000,
            "maxTokens": 8192
          },
          {
            "id": "claude-opus-4-6",
            "name": "Kibi Opus",
            "api": "anthropic-messages",
            "contextWindow": 200000,
            "maxTokens": 32000
          },
          {
            "id": "gpt-4o",
            "name": "Kibi GPT-4o",
            "contextWindow": 128000,
            "maxTokens": 16384
          },
          {
            "id": "gpt-4o-mini",
            "name": "Kibi GPT-4o Mini",
            "contextWindow": 128000,
            "maxTokens": 16384
          },
          {
            "id": "gemini-2.5-flash",
            "name": "Kibi Gemini Flash",
            "contextWindow": 1048576,
            "maxTokens": 8192
          },
          {
            "id": "gemini-2.5-pro",
            "name": "Kibi Gemini Pro",
            "contextWindow": 2097152,
            "maxTokens": 8192
          }
        ]
      }
    }
  },
  "agents": {
    "defaults": {
      "models": {
        "kibi/claude-haiku-4-5":  { "alias": "kibi-haiku" },
        "kibi/claude-sonnet-4-6": { "alias": "kibi-sonnet" },
        "kibi/claude-opus-4-6":   { "alias": "kibi-opus" },
        "kibi/gpt-4o":            { "alias": "kibi-gpt4o" },
        "kibi/gpt-4o-mini":       { "alias": "kibi-gpt4o-mini" },
        "kibi/gemini-2.5-flash":  { "alias": "kibi-gemini-flash" },
        "kibi/gemini-2.5-pro":    { "alias": "kibi-gemini-pro" }
      }
    }
  }
}
```

> **Note:** The `agents.defaults.models` block is required — it allowlists the models and registers aliases so both the model dropdown picker and `/model` command work correctly. The `alias` must match the model `id` exactly.

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

Then restart OpenClaw:
```
openclaw gateway restart
```

Switch models using the dropdown picker or `/model` command:
```
/model kibi-haiku
/model kibi-sonnet
/model kibi-opus
/model kibi-gpt4o
/model kibi-gpt4o-mini
/model kibi-gemini-flash
/model kibi-gemini-pro
```

### Available Models

| Model ID | Provider | Context |
|---|---|---|
| `claude-haiku-4-5` | Anthropic | 200k |
| `claude-sonnet-4-6` | Anthropic | 200k |
| `claude-opus-4-6` | Anthropic | 200k |
| `gpt-4o` | OpenAI | 128k |
| `gpt-4o-mini` | OpenAI | 128k |
| `gemini-2.5-flash` | Google | 1M |
| `gemini-2.5-pro` | Google | 2M |

Verify by asking your agent: *"what's my KibiBot Kibi Credit balance?"*

---

## What This Skill Can Do

### Kibi LLM Gateway
Use KibiBot-hosted AI models billed against your Kibi Credits. Same API key for LLM calls and agent actions.

- **Check balance:** "what's my KibiBot Kibi Credit balance?"
- **Add credits:** "I need to top up KibiBot credits" → agent links to kibi.bot/credits
- **Reload credits:** "reload my KibiBot credits from my trading wallet" (requires Agent Reload permission + configured on Credits page)
- **Model info:** "what models does KibiBot support?"

### Token Creation
Create tokens on Base, BSC, or Solana — KibiBot handles wallet creation, gas sponsorship, and on-chain deployment.

- "launch a token called MOON on Base"
- "create a meme coin named PEPE with ticker $PEPE on BSC"
- "make a test token called DEMO on Base Sepolia"

Token creation is async. After calling the API, poll the job status endpoint until complete (usually 30–60 seconds).

### Created Tokens
- "what tokens have I created on KibiBot?"
- "show my KibiBot token portfolio"

### Quota
- "how many free token launches do I have left today?"
- "what's my KibiBot quota per chain?"

### Wallet Balances
- "what's my KibiBot wallet balance?"
- "show my ETH, BNB, SOL and stablecoin balances on KibiBot"

### Token Lookup
- "what's the price of $MOON on KibiBot?"
- "look up token 0x... on Base"

### Account
- "show me my KibiBot profile"
- "what's my KibiBot Twitter username and wallet address?"

### Skills
- "what can KibiBot do?" → calls GET /agent/v1/skills

---

## API Reference

**Base URL:** `https://api.kibi.bot/agent/v1`  
**Auth header:** `X-Api-Key: kb_...`

---

### GET /me
Returns user profile, wallet addresses, and account info.

Response:
```json
{
  "twitter_user_id": "...",
  "twitter_username": "...",
  "profile_image_url": "...",
  "followers_count": 1234,
  "joined_at": "2025-01-01T00:00:00Z"
}
```

---

### GET /skills
List all available KibiBot Agent API capabilities with examples. No auth required.

Response:
```json
{
  "skills": [
    {
      "name": "token_create",
      "description": "Deploy a new token on Base, BSC, or Solana",
      "example": "POST /agent/v1/token/create {\"name\": \"MyToken\", \"symbol\": \"MTK\", \"chain\": \"base\"}"
    }
  ],
  "total": 9
}
```

---

### POST /token/create
Create a token on-chain (async). Returns a `job_id` to poll.

Request:
```json
{
  "name": "MOON",
  "symbol": "MOON",
  "chain": "base",
  "description": "To the moon",
  "image_url": "https://...",
  "platform": "basememe"
}
```

- `chain`: `base` | `bsc` | `solana`
- `platform` (optional): `basememe` | `clanker` | `flap` | `fourmeme` | `pumpfun` — defaults to chain default if omitted
- `image_url` (optional): HTTP/HTTPS URL or IPFS URI

Response (`202 Accepted`):
```json
{
  "job_id": 12345,
  "status": "pending",
  "chain": "base",
  "quota": {
    "chain": "base",
    "free_used_today": 1,
    "free_limit": 3,
    "sponsored_remaining": 2
  }
}
```

Pre-check errors:
- `403 insufficient_followers` — need minimum followers to create tokens
- `402 insufficient_balance` — free quota used, trading wallet balance too low
- `429 daily_cap_exceeded` — absolute daily cap reached across all chains

---

### GET /jobs/{job_id}
Poll token creation status.

Response:
```json
{
  "job_id": 12345,
  "status": "completed",
  "chain": "base",
  "token_address": "0x...",
  "error": null,
  "created_at": "2026-01-01T00:00:00Z",
  "completed_at": "2026-01-01T00:01:00Z"
}
```

Status values: `pending` | `processing` | `completed` | `failed`

---

### GET /token/{address}
Get token info by address.

Query: `?chain=base` (optional — searches all chains if omitted)

Response:
```json
{
  "token_address": "0x...",
  "name": "MOON",
  "symbol": "MOON",
  "chain": "base",
  "platform": "basememe",
  "creator_twitter_username": "...",
  "price_usd": "0.0001234",
  "market_cap_usd": "12340",
  "volume_24h_usd": "500",
  "creator_reward_usd": "1.23",
  "created_at": "2026-01-01T00:00:00Z"
}
```

---

### GET /tokens/created
Get paginated list of tokens you have created.

Query: `?page=1&page_size=20`

Response:
```json
{
  "tokens": [
    {
      "token_address": "0x...",
      "name": "MOON",
      "symbol": "MOON",
      "chain": "base",
      "platform": "basememe",
      "created_at": "2026-01-01T00:00:00Z"
    }
  ],
  "total": 5,
  "page": 1,
  "page_size": 20,
  "has_more": false
}
```

---

### GET /balance/credits
Get Kibi Credit balance and agent reload configuration.

Response:
```json
{
  "balance_usd": "4.92",
  "balance_usd_cents": 492,
  "agent_reload": {
    "enabled": true,
    "amount_usd": 5.0,
    "daily_limit_usd": 100.0,
    "chains": ["base"]
  }
}
```

`agent_reload` is `null` if not configured. `enabled: false` means agent reload is off.

---

### POST /balance/credits/reload
Trigger a Kibi Credit reload from the trading wallet.

**Requirements:**
- User must have enabled Agent Reload at [kibi.bot/credits](https://kibi.bot/credits)
- The API key must have `reload_enabled = true`
- The API key must have `llm_enabled = true`
- Trading wallet must have sufficient USDC/USDT on at least one configured chain
- Daily reload limit must not be exceeded

This is **manually triggered by the agent** — there is no automatic background polling. Call this endpoint when the agent determines a credit reload is needed (e.g. before a long task).

Response:
```json
{
  "success": true,
  "amount_usd": "5.00",
  "tx_hash": "0x...",
  "new_balance_usd": "9.92",
  "daily_used_usd": "5.00",
  "daily_remaining_usd": "95.00"
}
```

Errors:
- `403` — reload not enabled for user or key
- `429` — daily limit would be exceeded
- `400` — insufficient stablecoin balance on all configured chains
- `500` — transaction failed

---

### POST /balance/credits/reload/disable
Emergency kill switch — agent disables its own reload permission. **Cannot be re-enabled by the agent** (human must re-enable via dashboard).

Use this if the agent detects unexpected reload behaviour or wants to self-limit spending.

Response: `204 No Content`

---

### GET /balance/wallet
Get on-chain wallet balances across all chains (main + trading wallets).

Response:
```json
{
  "evm_main": {
    "address": "0x...",
    "balance_eth": "0.050000",
    "balance_bnb": "0.100000",
    "balance_usdc_base": "10.000000",
    "balance_usdt_bsc": "5.000000"
  },
  "evm_trading": {
    "address": "0x...",
    "balance_eth": "0.010000",
    "balance_bnb": "0.005000",
    "balance_usdc_base": "0.000000",
    "balance_usdt_bsc": "0.000000"
  },
  "solana_main": {
    "address": "...",
    "balance_sol": "0.500000",
    "balance_usdc_solana": "5.000000"
  },
  "solana_trading": {
    "address": "...",
    "balance_sol": "0.050000",
    "balance_usdc_solana": "0.000000"
  }
}
```

Any field may be `null` if the wallet is not set up or the RPC is unavailable. Check `*_error` fields (e.g. `eth_error: "rpc_unavailable"`) for failure reasons.

---

### GET /quota
Get daily token creation quota and trading wallet readiness per chain.

Response:
```json
{
  "chains": [
    {
      "chain": "base",
      "free_used_today": 1,
      "free_limit": 3,
      "sponsored_remaining": 2,
      "can_create_paid": true,
      "trading_wallet_balance": "0.010000 ETH",
      "trading_wallet_address": "0x..."
    },
    {
      "chain": "bsc",
      "free_used_today": 0,
      "free_limit": 3,
      "sponsored_remaining": 3,
      "can_create_paid": false,
      "trading_wallet_balance": "0.000000 BNB",
      "trading_wallet_address": "0x..."
    },
    {
      "chain": "solana",
      "free_used_today": 0,
      "free_limit": 3,
      "sponsored_remaining": 3,
      "can_create_paid": true,
      "trading_wallet_balance": "0.050000 SOL",
      "trading_wallet_address": "..."
    }
  ]
}
```

---

## Kibi LLM Gateway Reference

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

## Error Codes

| Code | Meaning |
|------|---------|
| 401 | Missing or invalid API key |
| 402 | Insufficient Kibi Credits (LLM) or trading wallet balance (token creation) |
| 403 | Permission denied — feature not enabled for this key or user |
| 404 | Resource not found |
| 422 | Validation error — check request body |
| 429 | Rate limited or daily cap exceeded — wait before retrying |
| 500 | Server error — retry |

---

## Troubleshooting

**402 on LLM calls**  
Kibi Credits exhausted. Top up at [kibi.bot/credits](https://kibi.bot/credits).  
Note: Kibi Credits ≠ trading wallet. Topping up one doesn't affect the other.

**403 on reload**  
Either Agent Reload is not enabled for the user ([kibi.bot/credits](https://kibi.bot/credits) → Agent Reload section), or the API key doesn't have `reload_enabled`. Check both.

**401 Unauthorized**  
API key missing or invalid. Manage keys at [kibi.bot/settings/api-keys](https://kibi.bot/settings/api-keys).  
Ensure you send: `X-Api-Key: kb_...`

**Token creation stuck at pending**  
Poll `GET /agent/v1/jobs/{job_id}` — creation usually takes 30–60 seconds.  
If still pending after 5 minutes, check the `error` field.

**429 on reload**  
Daily reload limit exceeded. Check `daily_remaining_usd` in the balance response.

**400 on reload — insufficient balance**  
No configured chain has enough USDC/USDT in the trading wallet. Check `GET /balance/wallet` and top up.

---

## Full Documentation
- Agent API: [kibi.bot/agent](https://kibi.bot/agent)
- API Keys: [kibi.bot/settings/api-keys](https://kibi.bot/settings/api-keys)
- Kibi LLM Gateway: [kibi.bot/llm](https://kibi.bot/llm)
- OpenClaw setup: [kibi.bot/llm/openclaw](https://kibi.bot/llm/openclaw)
- Kibi Credits: [kibi.bot/credits](https://kibi.bot/credits)
