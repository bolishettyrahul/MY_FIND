# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What This Repository Is

A knowledge base and reference collection for building **JARVIS** — a proactive developer intelligence layer powered by Claude (Anthropic API). The repo contains prompt engineering guides, tool schemas, system prompt templates, and model routing patterns. It is documentation and reference code, not a runnable application.

## AI Lead Context
- System prompt lives in: prompts/system_prompt.py
- jarvis.json lives in: jarvis.json (repo root)
- Regression tests: run python test_prompt.py
- All tool schemas are final — do not modify descriptions without running regression tests
- AI_MODE=local in .env — never switch to cloud during development
- Prompt version must be incremented and tests must pass before any prompt edit is committed

## Repository Structure

- **`my_find/`** — Core knowledge base (Markdown files covering JARVIS architecture):
  - `jarvis_mem.md` — `jarvis.json` project memory schema (single source of truth for project context)
  - `prompt_struc.md` — System prompt 5-section structure and v1 template with cache strategy
  - `prompt_fund.md` — Prompt engineering fundamentals: specificity, XML tags, conditional rules, hallucination prevention, output format control, regression testing
  - `prompts.md` — The 4 JARVIS-specific prompts: gate (Ollama), surface (Haiku), error diagnosis, research report
  - `tool.md` — Tool use conversation loop pattern with the Anthropic API
  - `tool_schema.md` — All 6 JARVIS tool schemas with disambiguation rules
  - `model.md` — Model routing logic (Sonnet/Haiku/Ollama) and the Ollama JSON parsing problem
  - `prompt_caching.md` — Anthropic prompt caching setup and cost breakdown
  - `bonus.md` — Context window management, streaming, temperature/sampling, 10 common failure modes

- **`pdf/`** — External reference PDFs on AI-assisted development

## Key Architectural Concepts

**JARVIS uses a 3-tier model routing strategy:**
- **Claude Sonnet** — Complex reasoning tasks (error diagnosis, research reports, multi-step analysis)
- **Claude Haiku** — Structured, repetitive tasks (git summaries, commit messages, session summaries)
- **Ollama/CodeLlama** — Free, always-on background tasks (proactive gate runs 50+/hour)

**System prompt is split into static (cached) and dynamic (fresh) blocks:**
- Static block (sections 4+5: identity rules + tool rules) → `cache_control: {"type": "ephemeral"}`
- Dynamic block (sections 1+2+3: project context + codebase map + recent sessions) → no caching, rebuilt from `jarvis.json` each call

**`jarvis.json` is the persistent project memory** — structured fields only, updated via explicit trigger phrases ("we decided", "going with", "lock this in"), never on tentative language.

**Tool descriptions control Claude's routing behavior** — the `description` field determines WHEN Claude calls a tool. Each schema includes positive triggers and negative exclusions to prevent overlap (especially `read_codebase` vs `read_git_history`).

## Working With This Repo

When editing or extending the knowledge base files:
- Keep the prompt/tool examples grounded in the JARVIS reference project (Intelligent Pest Risk Estimation System, EfficientNet-B3, Sentinel-2)
- Maintain the BAD/GOOD example pattern used throughout the docs
- System prompt total budget is 5,000 tokens — respect section token allocations in `prompt_struc.md`
- Regression tests (in `prompt_fund.md`) should pass before any prompt version is finalized
