# Custom Agents

This module provides custom agents for code review and convention-aware development.

## Agents

### code-reviewer

Multi-model code review orchestrator. Spawns 1-3 reviewer sub-agents (Claude Sonnet, ChatGPT, Gemini) in parallel and consolidates their findings into a unified report.

Use the `/review` command to start a code review session.

**Sub-agents:**
- **claude-sonnet-reviewer** -- Claude Sonnet 4.6 via Vertex
- **chatgpt-reviewer** -- GPT 5.3 Codex via OpenAI
- **gemini-reviewer** -- Gemini 2.5 Pro via Vertex

### build-following-repo-conventions

Build agent that explores repository conventions before writing code. Follows a strict workflow: reason about the task, learn the repo patterns, search for reusable code, implement following conventions, and verify against success criteria.

## Available Commands

- `/review` -- Start a multi-model code review
