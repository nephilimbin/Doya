# Knowledge Base Configuration Guide

> [中文](../../modules/knowledge.md) · English

> The knowledge base turns your collected articles into a searchable "intelligence library": it supports keyword search and hybrid semantic retrieval, plus Q&A-style queries over the stored content. The Settings → Knowledge Base page has four sections: **Retrieval Engine / Configure Models / Configure Knowledge Base / Test Retrieval**.

## Retrieval Engine

The top of the page shows an engine status badge (Running / Starting / Not Running / Crash Backoff). Use the "**Restart Engine**" button when the engine misbehaves or after changing models; a "Connected" badge means everything is normal.

> Without local models in place, the engine degrades to **keyword search only** (BM25), and the knowledge base stays usable — models are an optional enhancement that enables semantic retrieval (hybrid mode).

## Configuring Local Models (Enables Hybrid Retrieval, Optional)

Hybrid retrieval depends on **three local models** (embedding / query expansion / reranking), each consisting of a model file plus a companion checksum file — the model directory should end up with **6 files**. Two ways to obtain them:

### Option 1: The Author's Model Archive (Recommended, Extract and Use)

1. Download the model archive: <placeholder: archive download link / acquisition channel>
2. Click "**Open Model Directory**" (the directory is `~/.doya/models/rag/qmd/models/`; when accessing remotely from a phone, do this on a computer)
3. Extract all 6 files from the archive into that directory
4. Back on the page, click "**Detect Models**"; done once the badge shows "Ready"

### Option 2: Download from HuggingFace Yourself

The engine enforces strict file naming: after downloading, **rename** each file to the "Place As" name below and add the companion checksum file:

| Purpose | HuggingFace Repo / File | Place As | Companion Checksum File |
| ---- | ---- | ---- | ---- |
| Embedding | `Qwen3-Embedding-0.6B-Q8_0.gguf` from `Qwen/Qwen3-Embedding-0.6B-GGUF` | `hf_Qwen_Qwen3-Embedding-0.6B-Q8_0.gguf` | `Qwen3-Embedding-0.6B-Q8_0.gguf.etag` |
| Query expansion | `qmd-query-expansion-1.7B-q4_k_m.gguf` from `tobil/qmd-query-expansion-1.7B-gguf` | `hf_tobil_qmd-query-expansion-1.7B-q4_k_m.gguf` | `qmd-query-expansion-1.7B-q4_k_m.gguf.etag` |
| Reranking | `qwen3-reranker-0.6b-q8_0.gguf` from `ggml-org/Qwen3-Reranker-0.6B-Q8_0-GGUF` | `hf_ggml-org_qwen3-reranker-0.6b-q8_0.gguf` | `qwen3-reranker-0.6b-q8_0.gguf.etag` |

"Detect Models" checks every file for existence and integrity; readiness requires **all 6 files**. A missing companion checksum file blocks readiness, and later syncs may trigger the engine to re-download. The process is tedious — Option 1 is preferred.

Once the files are placed and detected, **no engine restart is needed** — the next retrieval automatically enables hybrid mode; the first retrieval takes roughly 20–30 seconds to load the models, after which everything is normal.

## Configuring Knowledge Bases

The knowledge base is organized into individual bases; each base can scope a different set of sources:

1. Click "**New Base**" and fill in:

| Field | Description |
| ---- | ---- |
| Base name | Must start with a lowercase letter or digit; only lowercase letters, digits, underscores, and hyphens; ≤64; must be unique |
| Index content | **Single-selection build** (recommended: for each article pick the best among AI summary → extracted body → original text; balances size and time) / **Composite build** (concatenates everything; most complete but heaviest in size and embedding time) |
| Scope | "All sources (dynamic)" = follows the source list automatically; or "Custom selection" to specify categories/sources |

2. Click "**Start Sync**" to build the index (with nothing selected, all enabled bases sync; "**Stop**" is available mid-run)
3. The "Enabled" column controls whether a base participates in retrieval; unwanted bases can be deleted (indexes and vectors are cleaned up together; rebuilding requires re-embedding)

**Sync interval** (minutes): defaults to 720 (12 hours) for one automatic incremental sync; set it to 0 to disable the schedule and go fully manual.

## Test Retrieval

At the bottom of the page you can verify instantly: choose the retrieval scope (all bases / specific bases), the retrieval method (auto / keyword / hybrid), and the number of results; enter query terms (supports `"phrase"` exact match and `-term` exclusion) and click "Retrieve"; results open up to the original text with relevance scores.

> For day-to-day intelligence Q&A, using the agent is recommended: on the "Agent" page, say "Search the knowledge base for information about XX". See the [Agent Guide](agent.md).

## Related

- Source management → [Feed Source Configuration](rss.md)
- Model (LLM for AI parsing) configuration → [Model Management](llm.md)
