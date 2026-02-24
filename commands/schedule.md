# /schedule — Create a Time Block

Create a single focused time block on your Fastmail calendar.

## Usage

```
/schedule <description>
```

**Examples:**
- `/schedule 2 hours for PhD experiment tomorrow morning`
- `/schedule 90 minutes for writing Wednesday afternoon`
- `/schedule 1 hour meeting prep Friday at 2pm`

## Instructions for Claude

When the user runs `/schedule`, follow these steps:

1. **Parse the request** — Extract:
   - What the time block is for (the project or activity name)
   - How long it should be (duration)
   - When they want it (date/time preference)

2. **Check the calendar** — Use the `get_events` tool to see what's already scheduled around the requested time. Use today's date and the configured timezone (`FASTMAIL_TIMEZONE`) as context for relative dates like "tomorrow" or "Wednesday."

3. **Find available time** — Use the `find_free_slots` tool to locate a slot that fits the user's preference. If they said "morning," look for slots before noon. If they said "afternoon," look after noon.

4. **List calendars if needed** — If the user hasn't specified which calendar, use `list_calendars` to show options and ask, or default to the first writable calendar.

5. **Propose the time block** — Present the proposed event to the user:
   ```
   📅 Proposed time block:
   Title: PhD Experiment — Tutorial videos
   Calendar: Work
   Start: Tuesday, March 5 at 9:00 AM (America/St_Johns)
   End: Tuesday, March 5 at 11:00 AM (America/St_Johns)
   Duration: 2 hours
   ```

6. **Wait for confirmation** — Ask the user to confirm before creating.

7. **Create the event** — Use the `create_event` tool with the confirmed details.

8. **Confirm creation** — Show the created event details and ID.

## Scheduling Preferences

- Prefer morning slots for cognitively demanding work (deep work)
- Prefer afternoon slots for lighter tasks (emails, admin, meetings)
- Leave 15-minute buffers around existing events
- Don't schedule before 8 AM or after 6 PM unless the user specifically asks
- Time blocks should be between 30 minutes and 3 hours

## Event Naming

Name time blocks descriptively: `"Project Name — Specific Task"` (e.g., `"PhD Experiment — Tutorial videos"`, `"Season Planning — Q2 roadmap"`)
