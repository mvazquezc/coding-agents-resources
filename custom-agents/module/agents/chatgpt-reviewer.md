---
description: Code reviewer powered by ChatGPT. Invoked by the code-reviewer orchestrator.
mode: subagent
model: openai/gpt-5.3-codex
temperature: 0.1
hidden: true
color: "#10A37F"
permission:
  edit: deny
  bash:
    "*": deny
    "git diff*": allow
    "git log*": allow
    "git show*": allow
    "git status*": allow
---
{file:../prompts/code-review.md}
