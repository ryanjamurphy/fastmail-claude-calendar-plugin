# /schedule-season — Batch Schedule a Season's Time Blocks

The primary workflow command. Reads an active season note from the Obsidian vault, parses the engagement list, and batch-schedules all time blocks across the season's date range.

## Usage

```
/schedule-season [path-to-season-note] [--hours START-END] [--deep-work PREFERENCE] [--exclude DAYS]
```

**Examples:**
- `/schedule-season`
- `/schedule-season "Calendar/Seasons/Winter Season of Shipping and Catching Up.md" --hours 9am-4pm --deep-work mornings --exclude weekends`
- `/schedule-season --hours 8am-3pm --exclude "no Fridays"`

## Arguments

- **Season note path** (optional): Path to the season note in the Obsidian vault. Defaults to the most recent file in `Calendar/Seasons/`.
- `--hours` (optional): Working hours window (e.g., `9am-5pm`, `8am-3pm`). Default: `9am-5pm`.
- `--deep-work` (optional): When to schedule cognitively demanding work (`mornings`, `before noon`, `afternoons`). Default: `mornings`.
- `--exclude` (optional): Days to exclude (`weekends`, `no Fridays`, `no Mondays`). Default: none.

## Instructions for Claude

### Step 1: Read the Season Note

Read the active season note from the user's Obsidian vault. Look for:

- The **engagement list table** — a markdown table with columns like: Project, Domain, Estimated Hours, MVP, and optionally Due Date.
- The **season date range** — usually mentioned in the note's header or metadata.
- Any **existing time blocking** section that may need updating.

Example engagement list table:
```markdown
| Project | Domain | Estimated Hours | MVP | Due Date |
|---------|--------|----------------|-----|----------|
| PhD Experiment — Tutorial videos | Research | 20 | Complete 3 tutorial scripts | March 15 |
| Partner emails | Admin | 5 | Send all Q1 updates | March 10 |
| Feature launch prep | Engineering | 15 | Demo-ready build | March 20 |
| Blog post series | Writing | 10 | Publish 2 posts | March 25 |
```

### Step 2: Parse the Engagement List

Extract from each row:
- **Project name**: Used as the event title
- **Estimated hours**: Total hours to schedule for this project
- **MVP**: Context for what the work entails (include in event description)
- **Due date**: Hard deadline (if present) — drives priority

### Step 3: Read Existing Calendar Events

Use `get_events` to fetch all events across the season's date range. This reveals:
- Already-scheduled commitments (meetings, appointments, existing time blocks)
- Available windows for new time blocks

### Step 4: Find Free Slots

Use `find_free_slots` across the full season date range with a minimum duration of 30 minutes. This gives the raw available time.

### Step 5: Distribute Hours Across Slots

Apply these **scheduling principles** when distributing project hours:

1. **Deadline-first**: Projects with hard due dates get scheduled before their deadlines. Front-load items due soonest.

2. **Variety**: Spread different projects across the week. Don't marathon one project for days straight. Aim for 2–3 different projects per day.

3. **Deep work preference**: Schedule cognitively demanding work (PhD experiment, writing, complex engineering) during the user's preferred deep work window (default: mornings). Schedule lighter work (emails, meeting prep, admin) in shorter or less optimal slots.

4. **Buffer time**: Leave 15-minute gaps between time blocks. Don't fill every available slot — aim for ~80% utilization.

5. **Realistic sessions**: Individual time blocks should be 1–3 hours. Never schedule a single block longer than 3 hours. Break large allocations into multiple sessions.

6. **Front-load urgency**: Items due in the first few days get scheduled immediately.

7. **Respect working hours**: Only schedule within the user's specified working hours window.

8. **Respect excluded days**: Don't schedule on excluded days (e.g., weekends).

### Step 6: Present the Proposed Schedule

Format the proposed schedule as a clear table for user review:

```markdown
## Proposed Schedule

### Week of March 3–7

| Day | Time | Duration | Project | Task Focus |
|-----|------|----------|---------|------------|
| Mon Mar 3 | 9:00–11:00 | 2h | PhD Experiment | Tutorial video script #1 |
| Mon Mar 3 | 11:15–12:15 | 1h | Partner emails | Q1 update drafts |
| Mon Mar 3 | 1:00–3:00 | 2h | Feature launch prep | API integration |
| Tue Mar 4 | 9:00–11:00 | 2h | Blog post series | Draft post #1 |
| ... | ... | ... | ... | ... |

### Hour Budget Summary

| Project | Budgeted | Scheduled | Remaining |
|---------|----------|-----------|-----------|
| PhD Experiment | 20h | 20h | 0h |
| Partner emails | 5h | 5h | 0h |
| Feature launch prep | 15h | 14h | 1h |
| Blog post series | 10h | 10h | 0h |
```

### Step 7: Wait for User Approval

**NEVER create events without explicit user approval.** Ask:
- "Does this schedule look good? I can adjust specific blocks, swap project slots, or reschedule anything before creating the events."

Handle adjustments:
- Swap projects between slots
- Move blocks to different days/times
- Add or remove specific blocks
- Adjust durations

### Step 8: Batch-Create Events

After approval, use the `create_event` tool for each time block. For each event:
- **Title**: `"Project Name — Task Focus"` (e.g., `"PhD Experiment — Tutorial video script #1"`)
- **Description**: Include the MVP context from the engagement list
- **Calendar**: Use the user's preferred calendar (ask if not specified)
- **Timezone**: Use the configured `FASTMAIL_TIMEZONE`

Report progress as events are created.

### Step 9: Update the Season Note (Optional)

If the user wants, update the season note's "Time blocking" section with the scheduled plan. This creates a reference copy in the vault.

## Error Handling

- If the season note can't be found, ask the user for the correct path
- If the engagement list table can't be parsed, show what was found and ask for clarification
- If there isn't enough free time for all projects, show the shortfall and ask the user to prioritize
- If any event creation fails, report it and continue with the remaining events
