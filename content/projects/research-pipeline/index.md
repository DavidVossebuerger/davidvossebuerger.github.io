---
title: "Research-Pipeline"
date: 2026-09-11
author: ["David Vossebürger"]
description: "Daily arXiv quant-finance paper scoring pipeline with a local LLM and Telegram delivery."
summary: "Daily arXiv quant-finance paper scoring pipeline with local LLM (Ollama / OpenAI- / Anthropic-compatible backends), SQLite state, two-stage scoring, and Telegram delivery."
editPost:
    URL: "https://github.com/DavidVossebuerger/Research-Pipeline"
    Text: "Source code"

---

---

##### Overview

**research-pipeline** v0.3.1 is a daily arXiv quant-finance paper scoring pipeline. It fetches new submissions across `q-fin.*`, `cs.LG`, `stat.ML` over a rolling 26-hour window, deduplicates them into a local SQLite state DB, scores abstracts with a local LLM using German-language prompts, re-scores survivors on the downloaded PDF, and ships the top picks to Telegram.

Two console scripts (`research-pipeline`, `research-pipeline-install`) install the package; an interactive wizard (`research-pipeline-install install`) wires up Ollama, validates the Telegram token, and schedules the cron entry.

---

##### Pipeline stages

| Stage | Description |
|---|---|
| **Fetch** | Roll up arXiv's `q-fin.*`, `cs.LG`, `stat.ML` over a 26-hour window via the `arxiv` Python client; 3× auto-retry on 5xx / 429. |
| **Dedup** | Insert into SQLite (`data/state.db` by default); unique on arXiv ID. |
| **Stage A — abstract score** | Score each abstract 0–10 with the chosen LLM using `src/research_pipeline/prompts/abstract_score.txt` (German); drop papers below threshold (default 7). |
| **Stage B — deep score** | Download PDF, extract text with `pypdf`, re-score survivors using `deep_score.txt` to produce a summary, `why` rationale, and tags. Stage B is skipped (warning only) on PDF failure. |
| **Notify** | Send top-K picks to Telegram with score + summary + arXiv link; topic-threads + end-of-day daily summary. |

---

##### Optional features (off by default)

- **Weekly digest** — ISO-week grouping + tag clustering.
- **`narrative_review`** — German-language narrative connecting the day's papers.
- **`autobuild`** — background code/explainer sessions.
- **Interactive Telegram bot** — `/ask`, `/last`, and similar commands.
- **`backfill` CLI** — re-score historical papers.

---

##### Technical stack

- **Language:** Python 3.10+ (CI matrix 3.11 + 3.12)
- **Build:** setuptools, `pyproject.toml`
- **Runtime deps:** `arxiv>=2.1.0`, `httpx>=0.27.0`, `pypdf>=5.0.0`, `python-dotenv>=1.0.0`, `click>=8.1.0`
- **LLM backends** (via `LLM_PROVIDER`): `ollama` (local, default), `openai_compat` (OpenAI / OpenRouter / Together / Groq), `anthropic_compat` (Anthropic-format APIs)
- **Storage:** SQLite with indexes on `published_at`, `abs_score`, `deep_score`, `picked`, `ai_events.ts`
- **Notify:** Telegram Bot API over `httpx`; fails open if `TELEGRAM_BOT_TOKEN` / `TELEGRAM_CHAT_ID` are unset
- **Dev tooling:** `ruff` (`line-length = 100`, `target-version = "py311"`), `pytest` + `pytest-cov`, `detect-secrets` in CI
- **Hardware:** Debian / Ubuntu Linux, 8 GB RAM minimum (16 GB recommended; Ollama model ≈ 4 GB)

---

##### Source layout

```
src/research_pipeline/
  cli.py, config.py, db.py, fetch_arxiv.py, pdf_extract.py, score.py, notify.py,
  autobuild.py, code_session.py, backfill.py, daily_summary.py,
  weekly_digest.py, narrative_review.py, logging_setup.py
  runtime/runner.py                # orchestrator (fetch → score → notify)
  install/wizard.py                # install wizard
  install/ollama.py, telegram.py, cron.py, doctor.py
  prompts/abstract_score.txt       # German-language scoring prompt
  prompts/deep_score.txt
tests/                             # pytest suite (-v --tb=short)
docs/INSTALL.md, docs/ARCHITECTURE.md (module map + ER diagram), docs/PROMPTS.md,
docs/superpowers/
.github/workflows/                 # CI on PRs
.github/ISSUE_TEMPLATE/, PULL_REQUEST_TEMPLATE.md
pyproject.toml, .env.example, .secrets.baseline (detect-secrets),
README.md, LICENSE (MIT)
```

---

##### Quick start

```bash
pip install -e .
research-pipeline-install install   # interactive wizard: Ollama + Telegram + cron
research-pipeline run
```

Defaults: 26-hour arXiv lookback, Stage-A threshold ≥ 7, single-process file-backed run, pipeline exits 0 if Telegram is unset.

---

##### External links

- Repo: <https://github.com/DavidVossebuerger/Research-Pipeline>
- Docs in tree: `docs/INSTALL.md`, `docs/ARCHITECTURE.md`, `docs/PROMPTS.md`
- License: MIT
