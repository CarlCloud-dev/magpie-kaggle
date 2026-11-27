# magpie-kaggle

Kaggle Models provider plugin for [magpie](https://usemagpie.ai) / OpenCode — your Kaggle account's free AI model credits (**$10/day, $100/month**) as a provider: Claude, Gemini, GPT, DeepSeek, Qwen, IBM Granite and more through Kaggle's Model Proxy.

## Models

| model id | upstream |
|---|---|
| `kaggle/sonnet-5` | `anthropic/claude-sonnet-5@default` |
| `kaggle/gemini-3-flash` | `google/gemini-3-flash-preview` |
| `kaggle/gemini-3.1-flash-lite` | `google/gemini-3.1-flash-lite-preview` |
| `kaggle/gpt-5.4-nano` | `openai/gpt-5.4-nano-2026-03-17` |
| `kaggle/gpt-oss-120b` | `openai/gpt-oss-120b` |
| `kaggle/granite-4h` | `ibm/granite-4.0-h-small` |
| `kaggle/deepseek-r1` | `deepseek-ai/deepseek-r1-0528` |
| `kaggle/qwen3-next-80b` | `qwen/qwen3-next-80b-a3b-instruct` |

Kaggle's catalog is larger; add entries to `MODELS` in `index.mjs` with exact catalog ids.

## Install & sign in

```sh
magpie plugin add magpie-kaggle     # or a local path / GitHub URL
magpie plugin login kaggle          # paste your Kaggle API token
```

**Sign-in token**: a Kaggle API token (`KGAT…`), created at [kaggle.com/settings](https://www.kaggle.com/settings) → API → "Create New Token" (or with `kaggle auth login` of the Kaggle CLI; it is then in `~/.kaggle/access_token`). Your account needs phone/identity verification for the Model Proxy. This is the long-lived token — you paste it once.

**Account label**: paste the token followed by a space and a name (e.g. \KGAT… work\) to show that name on the account in magpie — Kaggle has no whoami endpoint for these tokens, so the label is how multiple accounts tell themselves apart. The same key works without a label; re-login to add one.

**Multiple accounts & routing**: log in once per Kaggle account; magpie keeps each account's proxy token and quota windows separate. Models of this provider can join routing groups alongside other providers, and quota refusals carry the window's reset time so the router rests the account exactly until it refills.

## How it works

- The proxy itself takes a **short-lived (~2h) bearer token**. The plugin mints it in memory from your long-lived API token through the same endpoint the official `kaggle benchmarks auth` CLI uses, and renews it automatically a couple of minutes before expiry. The proxy token is never stored; no manual re-login.
- The loader intercepts requests to a dummy base URL and forwards them to the proxy's OpenAI-compatible endpoint (`<baseUri>/openapi/v1`), mapping magpie's short model ids to exact catalog slugs (the upstream 404s on short names).
- Streaming (SSE) is supported.
- The `usage` hook reports the daily $10 / monthly $100 windows with reset times to magpie's Usage page. Quota is shared with everything else using the proxy (e.g. Kaggle Benchmarks runs on the website).

## Requirements & limits

- Verified Kaggle account (phone/identity verification at kaggle.com/settings).
- Rate limits and per-model pricing are undocumented; the quota API reports dollars consumed.
- Chat Completions and Responses paths are forwarded; other endpoints are refused.

## License

MIT
