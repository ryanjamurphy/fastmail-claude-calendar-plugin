# JMAP Calendar API Reference

Quick reference for the Fastmail JMAP calendar API used by this plugin.

## Authentication

- **Session endpoint**: `GET https://api.fastmail.com/jmap/session`
- **Auth**: `Authorization: Bearer <FASTMAIL_API_TOKEN>`
- **Account ID**: Extracted from `session.accounts` (first key)
- **API URL**: Use `session.apiUrl` for all subsequent requests

## Request Format

All JMAP requests are `POST` to the API URL with this structure:

```json
{
  "using": [
    "urn:ietf:params:jmap:core",
    "urn:ietf:params:jmap:calendars"
  ],
  "methodCalls": [
    ["MethodName", { ...arguments }, "callId"]
  ]
}
```

## Methods

### Calendar/get

List all calendars in the account.

```json
["Calendar/get", {
  "accountId": "ACCOUNT_ID",
  "properties": ["id", "name", "color", "isVisible", "isSubscribed", "myRights"]
}, "calendars"]
```

Returns `{ list: [{ id, name, color, ... }] }`.

### CalendarEvent/query

Search for events matching a filter. Returns event IDs only.

```json
["CalendarEvent/query", {
  "accountId": "ACCOUNT_ID",
  "filter": {
    "after": "2024-03-15T00:00:00Z",
    "before": "2024-03-22T00:00:00Z",
    "inCalendars": ["calendar-id"]
  },
  "sort": [{ "property": "start", "isAscending": true }],
  "limit": 200
}, "query"]
```

Filter properties:
- `after` (UTC datetime): Events starting at or after this time
- `before` (UTC datetime): Events starting before this time
- `inCalendars` (string[]): Limit to specific calendar IDs

### CalendarEvent/get

Fetch full event objects by ID.

```json
["CalendarEvent/get", {
  "accountId": "ACCOUNT_ID",
  "ids": ["event-id-1", "event-id-2"],
  "properties": ["id", "title", "description", "start", "timeZone", "duration", "locations", "calendarIds", "status", "freeBusyStatus"]
}, "events"]
```

**Back-reference pattern** (chain with query in one request):

```json
["CalendarEvent/get", {
  "accountId": "ACCOUNT_ID",
  "#ids": {
    "resultOf": "query",
    "name": "CalendarEvent/query",
    "path": "/ids"
  },
  "properties": ["id", "title", "start", "timeZone", "duration", "locations", "calendarIds"]
}, "events"]
```

### CalendarEvent/set

Create, update, or delete events.

**Create:**
```json
["CalendarEvent/set", {
  "accountId": "ACCOUNT_ID",
  "create": {
    "newEvent": {
      "calendarIds": { "calendar-id": true },
      "title": "Event Title",
      "description": "Optional description",
      "start": "2024-03-15T09:00:00",
      "timeZone": "America/St_Johns",
      "duration": "PT2H",
      "locations": {
        "loc1": { "@type": "Location", "name": "Conference Room" }
      }
    }
  }
}, "create"]
```

**Update:**
```json
["CalendarEvent/set", {
  "accountId": "ACCOUNT_ID",
  "update": {
    "event-id": {
      "title": "Updated Title",
      "start": "2024-03-15T10:00:00",
      "duration": "PT1H30M"
    }
  }
}, "update"]
```

**Delete:**
```json
["CalendarEvent/set", {
  "accountId": "ACCOUNT_ID",
  "destroy": ["event-id-1", "event-id-2"]
}, "destroy"]
```

## JSCalendar Event Properties (RFC 8984)

| Property | Type | Description |
|----------|------|-------------|
| `id` | string | Server-assigned event ID |
| `calendarIds` | object | Map of calendar ID → `true` |
| `title` | string | Event title / summary |
| `description` | string | Event description / notes |
| `start` | string | Local datetime without offset: `"2024-03-15T09:00:00"` |
| `timeZone` | string | IANA timezone: `"America/St_Johns"` |
| `duration` | string | ISO 8601 duration: `"PT2H"`, `"PT1H30M"` |
| `locations` | object | Map of location ID → Location object |
| `status` | string | `"confirmed"`, `"tentative"`, `"cancelled"` |
| `freeBusyStatus` | string | `"busy"`, `"free"`, `"tentative"` |
| `showWithoutTime` | boolean | All-day event flag |

### Location Object

```json
{
  "@type": "Location",
  "name": "Conference Room A",
  "description": "Building 3, Floor 2"
}
```

### Duration Format

ISO 8601 durations:
- `PT30M` — 30 minutes
- `PT1H` — 1 hour
- `PT1H30M` — 1 hour 30 minutes
- `PT2H` — 2 hours
- `P1D` — 1 day

## Error Handling

JMAP errors appear in method responses:

```json
["error", {
  "type": "notFound",
  "description": "Event not found"
}, "callId"]
```

Set operation errors appear in `notCreated`, `notUpdated`, or `notDestroyed`:

```json
{
  "notCreated": {
    "newEvent": {
      "type": "invalidProperties",
      "description": "Missing required property: calendarIds"
    }
  }
}
```

## Rate Limits

Fastmail applies rate limits to JMAP requests. The server handles this gracefully — if a request fails with a rate limit error, retry with exponential backoff.
