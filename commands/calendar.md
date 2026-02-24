# /calendar — View Upcoming Events

Display calendar events for a specified date range.

## Usage

```
/calendar [date range]
```

**Examples:**
- `/calendar` — Shows today's events
- `/calendar today` — Shows today's events
- `/calendar this week` — Shows events for the current week (Monday–Sunday)
- `/calendar next week` — Shows events for next week
- `/calendar March 3-7` — Shows events for a specific date range
- `/calendar next two weeks` — Shows events for the next 14 days
- `/calendar tomorrow` — Shows tomorrow's events

## Instructions for Claude

When the user runs `/calendar`, follow these steps:

1. **Parse the date range** — Interpret the user's natural language date range using today's date and the configured timezone (`FASTMAIL_TIMEZONE`). Default to "today" if no range is specified.

   Convert to ISO 8601 UTC datetimes for the `after` and `before` parameters:
   - "today" → midnight today to midnight tomorrow (in the configured timezone, converted to UTC)
   - "this week" → Monday 00:00 to Sunday 23:59 of the current week
   - "next two weeks" → today to 14 days from now

2. **Fetch events** — Use the `get_events` tool with the computed `after` and `before` parameters.

3. **Format the output** — Present events in a clean, readable format grouped by day:

   ```
   ## Tuesday, March 5

   ▸ 9:00 AM – 10:30 AM  |  Team Standup  (Work calendar)
   ▸ 11:00 AM – 1:00 PM  |  PhD Experiment — Tutorial videos  (Personal calendar)
   ▸ 2:00 PM – 3:00 PM   |  1:1 with Manager  (Work calendar)

   ## Wednesday, March 6

   ▸ 9:00 AM – 11:00 AM  |  Deep Work — Blog post draft  (Personal calendar)
   ▸ 1:00 PM – 2:00 PM   |  Lunch with Alex  (Personal calendar)
   ```

4. **Show summary** — After the event list, show a brief summary:
   ```
   3 events on Tuesday, 2 events on Wednesday. 4 hours of meetings, 5 hours of focus time.
   ```

## Formatting Guidelines

- Group events by day with clear day headers
- Show times in the configured timezone
- Include the calendar name if events span multiple calendars
- Show event descriptions only if the user asks for detail
- For empty days in the range, either skip them or note "No events"
- Use 12-hour time format for readability
