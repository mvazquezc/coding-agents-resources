---
description: Code reviewer powered by Claude Sonnet. Invoked by the code-reviewer orchestrator.
mode: subagent
model: google-vertex/claude-sonnet-4.6
temperature: 0.1
hidden: true
color: "#D97706"
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
