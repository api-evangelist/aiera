---
name: aiera-event-calendar-monitoring
description: Track upcoming and estimated corporate events on Aiera, keep a local copy in sync with modified_since, and subscribe to the iCal feeds.
api: Aiera REST API
generated: '2026-09-14'
method: generated
source: openapi/aiera-rest-api-openapi.json, https://rest.aiera.com/docs/get-calendar, https://rest.aiera.com/docs/estimated-events
operations:
  - get_get_calendar
  - get_get_calendar_entry
  - get_get_event_estimates
  - get_equities_covered
  - get_calendar_ical_user
  - get_get_high_priority_calendar
  - get_get_events
---

# Monitor the corporate event calendar

Base URL `https://premium.aiera.com/api`, header `X-API-Key: <your key>`.

## Confirmed versus estimated

Aiera separates events it knows about from events it predicts, and conflating them is the usual
mistake.

- `GET /calendar-v2/` (`get_get_calendar`) — the calendar of scheduled events.
- `GET /calendar-v2/estimated` (`get_get_event_estimates`) — estimated events. Each `EstimateItem`
  carries a predicted date; `EstimateActual` carries the settled one once it firms up.
- `GET /calendar-v2/{event_id}` (`get_get_calendar_entry`) — a single calendar entry.

A realised `Event` links back to the estimate that predicted it through `estimate_id`, so you can
measure how far a prediction moved before it settled.

## Check coverage before you trust a gap

`GET /calendar-v2/coverage` (`get_equities_covered`) returns which equities Aiera covers. An empty
calendar for a name may mean nothing is scheduled, or that the name is not covered, or that your
account is not entitled to it. Coverage answers the first of those three.

## Keep a local copy in sync

Five operations accept `modified_since`. Use it for incremental sync rather than re-pulling a date
range: store the timestamp of your last successful pull and pass it on the next one. Combine with
`include_deleted` so tombstones reach you — without it, an event that was cancelled simply vanishes
from results and your local copy keeps a ghost.

## Subscribe instead of polling

Two iCalendar (RFC 5545) feeds exist:

- `GET /calendars/ical/user` (`get_calendar_ical_user`) — the account's calendar.
- `GET /calendars/ical/high_priority` (`get_get_high_priority_calendar`) — the high-priority subset.

Any standard calendar client consumes these with no bespoke connector. If a human needs to see the
schedule, this is a better answer than building a UI.

## Things that will bite you

- **There are no webhooks.** Aiera publishes no event-push surface, no AsyncAPI and no callback
  registration. Monitoring means polling (use `modified_since`) or subscribing to the iCal feed.
- **Two calendar surfaces exist.** The served spec uses `/calendar-v2/*`; Aiera's GitHub-published
  specs use `/calendar/*` (`getCalendar`, `getCalendarById`, `getCalendarCoverage`). They are not the
  same paths. Use the one your base URL actually serves.
- **`page_size` maxes at 25** on the documented surface, while Aiera's own MCP package defaults to 50
  with a max of 100. The two first-party sources disagree; test rather than assume.
