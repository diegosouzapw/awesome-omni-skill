---
name: velvet-unicorn
description: Query VelvetDAO Unicorn AI (/ask) for token analysis, wallet analysis, and trade analysis. Use when users ask about a token, a wallet, or “should I trade/buy/sell” and you want a structured {answer, txdata}.
compatibility: Requires internet access and python3 on PATH.
metadata: {"openclaw":{"emoji":"🦄","skillKey":"velvet-unicorn","requires":{"bins":["python3"]},"primaryEnv":"VELVET_UNICORN_API_KEY"}}
---

# Velvet Unicorn (AI /ask)

Use this skill to get a structured, AI-generated response from:
`POST https://gentweet.velvetdao.xyz/ask`

It’s meant to cover:
- token analyses (tickers, addresses, chain context)
- wallet analyses (EVM/Solana addresses, holdings, performance)
- trade analyses (may return tx data to *simulate/prepare*, not broadcast)

## What you send

Send a JSON body matching this shape:

```json
{
  "question": "string",
  "context": "string",
  "userid": "string | null",
  "userid_sol": "string | null",
  "previous_questions": ["string", "..."]
}
```

Notes:
- `context` should be a short, relevant summary (not the whole chat).
- `userid` is typically the user’s EVM wallet address (0x…).
- `userid_sol` is typically the user’s Solana address.
- `previous_questions` should be the last ~3 relevant user questions (optional).

## What you expect back

The endpoint should return a dict like:

```json
{
  "meta_category": 0,
  "vu_category": 1,
  "answer": "string",
  "txdata": { "..." : "..." } 
}
```

- `answer` is what you tell the user.
- `txdata` may be null/empty; if present, treat it as *unsigned / unbroadcast* intent.

## How to call it (preferred)

Use the helper script:

```bash
python3 {baseDir}/scripts/ask_unicorn.py \
  --question "Analyze $TOKEN and summarize risk + momentum" \
  --context "User is interested in short-term swing; medium risk tolerance." \
  --userid "0xabc..." \
  --userid-sol "" \
  --previous-questions '["what do you think of $TOKEN", "compare it to $OTHER"]'
```

Or use the shell wrapper:

```bash
bash {baseDir}/scripts/ask_unicorn.sh "Analyze $TOKEN" "Context here" "0xabc..." "" '["prev q1","prev q2"]'
```

## How to respond to the user

1. Run the script and parse the JSON output.
2. Show `answer` directly.
3. If `txdata` is present:
   - present it clearly as “proposed transaction data”
   - ask for explicit confirmation before any signing/broadcast workflow
   - never claim execution happened unless you have an onchain confirmation

## Environment variables (optional)

- `VELVET_UNICORN_ENDPOINT` to override the default endpoint.
- `VELVET_UNICORN_API_KEY` if your endpoint requires auth (sent as `Authorization: Bearer …`).

If you’re using OpenClaw config injection, per-skill env can be set under `skills.entries.<skillKey>.env`.
