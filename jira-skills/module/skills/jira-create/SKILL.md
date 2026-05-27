---
name: jira-create
description: Create a new Jira issue with optional type, priority, description, assignee, and labels using the Atlassian CLI (acli).
---

Create a new Jira issue: $ARGUMENTS

## Instructions

0. Verify that `acli` is installed and authenticated before proceeding. Run the following checks:
   ```bash
   command -v acli > /dev/null 2>&1
   ```
   If `acli` is not found, stop and tell the user: "acli (Atlassian CLI) is not installed. Install it from https://developer.atlassian.com/cloud/acli/guides/install-acli/"
   Then run:
   ```bash
   acli jira project list --limit 1 > /dev/null 2>&1
   ```
   If authentication fails, stop and tell the user: "Not authenticated to Jira or token is invalid. Re-authenticate with: `acli jira auth login --web` or `acli jira auth login --site \"yoursite.atlassian.net\" --email \"you@example.com\" --token`"

1. Parse the arguments to extract the project key (first argument) and the summary (everything after)
2. If no arguments are provided, ask the user for the project key and summary at minimum
3. Run `acli jira workitem create --project "$PROJECT_KEY" --type "Task" --summary "$SUMMARY"`
4. If the user specifies additional fields in their request (e.g., type, priority, description, assignee, labels), include them:
   - Type: `--type "$TYPE"` (default: Task)
   - Priority: `--priority "$PRIORITY"`
   - Description: `--description "$DESCRIPTION"`
   - Assignee: `--assignee "$ASSIGNEE"`
   - Labels: `--labels "$LABELS"`
5. Confirm the issue was created by showing the new issue key and URL
6. If the command fails with "unauthorized", suggest re-authenticating: `acli jira auth login --web` or `acli jira auth login --site "yoursite.atlassian.net" --email "you@example.com" --token`
