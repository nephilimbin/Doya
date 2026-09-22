# Model Management Guide

> [中文](../../modules/llm.md) · English

> Configure AI provider connections for **AI filtering and parsing of articles**. Entry point: Settings → Model Management. The platform catalog is built in (major domestic and international cloud providers + local inference engines); your job is not to "add a platform" but to **connect** the platforms you have API keys for.

> [!WARNING]
> This page is unrelated to the agent on the "Agent" page: the agent uses its own (Claude Code / Codex) account authentication. See the [Agent Guide](agent.md).

## Connect a Platform

1. Select a provider in the left platform list (searchable; logo states: configured / unreachable / unconfigured; local platforms are marked "local")
2. On the right, "Connection Settings":

| Field | Description |
| ---- | ---- |
| Display name | Name shown in the list; auto-saved on blur |
| API URL | Pre-filled with the factory default; usually no need to change (only for your own proxy/private deployment; clearing it and saving = restore the factory URL) |
| API key | The key you obtained from the provider; not needed for local platforms (Ollama / vLLM / llama.cpp / LM Studio) |

3. Click "**Save Connection**" — it **tests the connection live** and saves only if the test passes (the key is not echoed back; leave blank = keep unchanged)

## Add Models

Once the platform is connected:

1. Click "**Fetch Model List**": automatically pulls all available models for the platform (recommended first step)
2. For models missing from the list, use "**Add Model**" to add them manually: just enter the model name from the provider's website (the prefix is auto-completed, e.g. `glm-5.2` → `zai/glm-5.2`); "Test and Save" sends a minimal request to verify availability (incurs a tiny fee)
3. Capability tags on a model row (chat / vision / tts / asr / embed) toggle on click; just label them according to actual capabilities

## Two Ways to Provide the API Key

In descending priority:

1. **Enter it directly in the page** (recommended): "Save Connection" validates and saves, effective immediately
2. **Custom environment variable**: the "Key environment variable" field shows the platform's conventional variable name (e.g. `ZAI_API_KEY`); write `ZAI_API_KEY=sk-xxx` in the `.env` file of the data directory and it takes effect immediately after saving (click "Go to Settings" to open the `.env` editor directly; a system environment variable also works, but requires a service restart)


## Enable AI Parsing

After connecting a platform and adding a chat model, you must also designate the model used for parsing before AI parsing runs:

1. Go to Settings → Source Behavior and pick the platform and model in the **"Parse Model"** selector at the top of the page
2. Once selected, the new model takes effect from the next parse cycle

The prompt templates used by parsing are managed in Settings → Prompts and can be bound per source.

## Local Inference Engines

Four local platforms are built in, connecting to inference services self-deployed on your machine or LAN, no API key needed (change the API URL to point at your service as needed):
Ollama (default `http://127.0.0.1:11434`) · vLLM · llama.cpp · LM Studio

## Related

- Parsing parameters and model selection → [Feed Source Configuration · Source Behavior](rss.md)
- The knowledge base's local retrieval model (separate from the LLM platforms on this page) → [Knowledge Base Configuration](knowledge.md)
