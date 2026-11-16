# TriGuard — Validity-Aware LLM Semantic Cache

University of Michigan CSE 584 team project (three contributors). Research prototype for lowering repeated LLM requests through multi-stage caching, query normalization, dynamic time-to-live classification, and response verification.

## Architecture

FastAPI `POST /ask` → normalization → TTL/volatility gate → Redis exact match / ChromaDB semantic similarity → optional verification → LLM fallback → conditional caching. The implementation uses Python, Redis, ChromaDB, sentence embeddings, local Ollama inference, and configurable external LLM APIs.

## Stored evaluation (April 19, 2026)

The supplied evaluation summary covers **200 queries**: **65 cache hits (32.5%)**, **50 volatile bypasses**, and **85 other LLM responses**. Average cache-hit latency in that run was **87.6 ms**; average LLM response latency **9,271.95 ms**. Results are workload-specific, not production benchmarks.

**Metric caveat:** The saved summary's claimed **57.5% "API reduction"** excludes volatile-bypass calls from its denominator. When counting all upstream requests, 65/200 queries were served from cache, for an observed **32.5% avoided-call share**. Do not mix these results with measurements quoted in the accompanying course report. The saved run did not record successful `CACHE_HIT_VERIFIED` events.

## Repository structure

- `server.py`: FastAPI service
- `modules/trigaurd.py`: core orchestration (legacy spelling kept to preserve imports)
- `modules/dynamic_ttl_training.py`: TTL classifier workflow
- `modules/gate1.py`, `modules/gate3.py`, `modules/semantic_cache.py`: caching and verification components
- `prototypes/trigard_db_layer.py`: separate Redis/Chroma prototype
- `evaluate.py`: evaluation harness
- `data/`: recorded results; `triaguard_*_set.csv`: evaluation query sets

## Reproduce locally

Python 3.10+, Redis, and access to the configured LLM services are needed. A CUDA GPU is recommended for model inference. Ollama is required for the Gate 3 model. Install dependencies from `requirements.txt`, then configure your own API credentials using environment variables (never commit secrets).

```bash
python -m venv .venv
source .venv/bin/activate   # Windows PowerShell: .venv\Scripts\Activate.ps1
pip install -r requirements.txt
python evaluate.py --help
```

The full evaluation depends on external services and model downloads and has **not** been independently rerun for this portfolio copy. Serialized `joblib` objects must only be loaded from trusted sources. Course collaboration and individual contribution details should be confirmed before describing authorship in a resume.
