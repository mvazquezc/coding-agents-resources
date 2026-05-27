---
name: jira-link
description: Link two Jira issues together with a specified relationship type using the Atlassian CLI (acli).
---

Link Jira issues: $ARGUMENTS

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

1. Parse the arguments to extract the source issue key, link type, and target issue key
2. Common link types: "blocks", "is blocked by", "relates to", "duplicates", "is duplicated by", "clones", "is cloned by"
3. If no link type is provided or only two issue keys are given, default to "relates to"
4. Run `acli jira workitem link create --inward-key "$SOURCE_KEY" --outward-key "$TARGET_KEY" --link-type "$LINK_TYPE"`
5. Confirm the link was created by showing both issue keys and the link type
6. If the link type is invalid, run `acli jira workitem link type list` to show available link types and ask the user to pick one
7. If the command fails with "unauthorized", suggest re-authenticating: `acli jira auth login --web` or `acli jira auth login --site "yoursite.atlassian.net" --email "you@example.com" --token`
