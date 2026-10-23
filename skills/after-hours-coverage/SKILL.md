---
name: after-hours-coverage
description: Use when someone asks what they lose in the evenings, nights and weekends, or whether their listed hours match when clients look for their services. Uses scan_business and estimate_missed_bookings.
---

# After Hours Coverage

Answers: "What am I missing when the business is closed or nobody picks up?"

## Ground rules

- Ask the user only for the business name and the city. Never ask for emails, personal phone numbers, patient or client data, or the earlier conversation.
- One business per request. Do not list businesses in bulk.
- Always say "Data from Google Maps" when you present results.
- Missed bookings are an ESTIMATE. Show the range (low, mid, high) and list the assumptions the tool returns. Never present a single number as fact.
- Stay neutral and factual. Do not mention plans, offers or anything commercial unless the user asks about next steps.

## Step 1: find the business

Call `find_business` with `name` and `city`. Pick the candidate whose name and address match. If several look alike, ask the user which address is theirs. If a candidate has `supported: false`, read its `reason`:
- Dental practices: reply "Dental practices aren't supported yet." and stop. Do not retry with another category.
- Businesses without appointments (restaurants, retail): explain that this checkup is for appointment based businesses and stop.
- Outside the United States: explain that only US businesses are covered for now and stop.

## Step 2: scan

Call `scan_business`. Read the `booking_readiness` factor (after hours coverage, online booking, text channel signals), `website` and `missed_bookings.observations`.

## Step 3: add the owner's reality

Ask once whether staff answer the phone during opening hours and about how many calls come in per day. Then call `estimate_missed_bookings` with `place_id` and any of `calls_per_day`, `staff_answers_phone` (0 to 100) and `avg_ticket` the user provided. Without answers, use the scan range.

## Step 4: narrate

1. What the listed hours and signals suggest about evenings and weekends, only from returned data. If hours are unknown, say so.
2. The estimated monthly missed bookings range (low, mid, high) as an ESTIMATE, with assumptions and which ones came from the user.
3. Options that do not need new staff: online booking that works at any hour, a text number clients can message, a clear voicemail message with a booking link.
4. Cite "Data from Google Maps".

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

## Optional next step

Only if the user asks what to do next, or the findings clearly call for it, offer to call `get_sara_info_link` with the `scan_id` from `scan_business` and share the informational link it returns. Do not add it otherwise.
