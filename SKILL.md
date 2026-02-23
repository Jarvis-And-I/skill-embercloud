---
name: embercloud
description: Set up EmberCloud as a model provider. Fetches current models dynamically from the API.
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

[EmberCloud](https://embercloud.ai) provides access to GLM, MiniMax, and other powerful models with competitive pricing.

## What You Need From the User

Just the API key: `EMBERCLOUD_API_KEY`

They can get one at: https://embercloud.ai/dashboard/keys

## Setup Steps

### 1. Fetch Current Models

The models endpoint is public (no auth required):

```bash
curl -s https://api.embercloud.ai/v1/models
```

Response format:
```json
{
  "object": "list",
  "data": [
    {
      "id": "glm-5",
      "name": "GLM-5",
      "context_length": 203000,
      "max_output_length": 131000,
      "input_modalities": ["text"],
      "supported_parameters": ["messages", "model", "stream", "reasoning", ...],
      "pricing": {
        "prompt": "0.000001",
        "completion": "0.0000032",
        "cache_read": "0.0000002"
      }
    }
  ]
}
```

### 2. Transform to Clawdbot Config

For each model in `data[]`, create a model config entry:

| API Field | Config Field | Transform |
|-----------|--------------|-----------|
| `id` | `id` | Direct |
| `name` | `name` | Direct |
| `context_length` | `contextWindow` | Direct |
| `max_output_length` | `maxTokens` | Direct |
| `input_modalities` | `input` | Direct |
| `supported_parameters` | `reasoning` | `true` if array contains `"reasoning"` |
| `pricing.prompt` | `cost.input` | Multiply by 1,000,000 |
| `pricing.completion` | `cost.output` | Multiply by 1,000,000 |
| `pricing.cache_read` | `cost.cacheRead` | Multiply by 1,000,000 |

Additional fields to set:
- `cost.cacheWrite`: `0`
- `compat.supportsDeveloperRole`: `false` (GLM models don't support developer role)

### 3. Build the Config Patch

Use `gateway config.patch` with this structure:

```json
{
  "models": {
    "providers": {
      "embercloud": {
        "baseUrl": "https://api.embercloud.ai/v1",
        "apiKey": "${EMBERCLOUD_API_KEY}",
        "api": "openai-completions",
        "models": [
          // ... transformed models from API
        ]
      }
    }
  }
}
```

### 4. Add Model Aliases

Create intuitive aliases for common models. Suggested alias patterns:

| Model ID | Suggested Alias |
|----------|-----------------|
| `glm-5` | `glm5` |
| `glm-4.7` | `glm47` |
| `glm-4.7-flash` | `glm47flash` |
| `glm-4.6` | `glm46` |
| `glm-4.5` | `glm45` |
| `glm-4.5-air` | `glm45air` |
| `minimax-m2.5` | `minimax` |

Add to config patch:

```json
{
  "agents": {
    "defaults": {
      "models": {
        "embercloud/glm-5": { "alias": "glm5" },
        // ... etc for each model
      }
    }
  }
}
```

### 5. Set Environment Variable

Tell the user to set the env var:

```bash
export EMBERCLOUD_API_KEY="ek_live_..."
```

### 6. Apply and Restart

Use `gateway config.patch` with the combined config, then `gateway restart`.

## Example: Full Transform

Given this API response for one model:
```json
{
  "id": "glm-5",
  "name": "GLM-5",
  "context_length": 203000,
  "max_output_length": 131000,
  "input_modalities": ["text"],
  "supported_parameters": ["messages", "model", "reasoning"],
  "pricing": {
    "prompt": "0.000001",
    "completion": "0.0000032",
    "cache_read": "0.0000002"
  }
}
```

Transform to:
```json
{
  "id": "glm-5",
  "name": "GLM-5",
  "reasoning": true,
  "input": ["text"],
  "cost": {
    "input": 1.00,
    "output": 3.20,
    "cacheRead": 0.20,
    "cacheWrite": 0
  },
  "contextWindow": 203000,
  "maxTokens": 131000,
  "compat": { "supportsDeveloperRole": false }
}
```

## Troubleshooting

| Issue | Solution |
|-------|----------|
| "EMBERCLOUD_API_KEY not set" | User needs to set env var and restart |
| "Model not allowed" | Check that model is in `agents.defaults.models` |
| Models not appearing | Restart the gateway after config patch |

## Links

- 🌐 [EmberCloud](https://embercloud.ai)
- 🔑 [API Keys](https://embercloud.ai/dashboard/keys)
- 📚 [API Docs](https://embercloud.ai/docs/models)
