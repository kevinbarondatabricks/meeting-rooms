Review Google Calendar for the current week and ensure every meeting has a conference room reserved in SFO Floor 15. If a meeting already has an SFO Floor 15 room, skip it. If not, book one following the preference order below. Send a single summary email at the end if any meetings could not be booked.

## Room Preference Order

### SFO Floor 15 (Primary — try in this exact order)
1. **Gold Striker (6)** — `databricks.com_3731323338333530353431@resource.calendar.google.com`
2. **Giant Dipper (7)** — `databricks.com_2d36383936333638393530@resource.calendar.google.com`
3. **Boomerang (8)** — `databricks.com_3737393832333638313435@resource.calendar.google.com`
4. **Goliath (7)** — `databricks.com_2d3635303532333032343135@resource.calendar.google.com`

### SFO Floor 15 (Secondary — any other Floor 15 room, try in this order)
5. **Flashback (7)** — `databricks.com_3835383730373530393935@resource.calendar.google.com`
6. **Medusa (6)** — `databricks.com_3936353930323939323235@resource.calendar.google.com`

> Do NOT book Road Runner — it requires manual approval from Jenni Avdic.

### SFO Floor 14 (Fallback — only if no Floor 15 rooms are available)
7. **Smith (5)** — `databricks.com_36333033363638333831@resource.calendar.google.com`
8. **Selinger (5)** — `databricks.com_32343836363932343935@resource.calendar.google.com`
9. **Ochoa (6)** — `c_1889v4e58g5riik7luesif4glqosg@resource.calendar.google.com`

## Steps

### 1. Determine the current week's date range
- Calculate Monday through Friday of the current week (use the current date as reference).
- Use Pacific Time (America/Los_Angeles) for all time boundaries.

### 2. Fetch all calendar events for the week
- Use `mcp__google__google_read_api_call` with endpoint `calendar/events`.
- Set `timeMin` to Monday 00:00:00 and `timeMax` to Saturday 00:00:00 (Pacific).
- Use `singleEvents: true` and `orderBy: startTime` to expand recurring events.
- Set `maxResults: 250` to capture a full week.

### 3. Filter to real meetings only
Skip events that are:
- All-day events (have `start.date` instead of `start.dateTime`)
- Working location events (`eventType: "workingLocation"`)
- Events with `status: "cancelled"`
- Events the user has declined (`responseStatus: "declined"`)

### 4. Check each meeting for an SFO Floor 15 room
A meeting already has an SFO Floor 15 room if any attendee with `resource: true` has a `displayName` containing `"SFO-15"`. If it does, mark it as covered and move on.

### 5. Check room availability for meetings that need rooms
- Use `mcp__google__google_read_api_call` with endpoint `calendar/freebusy`.
- Query all 9 rooms listed above for the full week timespan in a single call.
- For each meeting that needs a room, check which rooms from the preference list are free during that meeting's exact time slot.
- A room is free if none of its busy periods overlap with the meeting's start/end time.

### 6. Book rooms
For each meeting needing a room, pick the highest-preference available room and update the event:
- Use `mcp__google__google_write_api_call` with endpoint `calendar/events/{eventId}`.
- Add the room's resource email to the attendees list (preserve all existing attendees).
- Update the `location` field to include the room name.
- If the update fails (e.g., permission denied because you're not the organizer and `guestsCanModify` is not true), log the failure and continue.

### 7. Present a summary table
After processing all events, display a table like this:

| Day | Time | Meeting | Room Status |
|-----|------|---------|-------------|
| Mon | 10:00–10:30 | Kevin + Nilesh 1:1 | SFO-15-Boomerang (already had) |
| Mon | 12:30–1:00 | Merlin Sync | SFO-15-Gold Striker (booked) |
| Mon | 2:00–2:30 | PERF Kevin / Farhad | FAILED — permission denied |

### 8. Send failure notification email (if needed)
If any meetings could NOT be booked (no rooms available or update failed), send a single summary email to `kevin.baron@databricks.com` using `mcp__google__google_write_api_call` with endpoint `gmail/messages`.

Subject: `Meeting Rooms — [N] meetings need manual booking ([date range])`

Body should list each unbooked meeting with its day, time, name, and reason (no rooms available / permission denied / etc.).

Only send this email if there are actual failures. If everything was booked successfully, skip the email.
