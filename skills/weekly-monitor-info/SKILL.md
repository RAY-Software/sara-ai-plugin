---
name: weekly-monitor-info
description: Use when someone asks whether they can keep track of their rating, reviews and local position over time. Explains what could be monitored and shares an informational link through get_sara_info_link.
---

# Weekly Monitor Information

Answers: "Can I keep an eye on this every week?"

There is no monitoring tool inside this plugin. This skill only explains what could be tracked and shares an informational page.

## Ground rules

- Ask the user only for the business name and the city. Never ask for emails, personal phone numbers, patient or client data, or the earlier conversation.
- One business per request. Do not list businesses in bulk.
- Always say "Data from Google Maps" when you present results.
- Missed bookings are an ESTIMATE. Show the range (low, mid, high) and list the assumptions the tool returns. Never present a single number as fact.
- Stay neutral and factual. Do not mention plans, offers or anything commercial unless the user asks about next steps.

## Steps

1. Make sure a scan exists. If the user has not scanned their business yet, run `find_business` and `scan_business` first (see the other skills for the full flow). You need the `scan_id` from `scan_business`.
2. Explain what a weekly check could cover, in plain words: rating changes, new review count, position against nearby competitors, and the missed bookings estimate. Say these come from the same public data as the scan.
3. Call `get_sara_info_link` with `{ "scan_id": "<scan_id>" }` and share the informational link it returns. Describe it as a page with more information.
4. Do not ask for an email address or phone number in the chat. If the user wants to share contact details, tell them to do it on the page, not here.
5. Cite "Data from Google Maps" when you restate any scan numbers.

## Handling errors

Every tool returns `structuredContent` with `ok`. When `ok` is false read `structuredContent.error.code` and `error.fix`:
- `daily_capacity_reached`: the shared daily limit for new scans was reached. Say so, suggest trying again tomorrow, and do not retry in a loop.
- `temporarily_unavailable` or `upstream_error`: say the service is busy, offer to try again in a few minutes (see `retry_at` if present). One retry at most.
- `rate_limited_subject`: the user reached their personal scan limit for now. Explain it and stop.
- `scan_required`: call `scan_business` for the same `place_id` first, then repeat the call.
- `unsupported_vertical`: dental or a business type without appointments. Explain, do not retry.
- `unsupported_region`: the business is outside the US. Explain, do not retry.
- `invalid_argument`: fix the input (for example a malformed `place_id`) using `find_business` and try once more.
- `not_found`: the place does not exist. Ask the user to check the name and city and search again.
