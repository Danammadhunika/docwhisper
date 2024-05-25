# docwhisper

**Local semantic search over your own documents. Index a folder of PDFs / Markdown / text, search it in plain English, ask it questions, no cloud, no server, no API key.**

Point it at a folder, and docwhisper builds a portable embedding index next to your files (`.docwhisper/`). Search returns the most semantically relevant passages; `ask` feeds them to a local LLM for a cited answer. Everything runs on your machine via [fastembed](https://github.com/qdrant/fastembed) (ONNX, CPU) and [Ollama](https://ollama.com).

```bash
docwhisper index ~/notes         # embed changed files (incremental)
docwhisper search ~/notes "how does our retry backoff work"
docwhisper ask ~/notes "what did we decide about rate limits?"
```

Real output over a folder of four mixed Markdown docs on this machine:

```
$ docwhisper index /tmp/dw_demo
indexed 4 files (+21 chunks, -0 files removed), total 21 chunks

$ docwhisper search /tmp/dw_demo "how does the scope guardrail refuse out-of-domain questions"
0.719  specsage-design.md   If the best fused-and-reranked retrieval score falls below a threshold …
0.667  specsage-readme.md    # specsage  Agentic RAG over the IETF RFC archive — with grounded citations …
0.659  specsage-readme.md    uv run specsage ask "Why must WebSocket clients mask frames sent to the server?" …
```

The query shares almost no keywords with the top hit ("scope guardrail refuse out-of-domain" vs. "retrieval score falls below a threshold ... refuses") - that's the embeddings working, not grep.

## What makes it more than a toy

- **Incremental indexing.** Re-indexing only re-embeds files whose mtime changed. Delete a file and its chunks are evicted; edit a file down to nothing and its stale chunks are evicted too (that second case was a real bug caught in review - see below).
- **Real formats.** PDF (via pypdf), Markdown, `.txt`, `.rst`. Section-agnostic chunking with paragraph-aware packing and overlap.
- **Portable index.** Plain `chunks.jsonl` + `embeddings.npy` + `manifest.json` under `.docwhisper/`. Copy the folder, keep the index.
- **Grounded `ask`.** Answers cite the source files inline as `[n]`; if the passages don't contain the answer, the model is told to say so.
- **Fast where it counts.** The embedding model loads lazily, so `search`/`stats` on an existing index start instantly.

## Install & use

Prereqs: [uv](https://docs.astral.sh/uv/); [Ollama](https://ollama.com) + `ollama pull llama3.1:8b` only if you use `ask`.

```bash
git clone https://github.com/your-account/docwhisper && cd docwhisper
uv sync
uv run docwhisper index ./docs
uv run docwhisper search ./docs "your question in plain english"
uv run docwhisper ask ./docs "your question" --model llama3.1:8b
```

Commands: `index <folder>`, `search <folder> <query> [-k N] [--paths-only]`, `ask <folder> <question> [-k N] [--model M]`, `stats <folder>`. Exit codes: `2` if no index, `3` if Ollama is unreachable.

## Development

```bash
uv run pytest -q      # 42 tests: chunking, incremental ingest, cosine search, CLI, + regressions
uv run ruff check src tests && uv run mypy src
```

CI runs lint, types, and the full suite on every push. Every test runs offline, a deterministic hash-based fake embedder stands in for the model, so no network or model download is needed.

## What I'd build next

- `docwhisper watch <folder>` - re-index on file change.
- Snippet highlighting of the matched span within each hit.
- Optional cross-encoder rerank of the top-N (the same lever that helped specsage), behind a flag.

## Maintainer

Maintained by **Madhunika Danam**, AI/ML Engineer, with a focus on semantic search, RAG systems, and LLM-powered applications.

Contact: [madhunika.danam@belvionus.com](mailto:madhunika.danam@belvionus.com).