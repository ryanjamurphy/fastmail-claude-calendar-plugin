# CalDAV Calendar API Reference

Quick reference for the Fastmail CalDAV calendar integration used by this plugin.

## Authentication

- **CalDAV server**: `https://caldav.fastmail.com/`
- **Auth**: HTTP Basic with Fastmail username + app password
- **Protocol**: CalDAV (RFC 4791) over HTTPS

## Environment Variables

| Variable | Required | Description |
|----------|----------|-------------|
| `FASTMAIL_USERNAME` | Yes | Full Fastmail email address |
| `FASTMAIL_APP_PASSWORD` | Yes | App password with calendar access |
| `FASTMAIL_TIMEZONE` | No | IANA timezone (default: `America/St_Johns`) |

## MCP Tools

### list_calendars

Returns all calendars with their CalDAV URLs and display names.

### get_events

Fetch events in a date range. Uses CalDAV time-range REPORT queries.

**Parameters:**
- `calendarUrl` (optional) — CalDAV URL of a specific calendar
- `after` — ISO 8601 UTC datetime (start of range)
- `before` — ISO 8601 UTC datetime (end of range)

**Returns:** Array of events with `url`, `title`, `start`, `end`, `duration`, `description`, `location`. The `url` field is needed for update/delete operations.

### create_event

Create a new calendar event. Generates an ICS file and PUTs it to the CalDAV server.

**Parameters:**
- `calendarUrl` — CalDAV URL of the target calendar (from `list_calendars`)
- `title` — Event summary
- `description` (optional) — Event notes
- `start` — ISO 8601 datetime
- `end` — ISO 8601 datetime
- `timeZone` (optional) — IANA timezone
- `location` (optional) — Event location

### update_event

Update an existing event. Fetches the current ICS, modifies it, and PUTs it back with If-Match for safe concurrent updates.

**Parameters:**
- `eventUrl` — CalDAV URL of the event (from `get_events`)
- `title`, `description`, `start`, `end`, `location`, `timeZone` (all optional)

### delete_event

Delete an event by its CalDAV URL.

**Parameters:**
- `eventUrl` — CalDAV URL of the event (from `get_events`)

### find_free_slots

Find available time slots by checking all calendars for conflicts.

**Parameters:**
- `after`, `before` — Date range (ISO 8601 UTC)
- `minDuration` — Minimum slot duration (ISO 8601 duration, e.g. `PT1H`)
- `calendarUrl` (optional) — Only check this calendar for conflicts

## iCalendar (ICS) Format

Events are stored in iCalendar format (RFC 5545). The MCP server handles all ICS parsing and generation — tools accept and return structured JSON.

### Duration Format

ISO 8601 durations:
- `PT30M` — 30 minutes
- `PT1H` — 1 hour
- `PT1H30M` — 1 hour 30 minutes
- `PT2H` — 2 hours

## Error Handling

CalDAV errors are returned as HTTP status codes:
- `404` — Event or calendar not found
- `412` — Precondition failed (event was modified by another client)
- `403` — Permission denied (check app password scope)
