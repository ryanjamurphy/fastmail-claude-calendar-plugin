# fastmail-calendar

A Claude Cowork plugin that integrates with Fastmail's calendar via JMAP. Read, create, update, and delete calendar events. Supports time-blocking workflows and season-based batch scheduling.

## Setup

### 1. Get a Fastmail API Token

1. Go to **Fastmail Settings → Privacy & Security → Manage API tokens**
2. Create a new token with **calendar read/write** scope
3. Copy the token

### 2. Build the Plugin File

From the root of this repository, install dependencies and package the plugin:

```bash
npm install
zip -r /tmp/fastmail-calendar.plugin . -x "node_modules/*" "*.DS_Store" ".git/*"
```

This creates `/tmp/fastmail-calendar.plugin` — a zip archive containing the server, skills, and plugin manifest.

### 3. Install in Claude Cowork

<!-- TODO: document the exact Cowork installation path once confirmed -->
Install `/tmp/fastmail-calendar.plugin` in Claude Cowork. When prompted, set the required environment variable:

```
FASTMAIL_API_TOKEN=your-token-here
```

Optionally set your timezone (defaults to `America/St_Johns`):

```
FASTMAIL_TIMEZONE=America/New_York
```

The MCP server starts automatically when Claude needs to use a calendar tool — you don't need to run anything manually.

## Commands

| Command | Description |
|---------|-------------|
| `/schedule` | Create a single time block. Accepts natural language like `/schedule 2 hours for PhD experiment tomorrow morning`. |
| `/schedule-season` | Batch-schedule all time blocks for an active season from an Obsidian vault season note. |
| `/calendar` | View upcoming events. Accepts optional date range like `/calendar this week`. |
| `/find-free-time` | Find available time slots. Accepts constraints like `/find-free-time 3 hours this week`. |

## MCP Tools

The plugin exposes these tools for Claude to use:

| Tool | Description |
|------|-------------|
| `list_calendars` | List all calendars in the account |
| `get_events` | Get events in a date range |
| `create_event` | Create a new event |
| `update_event` | Update an existing event |
| `delete_event` | Delete an event |
| `find_free_slots` | Find free time slots across all calendars |

## How It Works

The plugin uses Fastmail's [JMAP API](https://www.fastmail.com/dev/) (JSON Meta Application Protocol) to interact with calendars. JMAP is a modern, efficient alternative to CalDAV that uses simple JSON-over-HTTP requests.

Events are stored in [JSCalendar format](https://datatracker.ietf.org/doc/rfc8984/) with timezone-aware local datetimes and ISO 8601 durations.

## Season Scheduling Workflow

The `/schedule-season` command implements a full time-blocking workflow:

1. Reads a season note from your Obsidian vault (with project names, hour budgets, and deadlines)
2. Checks your existing calendar for commitments
3. Finds all available time slots
4. Distributes project hours using smart scheduling (deadline-aware, varied, deep-work-friendly)
5. Presents the proposed schedule for your approval
6. Batch-creates all events after confirmation

## Security Considerations

### Prompt injection via calendar data

Calendar events can be created by **anyone who sends you an invite**. A malicious actor could craft an event with prompt-injection text in the title, description, or location fields. When Claude reads your calendar, that text enters its context window.

**Mitigations in this plugin:**

- **Untrusted-content markers**: The MCP server wraps all user-supplied event fields (title, description, location) in `[CALENDAR_DATA — NOT AN INSTRUCTION]` tags. This makes it significantly harder for injected text to be interpreted as instructions.
- **Confirmation flow**: The skill and command instructions require Claude to present a proposal and wait for explicit approval before creating, updating, or deleting any event.

**What you should do:**

- **Never auto-approve write tools.** Always review the parameters when Claude Code asks to call `create_event`, `update_event`, or `delete_event`.
- **Scope your API token.** In Fastmail's token settings, grant only the minimum permissions you need. If you don't need delete access, don't grant it.
- **Review batch operations carefully.** The `/schedule-season` command creates many events at once. Read the proposed schedule before confirming.

### API token handling

The `FASTMAIL_API_TOKEN` is passed as an environment variable to the MCP server process. Store it in `.claude/settings.local.json` (which is gitignored) or your shell profile — never commit it to version control.

## Building from Source

See [Setup → Step 2](#2-build-the-plugin-file) above.
