# /find-free-time — Find Available Time Slots

Find free time slots in your calendar for scheduling work blocks.

## Usage

```
/find-free-time <duration> [date range]
```

**Examples:**
- `/find-free-time 2 hours this week`
- `/find-free-time 90 minutes tomorrow`
- `/find-free-time 3 hours next two weeks`
- `/find-free-time 1 hour Monday-Wednesday`

## Instructions for Claude

When the user runs `/find-free-time`, follow these steps:

1. **Parse the request** — Extract:
   - **Minimum duration**: How long the free slot needs to be (e.g., "2 hours" → `PT2H`, "90 minutes" → `PT1H30M`)
   - **Date range**: When to search (default: this week)

2. **Convert to parameters** — Translate to ISO 8601:
   - Duration → ISO 8601 duration string (e.g., `PT2H`, `PT1H30M`)
   - Date range → UTC `after` and `before` timestamps

3. **Find free slots** — Use the `find_free_slots` tool with the computed parameters.

4. **Filter to working hours** — From the raw free slots, filter or trim to reasonable working hours (default: 8 AM – 6 PM in the configured timezone) unless the user specifies otherwise.

5. **Present options** — Show available slots in a clear format:

   ```
   ## Free slots (≥ 2 hours) this week

   1. **Monday, March 3** — 9:00 AM – 12:00 PM (3 hours)
   2. **Monday, March 3** — 1:30 PM – 4:00 PM (2.5 hours)
   3. **Tuesday, March 4** — 10:00 AM – 1:00 PM (3 hours)
   4. **Wednesday, March 5** — 9:00 AM – 11:30 AM (2.5 hours)
   5. **Wednesday, March 5** — 2:00 PM – 5:00 PM (3 hours)
   6. **Thursday, March 6** — 9:00 AM – 12:00 PM (3 hours)

   Found 6 slots totaling 17 hours of available time.
   ```

6. **Offer to schedule** — Ask: "Would you like me to schedule a time block in any of these slots?"

   If the user picks a slot, transition to the `/schedule` workflow to create the event.

## Filtering Guidelines

- Only show slots during working hours (8 AM – 6 PM by default)
- Skip slots on weekends unless the user explicitly asks
- Merge adjacent slots that are only separated by a small gap (< 15 min)
- Sort slots chronologically
- Show the slot duration alongside start/end times
- Cap the list at ~10 slots to avoid overwhelming output; mention if more exist
