---
name: missed-bookings-audit
description: Use when an owner of a salon, spa, med spa or clinic asks how many appointments they may be missing from unanswered calls and messages. Chains find_business, scan_business and estimate_missed_bookings and presents a range with assumptions.
---

# Missed Bookings Audit

Answers: "How many appointments am I likely losing each month?"

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

Call `scan_business` with the `place_id`. Pass `relationship` (owner, staff, agency, other) only if the user said it. Read `data.missed_bookings` (`monthly.missedContacts`, `monthly.missedBookings`, `monthly.lostRevenueUsd`, each with low, mid, high), `assumptions` and `observations`.

## Step 3: refine with the user's own figures (optional)

Ask once, in one message, whether they know their average ticket (revenue per booking), calls per day, and the share of calls staff answer. If they give any, call `estimate_missed_bookings` with the same `place_id` and `avg_ticket`, `calls_per_day`, `staff_answers_phone` (0 to 100). Compare the new range with the first one. This call does not query Google again.

## Step 4: narrate

1. One sentence with the range of missed bookings per month (low to high) and the mid value as the central guess. Call it an estimate.
2. A small table of the assumptions, marking which came from the user and which are vertical defaults.
3. The two or three observations that drive the estimate (for example no online booking detected, no text channel, limited evening hours).
4. Up to three actions the owner can take alone, taken from `next_steps` where `who` is `owner`.
5. Cite "Data from Google Maps".

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
