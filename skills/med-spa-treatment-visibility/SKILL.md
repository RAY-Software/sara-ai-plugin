---
name: med-spa-treatment-visibility
description: Use when a med spa or aesthetic clinic wants to know whether treatments such as injectables, fillers, laser and skin treatments are visible on their Google profile and website compared with nearby clinics. Uses scan_business and compare_competitors.
---

# Med Spa Treatment Visibility

Answers: "Do people searching for my treatments find me, and do competitors show them better?"

The tools return profile, reputation, competitor and website signals. They do not return the list of treatments on a profile or site. Be explicit about that limit and ask the user to paste their service list or profile description if they want a content review.

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

Call `scan_business` (pass `vertical` as `med_spa` only if the user confirms it), then `compare_competitors`. Read `vertical`, `vertical_confidence`, `competitor_query`, the `google_profile` and `local_visibility` factors, the competitor table and `website`.

## Step 3: narrate

1. Local visibility: rank for the search term and how it compares with nearby clinics.
2. Profile completeness signals that affect treatment discovery: primary category, website link, photos, hours.
3. If the user pasted their treatment list or profile text, check it: are the main treatments named in plain words, with one clear booking path? Suggest wording gaps. Do not claim results or outcomes for any treatment.
4. Compare ratings and review volume with the leaders.
5. Cite "Data from Google Maps".

Health reminder: do not draft claims about results or safety. Remind the owner that medical advertising rules vary by state.

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
