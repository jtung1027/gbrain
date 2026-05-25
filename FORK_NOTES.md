# Fork Notes — DeepSeek Think Support

Fork of [garrytan/gbrain](https://github.com/garrytan/gbrain) v0.41.0.0
with fixes to support DeepSeek as the LLM provider for the `think` synthesis tool.

## Why This Fork

gbrain's `think` tool was designed for Anthropic's structured JSON output mode.
Three barriers prevent DeepSeek from working:

1. DeepSeek doesn't produce strict JSON → `LLM_OUTPUT_NOT_JSON` warnings
2. No `deepseek_api_key` field in config → can't store key for MCP subprocesses
3. `process.env` spread includes `undefined` values → config-file keys silently overwritten

## Install

```bash
bun install -g github:jtung1027/gbrain#deepseek-think-support
```

Then configure:

```bash
# Set chat model
gbrain config set chat_model deepseek:deepseek-chat

# Store API key (edit ~/.gbrain/config.json):
#   "deepseek_api_key": "sk-..."
```

Restart Cursor's gbrain MCP server after installing.

## Changes (4 commits ahead of upstream)

| Commit | File | Change |
|--------|------|--------|
| `bf6773e0` | `think/index.ts` | Add `jsonrepair` fallback for non-Anthropic JSON parsing |
| `bb429244` | `config.ts`, `cli.ts` | Add `deepseek_api_key` to config + gateway wiring |
| `71756523` | `think/index.ts`, `cli.ts` | Temp debug logging (can be reverted) |
| `3c002a57` | `cli.ts` | **`filterDefinedEnv()`** — strip `undefined` from `process.env` spread |

## Key Finding: `filterDefinedEnv`

The root bug: `{ ...envFromConfig, ...process.env }` in `buildGatewayConfig()`
includes `undefined` values from unset env vars, which **overwrite** config-file
API keys. This affects all config-stored API keys in MCP/daemon subprocesses,
not just DeepSeek. Should be upstreamed.

## Upstreaming

```bash
# Sync with upstream
git fetch origin
git checkout master && git merge origin/master
git checkout deepseek-think-support && git rebase master
git push fork deepseek-think-support --force-with-lease
```

Capture in gbrain: search for `gbrain deepseek fix` or see `concepts/gbrain-deepseek-integration`.
