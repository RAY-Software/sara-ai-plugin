---
name: local-competitor-snapshot
description: Use when someone asks where their business ranks on Google Maps for its category and who is ahead of them nearby. Chains scan_business and compare_competitors.
---

# Local Competitor Snapshot

Answers: "Where do I stand locally and who beats me, and why?"

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

## Step 2: scan and compare

Call `scan_business`, then `compare_competitors` with the same `place_id`. If the user gave a `vertical` to `scan_business`, pass the same one to `compare_competitors`. Read `business_rank`, `competitor_query`, `radius_meters` and the competitors list (`rank`, `name`, `rating`, `review_count`, `rating_delta`, `review_count_delta`).

## Step 3: narrate

1. Where the business ranks for the search term (`competitor_query`) and within what radius.
2. A table of the top competitors: rank, name, rating, reviews, difference versus the user's business.
3. Why the leaders are ahead: usually more reviews, higher rating, or both. Use only the deltas returned; do not guess other reasons.
4. Two or three realistic steps to close the biggest gap.
5. Cite "Data from Google Maps".

Competitors are always relative to the scanned business, at most 10. Do not turn this into a market wide list.

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
