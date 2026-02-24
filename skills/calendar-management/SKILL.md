# Calendar Management Skill

## Triggers

This skill activates when the user mentions:
- Scheduling, time blocking, calendar, availability
- "Schedule my time blocks"
- "What's on my calendar?"
- "Find me some free time"
- "Schedule this season" or "plan my season"
- Any request to create, move, update, or delete calendar events
- Questions about their availability or upcoming commitments

## Capabilities

You have access to a Fastmail calendar integration via MCP tools. You can:

1. **Read calendars and events** — List all calendars, view events for any date range
2. **Create events** — Schedule new time blocks with title, description, time, timezone, and calendar
3. **Update events** — Reschedule, rename, or modify existing events
4. **Delete events** — Remove events by URL
5. **Find free time** — Discover available slots across all calendars
6. **Batch schedule** — Plan and create multiple time blocks for a season's worth of projects

## Available MCP Tools

- `list_calendars` — List all Fastmail calendars (names, URLs)
- `get_events` — Fetch events in a date range
- `create_event` — Create a new event
- `update_event` — Modify an existing event
- `delete_event` — Delete an event
- `find_free_slots` — Find available time slots

## Season-Aware Scheduling

The user follows a **seasonal planning** workflow using Obsidian:

1. **Season notes** live in `Calendar/Seasons/` in their Obsidian vault
2. Each season note contains an **engagement list** — a markdown table listing projects, estimated hours, MVPs, and due dates
3. The `/schedule-season` command reads this table and creates time blocks for all projects

When scheduling a season:
- Read the season note to understand the project portfolio
- Parse the engagement list table for hours, priorities, and deadlines
- Check existing calendar events to understand current commitments
- Distribute project hours across free slots following the scheduling principles below
- Always present the full proposed schedule for review before creating any events

## Scheduling Principles

Follow these rules when placing time blocks:

### Priority Ordering
1. **Deadline-first**: Projects with hard due dates get scheduled before their deadlines
2. **Front-load urgency**: Items due soonest get scheduled in the first available days
3. **Spread the rest**: Remaining projects fill available time with variety

### Time Block Structure
- **Duration**: 1–3 hours per block. Never schedule a single block longer than 3 hours.
- **Buffers**: Leave 15-minute gaps between blocks
- **Utilization**: Aim for ~80% of available time. Don't fill every slot.
- **Working hours**: Default 9 AM – 5 PM unless the user specifies otherwise

### Daily Variety
- Schedule 2–3 different projects per day
- Don't marathon one project for an entire day
- Alternate between deep and light work

### Deep Work Preference
- **Mornings** (default): Schedule cognitively demanding work (research, writing, complex engineering) before noon
- **Afternoons**: Schedule lighter tasks (emails, admin, meeting prep) in the afternoon or shorter slots
- The user can override this preference

### Exclusions
- Respect the user's excluded days (weekends by default)
- Don't schedule over existing events
- Check ALL calendars when determining busy times (avoid double-booking)

## Confirmation Flow

**CRITICAL: Never silently create, modify, or delete events.**

Always follow this flow:
1. Analyze the request and gather information
2. Present a clear proposal to the user
3. Wait for explicit approval
4. Execute the changes
5. Confirm what was done

For batch operations (like `/schedule-season`), present the full schedule as a formatted table and get approval before creating any events.

## Event Naming Convention

Time block events should use the format:
```
Project Name — Specific Task
```

Examples:
- `PhD Experiment — Tutorial video script #1`
- `Blog Post Series — Draft post on API design`
- `Feature Launch Prep — API integration testing`
- `Partner Emails — Q1 update drafts`

This makes events identifiable at a glance in the calendar.

## Timezone Handling

- All times are handled in the configured `FASTMAIL_TIMEZONE` (default: `America/St_Johns`)
- When interpreting user input like "tomorrow morning" or "Wednesday at 2pm," use the configured timezone
- Always display times in the user's timezone for readability
- The MCP server stores events in UTC and handles timezone conversion

## Working with the Obsidian Vault

Season notes follow this structure:

```markdown
# Season Name

## Engagement List

| Project | Domain | Estimated Hours | MVP | Due Date |
|---------|--------|----------------|-----|----------|
| ... | ... | ... | ... | ... |

## Time Blocking

(This section gets populated after scheduling)
```

When reading season notes:
- The engagement list is the authoritative source for what needs scheduling
- Estimated hours drive how many time blocks to create
- MVPs provide context for event descriptions
- Due dates determine scheduling priority
