# Google Workspace Skills

This module provides skills for interacting with Google Workspace services via the `gws` CLI.

## Prerequisites

- `gws` CLI must be installed and on `$PATH`
- Must be authenticated: `gws auth login`

## Available Skills

### Shared

- **gws-shared** -- Authentication, global flags, output formatting, and security rules

### Calendar

- **gws-calendar** -- Manage calendars and events
- **gws-calendar-agenda** -- Show upcoming events across all calendars

### Docs

- **gws-docs** -- Read and write Google Docs
- **gws-docs-write** -- Append text to a document

### Gmail

- **gws-gmail** -- Send, read, and manage email
- **gws-gmail-read** -- Read a message and extract its body or headers
- **gws-gmail-triage** -- Show unread inbox summary
- **gws-gmail-watch** -- Watch for new emails and stream them as NDJSON
