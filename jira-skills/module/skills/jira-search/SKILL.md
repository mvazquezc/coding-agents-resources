---
name: jira-search
description: Search Jira issues by text, JQL, or filters using the Atlassian CLI (acli).
---

Search Jira for issues matching: "$ARGUMENTS"

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

1. If `$ARGUMENTS` looks like a JQL query (contains operators like `=`, `AND`, `OR`, `IN`, `~`, `>=`, `<=`), pass it directly: `acli jira workitem search --jql "$ARGUMENTS" --limit 20`
2. Otherwise, search by text: `acli jira workitem search --jql "summary ~ \"$ARGUMENTS\" OR description ~ \"$ARGUMENTS\" ORDER BY updated DESC" --limit 20`
3. Present results in a table with columns: Key, Status, Type, Priority, Summary
4. If 20 results are returned, the results are likely truncated. Suggest the user refine their query or use JQL for more precise filtering
5. If no results found, suggest alternative search terms or broader filters
6. If the command fails with "unauthorized", suggest re-authenticating: `acli jira auth login --web` or `acli jira auth login --site "yoursite.atlassian.net" --email "you@example.com" --token`
