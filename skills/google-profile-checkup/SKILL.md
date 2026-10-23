---
name: google-profile-checkup
description: Use when someone wants to know what is missing or weak on a business Google Business Profile and in which order to fix it. Uses scan_business factors and signals.
---

# Google Profile Checkup

Answers: "What is wrong with my Google profile and what should I fix first?"

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

Call `scan_business` with the `place_id`. Read `data.factors`, especially the factor with `key` `google_profile` and its `signals` (each has `name`, `points`, `maxPoints`, `value`, `known`), plus `business.phone_listed`, `business.website_listed`, `score` and `coverage`.

## Step 3: narrate

1. Overall score and the status of each factor in one line each.
2. A prioritized fix list for the profile: sort signals by missing points (`maxPoints - points`), largest first. Skip signals where `known` is false and say they could not be checked.
3. For each fix, say what to change in plain words (for example add a website link, list a phone number, add hours, add more photos, pick the right primary category).
4. If `degraded` lists a source, mention that part of the data was unavailable.
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
