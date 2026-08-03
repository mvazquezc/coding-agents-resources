# Coding Agents Resources

A collection of AI agent skills, commands, and agents packaged as [lola](https://github.com/LobsterTrap/lola) modules. Install them into any supported AI assistant (OpenCode, Claude Code, Cursor, Gemini CLI).

## Modules

### jira-skills

Manage Jira issues via the [Atlassian CLI](https://developer.atlassian.com/cloud/acli/) (`acli`).

| Type | Contents |
|------|----------|
| Skills | `jira-get`, `jira-search`, `jira-mine`, `jira-create`, `jira-transition`, `jira-assign`, `jira-comment`, `jira-comments`, `jira-link` |
| Commands | `/jira-get`, `/jira-search`, `/jira-mine`, `/jira-create`, `/jira-transition`, `/jira-assign`, `/jira-comment`, `/jira-comments`, `/jira-link` |

**Requires:** `acli` installed and authenticated (`acli jira auth login --web`)

### gws-skills

Interact with Google Workspace services via the [`gws`](https://github.com/googleworkspace/cli) CLI.

| Type | Contents |
|------|----------|
| Skills | `gws-shared`, `gws-calendar`, `gws-calendar-agenda`, `gws-docs`, `gws-docs-write`, `gws-gmail`, `gws-gmail-read`, `gws-gmail-triage`, `gws-gmail-watch` |

**Requires:** `gws` CLI installed and authenticated (`gws auth login`)

### custom-agents

Custom agents for code review and convention-aware development.

| Type | Contents |
|------|----------|
| Agents | `code-reviewer` (orchestrator), `claude-sonnet-reviewer`, `chatgpt-reviewer`, `gemini-reviewer` (sub-agents), `build-following-repo-conventions` |
| Commands | `/review` |

The `code-reviewer` agent dispatches 1-3 reviewer sub-agents (Claude Sonnet, ChatGPT, Gemini) in parallel and consolidates their findings. The `build-following-repo-conventions` agent explores repo patterns before writing code.

### coding-skills

Methodology skills for writing better code.

| Type | Contents |
|------|----------|
| Skills | `as-karpathy`, `code-simplifier`, `interactive-architecture-diagram` |
| Commands | `/as-karpathy`, `/code-simplifier` |

### other-skills

Additional utility skills.

| Type | Contents |
|------|----------|
| Skills | `write-as-mario` |
| Commands | `/write-as-mario` |

Enables Mario Vazquez's technical writing style based on [linuxera.org](https://linuxera.org/).

### ocp-operator-troubleshooting

Troubleshoot OLM-managed operators on OpenShift 4.12+.

| Type | Contents |
|------|----------|
| Skills | `ocp-operator-troubleshoot` |

Three-phase diagnostic: OLM triage, workload triage, and deep source code analysis. Supports live cluster (`oc`) and must-gather (`omc`) modes.

**Requires:** `oc` and/or `omc` installed

## Installation

### Prerequisites

Install [lola](https://github.com/LobsterTrap/lola):

### Option 1: Install via marketplace (recommended)

Register the marketplace once, then search and install modules by name:

```bash
# Register the marketplace (one-time)
lola market add mario https://raw.githubusercontent.com/mvazquezc/coding-agents-resources/main/marketplace.yml

# Search for available modules
lola mod search jira
lola mod search openshift

# Install a module
lola install jira-skills -a opencode --scope user
```

### Option 2: Install directly from Git

If you prefer not to register a marketplace, add modules directly:

```bash
lola mod add https://github.com/mvazquezc/coding-agents-resources.git#subdirectory=jira-skills
lola install jira-skills -a opencode --scope user
```

Replace `jira-skills` with any module name: `gws-skills`, `custom-agents`, `coding-skills`, `other-skills`, or `ocp-operator-troubleshooting`.

### Option 3: Install from a local clone

```bash
git clone https://github.com/mvazquezc/coding-agents-resources.git
cd coding-agents-resources

lola mod add ./jira-skills
lola install jira-skills -a opencode --scope user
```

### Project vs user scope

By default, `lola install` installs at **project scope** -- skills, commands, and agents are added to the current project directory only.

To install at **user scope** (available globally across all your projects), use `--scope user`:

```bash
# Install globally for the current user
lola install jira-skills -a opencode --scope user
```

| Scope | Flag | Install location (OpenCode) | Install location (Claude Code) |
|-------|------|-----------------------------|-------------------------------|
| Project | `--scope project` (default) | `.opencode/`, `AGENTS.md` in the project | `.claude/` in the project |
| User | `--scope user` | `~/.config/opencode/` | `~/.claude/` |

### Update modules

```bash
# Update a specific module from source
lola mod update jira-skills

# Regenerate installed assistant files
lola update
```

### Uninstall

```bash
lola uninstall jira-skills
lola mod rm jira-skills
```

## Validation

Skills in this repo are validated with [skillsaw](https://github.com/stbenjam/skillsaw):

```bash
# Lint a specific module
skillsaw lint --strict jira-skills/

# Lint all modules
for mod in jira-skills gws-skills custom-agents coding-skills other-skills ocp-operator-troubleshooting; do
  echo "--- $mod ---"
  skillsaw lint --strict "$mod/"
done
```
