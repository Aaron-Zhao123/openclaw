---
title: "Sequrity"
---

# Sequrity

[Sequrity](https://sequrity.ai) is a security-focused AI gateway that proxies requests to
underlying models (e.g. Azure OpenAI) while enforcing policy, session tracking, and
multi-step planning via its FSM architecture.

## Endpoint setup

Sequrity exposes an OpenAI-compatible `/v1/chat/completions` endpoint. Configure it as a
custom provider in OpenClaw:

```json
{
  "models": {
    "providers": {
      "sequrity": {
        "baseUrl": "https://api.sequrity.ai/control/chat/sequrity_azure/v1",
        "apiKey": "<your-bearer-token>",
        "api": "openai-completions",
        "authHeader": true,
        "headers": {
          "X-Api-Key": "<your-api-key>",
          "X-Features": "{\"agent_arch\":\"dual-llm\"}",
          "X-Policy": "{\"mode\":\"standard\",\"presets\":{\"default_allow\":true,\"default_allow_enforcement_level\":\"soft\"}}",
          "X-Config": "{\"fsm\":{\"enable_multistep_planning\":true,\"disable_rllm\":true,\"max_n_turns\":50,\"max_pllm_steps\":50,\"max_pllm_failed_steps\":10,\"history_mismatch_policy\":\"restart_turn\"},\"response_format\":{\"strip_response_content\":true}}"
        },
        "models": [
          {
            "id": "gpt-5.2",
            "name": "Sequrity Azure GPT-5.2",
            "contextWindow": 200000,
            "maxTokens": 16384
          }
        ]
      }
    }
  }
}
```

Or via CLI:

```bash
openclaw config set models.providers.sequrity.baseUrl "https://api.sequrity.ai/control/chat/sequrity_azure/v1"
openclaw config set models.providers.sequrity.apiKey "<your-bearer-token>"
```

## Required headers

| Header                          | Description                                        |
| ------------------------------- | -------------------------------------------------- |
| `Authorization: Bearer <token>` | Bearer token for authentication (set via `apiKey`) |
| `X-Api-Key`                     | Secondary API key                                  |
| `X-Features`                    | Feature flags (e.g. `agent_arch`)                  |
| `X-Policy`                      | Policy configuration (allow/deny rules)            |
| `X-Config`                      | FSM and response format configuration              |

## Tool call ID format

Sequrity encodes session state in tool call IDs. The format is:

```
tc-<session-uuid>-<call-uuid>
```

These IDs must be echoed back **verbatim** in tool results — do not strip or transform
them. OpenClaw's `openai-completions` transport preserves IDs unchanged, which is required
for Sequrity's session tracking to work correctly.

**Note:** The Sequrity server may also be configured to return alphanumeric-only IDs
(dashes stripped), in which case no special handling is needed on the client side.

## Local development

To point OpenClaw at a local Sequrity instance (e.g. running on a Tailscale peer):

```bash
openclaw config set models.providers.sequrity.baseUrl "http://<tailscale-ip>:8000/control/chat/sequrity_azure/v1"
```

Restore to production:

```bash
openclaw config set models.providers.sequrity.baseUrl "https://api.sequrity.ai/control/chat/sequrity_azure/v1"
```
