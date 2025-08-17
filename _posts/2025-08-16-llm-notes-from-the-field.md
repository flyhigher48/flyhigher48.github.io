---
title: "LLM Notes from the Field: A Working Brain Dump"
description: "Ongoing experiments with local vLLM, MCP integrations, agents, prompts, and small automations for engineering work."
date: 2025-08-16
categories: [llm-ml]
tags: [llm, vllm, engineering, productivity, prompts, workflows, mcp, agents]
published: false
---

This is a **brain dump**, not a verdict. I’ll update it as I try new ideas.

## Context
- Local GPU server running **vLLM** to host models
- Use cases: summarizing experiment notes, drafting validation sections, generating run README stubs, suggesting naming conventions
- Constraints: privacy (no cloud for sensitive data), mixed-quality inputs (emails, PDFs, CSVs)

## What I’m poking at
- **MCP (Model Context Protocol)** to wire tools and context in cleanly
- **Agents** for repeatable tasks (summaries, checklists, comparisons)
- Small **RAG** over current project notes and `DECISIONS.md`
- Prompt patterns for analysis and validation narratives

## Early notes
- Short, structured prompts beat long, narrative ones
- Linking to concrete folders/files reduces hallucination
- One model for drafting + one human pass is fast enough most days

## Didn’t love
- Mega “do everything” prompts
- Asking for numbers without giving the source CSV/table
- Treating big email threads as one blob (better to pre-extract key messages)

## Next
- Build a tiny prompt library for run hygiene and validation writeups
- Try DVC + LLM to auto‑generate run manifests
- Benchmark cost/latency vs. usefulness for routine tasks

*If you try any of this, feel free to open an issue with your notes. I’ll learn from them and give credit.*
