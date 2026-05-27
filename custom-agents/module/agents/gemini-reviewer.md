---
description: Code reviewer powered by Gemini. Invoked by the code-reviewer orchestrator.
mode: subagent
model: google-vertex/gemini-2.5-pro
temperature: 0.1
hidden: true
color: "#4285F4"
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
