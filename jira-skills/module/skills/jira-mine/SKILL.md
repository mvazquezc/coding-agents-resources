---
name: jira-mine
description: List your open Jira issues, optionally filtered by status, using the Atlassian CLI (acli).
---

List my Jira issues: $ARGUMENTS

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

1. If no arguments provided, run `acli jira workitem search --jql "assignee = currentUser() AND status NOT IN (Done, Closed) ORDER BY updated DESC" --limit 20`
2. If a status is provided (e.g., "In Progress", "To Do"), run `acli jira workitem search --jql "assignee = currentUser() AND status = \"$STATUS\" ORDER BY updated DESC" --limit 20`
3. Present results in a table with columns: Key, Status, Type, Priority, Summary
4. If no issues found, say so
5. If the command fails with "unauthorized", suggest re-authenticating: `acli jira auth login --web` or `acli jira auth login --site "yoursite.atlassian.net" --email "you@example.com" --token`
