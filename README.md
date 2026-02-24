# fastmail-calendar

A Claude Cowork plugin that integrates with Fastmail's calendar via JMAP. Read, create, update, and delete calendar events. Supports time-blocking workflows and season-based batch scheduling.

## Setup

### 1. Get a Fastmail API Token

1. Go to **Fastmail Settings → Privacy & Security → Manage API tokens**
2. Create a new token with **calendar read/write** scope
3. Copy the token

### 2. Install the Plugin

Install the `.plugin` file in Claude Cowork, then set the required environment variable:

```
FASTMAIL_API_TOKEN=your-token-here
```

Optionally set your timezone (defaults to `America/St_Johns`):

```
FASTMAIL_TIMEZONE=America/New_York
```

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

## Building from Source

```bash
npm install
```

## Packaging

```bash
zip -r /tmp/fastmail-calendar.plugin . -x "node_modules/*" "*.DS_Store" ".git/*"
```
