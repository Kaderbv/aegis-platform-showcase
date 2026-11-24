# Results Summary

Results from the project's recorded experiments. Each experiment in the private repository has a written
report with its hypothesis, setup, raw output, and limitations. This page reproduces the result tables.

**Caveat:** these are small, sequential runs (3–10 iterations) on a single developer laptop. Most of them
call the live NVIDIA API, so network latency dominates and varies between runs. Read them as directional
evidence, not as capacity benchmarks.

---

## Startup warm-up of the retrieval models

Request: `POST /rag/search?rerank=true`, `top_k=2`. Models already on disk: spaCy `en_core_web_lg` (Presidio)
and `cross-encoder/ms-marco-MiniLM-L-6-v2`. Embeddings: NVIDIA `nemotron-3-embed-1b` (2048 dimensions).
3 runs per setting, fresh process each time. Times in ms, measured from process start: median (min–max).

| `WARMUP_ON_STARTUP` | start → live | start → ready | 1st search | 2nd search |
|---|---|---|---|---|
| `false` | 1,332 (1,319–1,337) | 1,333 (1,320–1,338) | **7,407** (7,128–8,034) | 587 (490–599) |
| `true` | 1,499 (1,407–1,638) | 8,385 (8,034–8,412) | **282** (251–432) | 427 (268–430) |

- About 7 s of model loading moves from the first user request into the startup window.
- Liveness is unaffected because warm-up runs in a worker thread.
- Cost: readiness arrives ~7 s later, and every replica holds both models in memory.

## Grounded agent answers (RAG node)

Corpus: 3 short documents (two INTERNAL for one organization, one PUBLIC). Caller: analyst role, same
organization. 5 sequential iterations per measurement.

| Measurement | n | median ms | min ms | max ms | p95 ms |
|---|---|---|---|---|---|
| Knowledge-base answer, time to first token | 5 | 757 | 537 | 1,219 | 1,219 |
| Knowledge-base answer, total | 5 | 881 | 667 | 1,413 | 1,413 |
| Plain answer, time to first token | 5 | 274 | 258 | 286 | 286 |
| Plain answer, total | 5 | 579 | 550 | 598 | 598 |
| Retrieval duration | 6 | 442 | 262 | 6,092 | 6,092 |
| `/health` latency during a reranked `/rag/search` | 57 | 4 | 1 | 94 | 23 |

- Grounding adds ~483 ms to the first token, close to the median retrieval time.
- The 6,092 ms outlier is the cold first request. It was later fixed by the startup warm-up above.
- `/health` stayed responsive while a 3.3 s reranked search ran, because the models run off the event loop.
- Manual quality check: the correct source was cited, PII was masked in both prompt and answer, an anonymous
  caller couldn't read INTERNAL documents, and out-of-corpus questions returned no citations.

## Streaming reliability

Provider: NVIDIA, model `openai/gpt-oss-20b`.

| Scenario | Iterations | Failures | Median | P95 |
|---|---|---|---|---|
| Calculator (no LLM) | 10 | 0 | 1.3 ms | 4.3 ms |
| Fake LLM provider | 10 | 0 | 1.2 ms | 1.5 ms |

| Live run | Time to first chunk | Total | Chunks |
|---|---|---|---|
| Generate (non-streamed) | n/a | 5,693 ms | n/a |
| Stream | **839 ms** | 1,937 ms | 91 |

## Provider retry and failure behavior

Scripted provider, no network, 10 iterations per scenario.

| Scenario | Attempts (expected) | Result |
|---|---|---|
| Generate retries a transient timeout | 20 (20); mean recovery 15 ms | ✅ PASS |
| Generate stops at `max_attempts` when failures persist | 30 (30) | ✅ PASS |
| Stream retries a timeout before the first chunk | 20 (20); mean recovery 16 ms | ✅ PASS |
| Stream does not retry after delivering partial output | 10 (10) | ✅ PASS |

## Test suite

| Suite | Scope |
|---|---|
| Unit | Agent graph and streaming, grounding, LLM providers and retries, observability, chunking, embeddings, access control, PII, reranking, citations, hallucination scoring, document loading, persistence settings |
| Integration | API and error mapping, app factory and warm-up, agent + RAG over HTTP, full safety pipeline, data integrity (tenant isolation, delete, change detection), file upload, database readiness |
| Smoke | End-to-end API, graph, and tool checks against a stub LLM |
| **Total collected** | **924 tests**, no API keys required |

CI (GitHub Actions) runs Ruff lint and format checks, mypy, pytest (with a PostgreSQL 16 service for tests
marked `postgres`), and pre-commit on every push.
