---
name: aiera-earnings-call-research
description: Find a company's most recent earnings call on Aiera and pull its full transcript, using industry-standard symbology rather than Aiera-internal ids.
api: Aiera REST API
generated: '2026-09-14'
method: generated
source: openapi/aiera-rest-api-openapi.json, https://rest.aiera.com/docs/find-events, https://rest.aiera.com/docs/get-transcripts
operations:
  - get_get_equities
  - get_get_events
  - get_get_event
  - get_get_event_transcripts
  - get_get_event_transcripts_text
---

# Research a company's earnings call

Base URL `https://premium.aiera.com/api`. Every request carries `X-API-Key: <your key>`.

## 1. Resolve the company to an equity

Do not guess `equity_id` — Aiera ids are bare integers and carry no type information. Resolve through
symbology instead.

`GET /equities-v2/` (`get_get_equities`) accepts `bloomberg_ticker`, `isin`, `ric`, `permid`, `cusip`
or `ticker`. Use whichever identifier your system already holds; ISIN and CUSIP are unambiguous,
a bare ticker is not.

Read `equity_id` and `company_id` from the result. `mic` tells you which venue the listing is on, and
`gics_sector` / `gics_sub_sector` give you GICS classification if you need to peer-group later.

## 2. Find the earnings call

`GET /events-v2/` (`get_get_events`) with the equity identifier plus `start_date` and `end_date`.
Filter to earnings by event type. Results paginate with `page` and `page_size` (default 25, max 25) —
check the `pagination` block rather than assuming a single page.

Sort by date descending and take the first event to get "most recent". Keep its `event_id`.

## 3. Pull the transcript

Two options, and they are not equivalent:

- `GET /events-v2/{event_id}` (`get_get_event`) returns the event with `transcripts` embedded, plus
  `linguistics`, `price_data`, `event_tags` and `company_metadata`. Use this when you want the
  surrounding context.
- `GET /events-v2/{event_id}/transcript` (`get_get_event_transcripts`) returns just the transcript
  items. Each item carries `speaker_id`, timing, and optional `word_offsets`.

For a plain-text dump suitable for feeding to a model, use
`GET /events-v2/{event_id}/transcript/text` (`get_get_event_transcripts_text`). HTML and PDF
renditions exist at `/transcript/html` and `/transcript/pdf`. These are separate paths, not `Accept`
header negotiation.

## 4. Attribute the analyst questions

Transcript items resolve to a `TranscriptSpeaker`, which resolves to a `Person` and to a `Firm`. The
`Firm` is what lets you say "this question came from a particular bank" rather than just "an analyst
asked". If you need the speaker's history, `GET /people-v1/{person_id}/` (`get_get_person`) returns
their `representations` and the events they have spoken at.

## Things that will bite you

- **Entitlements, not plans.** A valid key can still get HTTP 403 on content the account is not
  entitled to. Aiera's 403 covers both "bad key" and "not entitled" and the spec does not distinguish
  them. Read `/users-v1/entitlements` (`get_get_entitlements`) if you need to know in advance.
- **No rate limits are published.** There is no documented 429, no `Retry-After` and no
  `X-RateLimit-*` header. Build backoff defensively; you cannot read a budget off the contract.
- **Error bodies are undeclared.** 400 and 404 carry a description in the spec and no schema. Observed
  bodies differ by gateway: `{"detail": "..."}` from the application, `{"message": "..."}` from AWS
  API Gateway. Do not write a parser that assumes one shape.
