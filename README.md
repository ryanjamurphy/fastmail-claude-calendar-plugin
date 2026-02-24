# fastmail-calendar

A Claude Cowork plugin that integrates with Fastmail's calendar via CalDAV. Read, create, update, and delete calendar events. Supports time-blocking workflows and season-based batch scheduling.

## Setup

### 1. Create a Fastmail App Password

1. Go to **Fastmail Settings → Privacy & Security → Manage app passwords**
2. Click **New app password**
3. Give it a name (e.g. "Claude Calendar Plugin")
4. Under access, select **Mail, Contacts & Calendars** (or a scope that includes CalDAV)
5. Click **Generate password** and copy the generated password

You'll also need your full Fastmail email address (e.g. `you@fastmail.com`).

> **Note:** Fastmail does not currently support JMAP for calendars. This plugin uses CalDAV, which requires an app password — not an API token.

### 2. Build the Plugin File

From the root of this repository, install dependencies and package the plugin:

```bash
npm install
zip -r /tmp/fastmail-calendar.plugin . -x "node_modules/*" "*.DS_Store" ".git/*"
```

This creates `/tmp/fastmail-calendar.plugin` — a zip archive containing the server, skills, and plugin manifest.

### 3. Install in Claude Cowork

1. In Claude Cowork, select **Customize** in the left sidebar
2. Select **Browse plugins**
3. Go to the **Personal** tab in the modal that appears
4. Click the **+** in the upper-left of that tab
5. Select **Upload plugin** and upload the `.plugin` file you built in step 2

When prompted, set the required environment variables:

```
FASTMAIL_USERNAME=you@fastmail.com
FASTMAIL_APP_PASSWORD=your-app-password-here
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

The plugin uses Fastmail's [CalDAV](https://www.fastmail.help/hc/en-us/articles/360058752754-How-to-synchronize-a-calendar) endpoint (`caldav.fastmail.com`) to interact with calendars. Authentication uses a Fastmail app password with Basic auth.

Events are stored in [iCalendar format (ICS)](https://datatracker.ietf.org/doc/html/rfc5545) and converted to/from structured JSON by the MCP server. The [tsdav](https://github.com/natelindev/tsdav) library handles the CalDAV protocol.

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
- **Scope your app password.** When creating the app password, only grant the access you need.
- **Review batch operations carefully.** The `/schedule-season` command creates many events at once. Read the proposed schedule before confirming.

### Credential handling

`FASTMAIL_USERNAME` and `FASTMAIL_APP_PASSWORD` are passed as environment variables to the MCP server process. Store them in `.claude/settings.local.json` (which is gitignored) or your shell profile — never commit them to version control.

## Building from Source

See [Setup → Step 2](#2-build-the-plugin-file) above.
