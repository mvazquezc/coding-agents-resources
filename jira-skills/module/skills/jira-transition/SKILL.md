---
name: jira-transition
description: Transition a Jira issue to a new status using the Atlassian CLI (acli).
---

Transition Jira issue: $ARGUMENTS

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

1. Parse the arguments to extract the issue key (first argument) and the target status (everything after)
2. If no target status is provided, run `acli jira workitem transition list --key "$ISSUE_KEY"` to show available transitions and ask the user which one to use
3. Run `acli jira workitem transition --key "$ISSUE_KEY" --transition "$TARGET_STATUS"`
4. Confirm the transition was successful by showing the issue key and new status
5. If the transition name is invalid, run `acli jira workitem transition list --key "$ISSUE_KEY"` to show available transitions and ask the user to pick one
6. If the command fails with "unauthorized", suggest re-authenticating: `acli jira auth login --web` or `acli jira auth login --site "yoursite.atlassian.net" --email "you@example.com" --token`
