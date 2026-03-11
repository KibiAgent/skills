# KibiBot Agent API — Quick Reference

**Base URL:** `https://api.kibi.bot/agent/v1`  
**Auth:** `X-Api-Key: kb_...` header on all requests

---

## Endpoints

| Method | Path | Description |
|--------|------|-------------|
| GET | `/me` | User profile & wallet addresses |
| POST | `/token/create` | Create token on-chain (async) |
| GET | `/jobs/{job_id}` | Poll token creation status |
| GET | `/token/{address}` | Token price & info (`?chain=base`) |
| GET | `/portfolio` | Fees earned & tokens created |
| GET | `/balance/llm` | LLM credit balance |
| GET | `/balance/wallet` | On-chain wallet balances |
| GET | `/quota` | Daily token creation quota |

---

## POST /token/create

```json
{ "name": "MOON", "ticker": "MOON", "chain": "base", "description": "..." }
```

Chains: `base` · `bsc` · `base-sepolia`  
Returns: `{ "job_id": "uuid", "status": "pending" }`  
Poll `/jobs/{job_id}` until `status` is `completed` or `failed`.

---

## GET /jobs/{job_id}

```json
{
  "job_id": "uuid",
  "status": "completed",
  "token_address": "0x...",
  "tx_hash": "0x...",
  "chain": "base"
}
```

---

## Error Codes

| Code | Meaning |
|------|---------|
| 401 | Missing or invalid API key |
| 402 | Insufficient LLM credits |
| 422 | Validation error (check request body) |
| 429 | Rate limited — wait 60s |
| 500 | Server error — retry |

---

## LLM Gateway

**Base URL:** `https://llm.kibi.bot`  
Same `X-Api-Key` header.  
- `POST /v1/messages` — Anthropic format (Claude models)  
- `POST /v1/chat/completions` — OpenAI format (all models)  
- `GET /v1/models` — list models  
- `GET /v1/models/openclaw` — ready-to-paste OpenClaw config
