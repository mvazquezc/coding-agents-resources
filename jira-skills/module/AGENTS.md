# Jira Skills

This module provides 9 skills for managing Jira issues via the Atlassian CLI (`acli`).

## Prerequisites

- `acli` (Atlassian CLI) must be installed and on `$PATH`
- Must be authenticated to a Jira instance: `acli jira auth login --web`

## Available Skills

- **jira-get** -- View issue details and recent comments
- **jira-search** -- Search issues by text or JQL
- **jira-mine** -- List your open issues
- **jira-create** -- Create a new issue
- **jira-transition** -- Transition an issue to a new status
- **jira-assign** -- Assign or unassign an issue
- **jira-comment** -- Add a comment to an issue
- **jira-comments** -- List comments on an issue
- **jira-link** -- Link two issues together

## Available Commands

Each skill has a matching slash command: `/jira-get`, `/jira-search`, `/jira-mine`, `/jira-create`, `/jira-transition`, `/jira-assign`, `/jira-comment`, `/jira-comments`, `/jira-link`.
