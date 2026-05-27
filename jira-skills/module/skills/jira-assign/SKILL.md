---
name: jira-assign
description: Assign a Jira issue to a user, yourself, or unassign it using the Atlassian CLI (acli).
---

Assign Jira issue: $ARGUMENTS

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

1. Parse the arguments to extract the issue key and the assignee
2. If the assignee is "me" or omitted, run `acli jira workitem assign --key "$ISSUE_KEY" --assignee "@me"`
3. If the assignee is "unassign" or "x", run `acli jira workitem assign --key "$ISSUE_KEY" --remove-assignee`
4. Otherwise, run `acli jira workitem assign --key "$ISSUE_KEY" --assignee "$ASSIGNEE_EMAIL"`
5. Confirm the assignment was successful by showing the issue key and new assignee
6. If the command fails with "unauthorized", suggest re-authenticating: `acli jira auth login --web` or `acli jira auth login --site "yoursite.atlassian.net" --email "you@example.com" --token`
