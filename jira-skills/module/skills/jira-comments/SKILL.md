---
name: jira-comments
description: List comments on a Jira issue using the Atlassian CLI (acli).
---

List comments for Jira issue: $ARGUMENTS

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

1. If the issue key looks invalid (not in PROJ-123 format), ask the user to provide a valid issue key
2. Run `acli jira workitem comment list --key "$ARGUMENTS" --limit 10`
3. Present the comments showing: Author, Created date, and Body for each comment
4. If no comments found, say so
5. If the command fails with "unauthorized", suggest re-authenticating: `acli jira auth login --web` or `acli jira auth login --site "yoursite.atlassian.net" --email "you@example.com" --token`
