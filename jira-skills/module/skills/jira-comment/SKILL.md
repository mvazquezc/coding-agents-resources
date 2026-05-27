---
name: jira-comment
description: Add a comment to a Jira issue using the Atlassian CLI (acli).
---

Add a comment to a Jira issue: $ARGUMENTS

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

1. Parse the arguments to extract the issue key (first argument) and the comment text (everything after)
2. Run `acli jira workitem comment create --key "$ISSUE_KEY" --body "$COMMENT_TEXT"`
3. Confirm the comment was added successfully
4. If the command fails with "unauthorized", suggest re-authenticating: `acli jira auth login --web` or `acli jira auth login --site "yoursite.atlassian.net" --email "you@example.com" --token`
