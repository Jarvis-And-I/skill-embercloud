---
name: embercloud
description: Set up EmberCloud as a model provider. Adds GLM and MiniMax models with one command.
homepage: https://embercloud.ai
metadata:
  {
    "clawdbot": {
      "emoji": "🔥",
      "primaryEnv": "EMBERCLOUD_API_KEY"
    }
  }
---

# EmberCloud Setup 🔥

[EmberCloud](https://embercloud.ai) provides access to GLM-5, MiniMax, and other powerful models with competitive pricing.

## What You Need From the User

Just the API key: `EMBERCLOUD_API_KEY`

They can get one at: https://embercloud.ai/dashboard/keys

## Fetching Current Models & Pricing

To get the latest models and pricing from EmberCloud:

```bash
curl -s -H "Authorization: Bearer $EMBERCLOUD_API_KEY" https://api.embercloud.ai/v1/models
```

Response contains `data[]` with each model having:
- `id` — model identifier (e.g., `glm-5`)
- `name` — display name
- `context_length` — max context window
- `max_output_length` — max output tokens
- `supported_parameters` — array, check for `"reasoning"` to determine if it's a reasoning model
- `pricing.prompt` — input cost per token (multiply by 1M for $/1M)
- `pricing.completion` — output cost per token
- `pricing.cache_read` — cache read cost per token

## Setup Steps

Once the user provides their API key, use `gateway config.patch` with this config:

### 1. Add the Provider

Patch `models.providers.embercloud`:

```json
{
  "models": {
    "providers": {
      "embercloud": {
        "baseUrl": "https://api.embercloud.ai/v1",
        "apiKey": "${EMBERCLOUD_API_KEY}",
        "api": "openai-completions",
        "models": [
          {
            "id": "glm-5",
            "name": "GLM-5",
            "reasoning": true,
            "input": ["text"],
            "cost": { "input": 1.00, "output": 3.20, "cacheRead": 0.20, "cacheWrite": 0 },
            "contextWindow": 203000,
            "maxTokens": 131000,
            "compat": { "supportsDeveloperRole": false }
          },
          {
            "id": "glm-4.7",
            "name": "GLM-4.7",
            "reasoning": true,
            "input": ["text"],
            "cost": { "input": 0.40, "output": 1.75, "cacheRead": 0.08, "cacheWrite": 0 },
            "contextWindow": 200000,
            "maxTokens": 131000,
            "compat": { "supportsDeveloperRole": false }
          },
          {
            "id": "glm-4.7-flash",
            "name": "GLM-4.7 Flash",
            "reasoning": false,
            "input": ["text"],
            "cost": { "input": 0.06, "output": 0.40, "cacheRead": 0.01, "cacheWrite": 0 },
            "contextWindow": 200000,
            "maxTokens": 131000,
            "compat": { "supportsDeveloperRole": false }
          },
          {
            "id": "glm-4.6",
            "name": "GLM-4.6",
            "reasoning": true,
            "input": ["text"],
            "cost": { "input": 0.43, "output": 1.74, "cacheRead": 0.08, "cacheWrite": 0 },
            "contextWindow": 200000,
            "maxTokens": 128000,
            "compat": { "supportsDeveloperRole": false }
          },
          {
            "id": "glm-4.5-air",
            "name": "GLM-4.5 Air",
            "reasoning": false,
            "input": ["text"],
            "cost": { "input": 0.13, "output": 0.85, "cacheRead": 0.025, "cacheWrite": 0 },
            "contextWindow": 131000,
            "maxTokens": 96000,
            "compat": { "supportsDeveloperRole": false }
          },
          {
            "id": "minimax-m2.5",
            "name": "MiniMax-M2.5",
            "reasoning": true,
            "input": ["text"],
            "cost": { "input": 0.30, "output": 1.10, "cacheRead": 0.03, "cacheWrite": 0 },
            "contextWindow": 196600,
            "maxTokens": 196600
          }
        ]
      }
    }
  }
}
```

### 2. Add Model Aliases

Patch `agents.defaults.models` to add aliases so users can use `/model glm5` instead of `/model embercloud/glm-5`:

```json
{
  "agents": {
    "defaults": {
      "models": {
        "embercloud/glm-5": { "alias": "glm5" },
        "embercloud/glm-4.7": { "alias": "glm47" },
        "embercloud/glm-4.7-flash": { "alias": "glm47flash" },
        "embercloud/glm-4.6": { "alias": "glm46" },
        "embercloud/glm-4.5-air": { "alias": "glm45air" },
        "embercloud/minimax-m2.5": { "alias": "minimax" }
      }
    }
  }
}
```

### 3. Set the Environment Variable

Tell the user to set the env var before starting Clawdbot:

```bash
export EMBERCLOUD_API_KEY="ek_live_..."
```

Or add it to their shell profile / systemd service.

### 4. Restart

Use `gateway restart` to apply the changes. The user can then switch models with `/model glm5`.

## Model Quick Reference

| Alias | Model | Reasoning | Context | Input $/1M | Output $/1M |
|-------|-------|-----------|---------|------------|-------------|
| `glm5` | GLM-5 | ✅ | 203K | $1.00 | $3.20 |
| `glm47` | GLM-4.7 | ✅ | 200K | $0.40 | $1.75 |
| `glm47flash` | GLM-4.7 Flash | ❌ | 200K | $0.06 | $0.40 |
| `glm46` | GLM-4.6 | ✅ | 200K | $0.43 | $1.74 |
| `glm45air` | GLM-4.5 Air | ❌ | 131K | $0.13 | $0.85 |
| `minimax` | MiniMax-M2.5 | ✅ | 196K | $0.30 | $1.10 |

## Troubleshooting

| Issue | Solution |
|-------|----------|
| "EMBERCLOUD_API_KEY not set" | User needs to set the env var and restart |
| "Model not allowed" | Check that aliases are in `agents.defaults.models` |
| Models not appearing after setup | Restart the gateway |

## Links

- 🌐 [EmberCloud](https://embercloud.ai)
- 🔑 [API Keys](https://embercloud.ai/dashboard/keys)
- 📚 [Documentation](https://embercloud.ai/docs)
