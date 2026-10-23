---
name: new-location-scout
description: Use when an existing business owner wants to understand competition around a possible new location. Requires the user's own business as an anchor and works one business at a time; it is not a tool for listing businesses in bulk.
---

# New Location Scout

Answers: "How crowded and how strong is the competition around the place I am considering?"

This skill needs an anchor: a business the user owns or runs. It compares the anchor with its nearby competitors to give a sense of local competition. It does not list businesses across a city, and it handles one business per request. If the user has no business of their own, or asks for a list of all businesses in an area, explain that this is not supported and stop.

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

Call `scan_business` on the anchor, then `compare_competitors`. Read `competitor_query`, `radius_meters`, the competitors (at most 10) and their ratings and review counts.

## Step 3: narrate

1. How many competitors appear within the radius and how strong they are (ratings and review counts).
2. How the anchor sits among them, as a reference for what a similar business achieves in that kind of area.
3. What to check next about the new address: compare it by running the same checkup with a nearby business the user knows, rather than listing neighbors.
4. State plainly that the data shows Google Maps competitors only, not rent, foot traffic or demand.
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
