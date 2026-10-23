---
name: cosmetic-surgery-consult-funnel
description: Use when a plastic or cosmetic surgery practice wants to assess how easily a prospective patient can book a consultation compared with nearby surgeons. Uses scan_business and compare_competitors.
---

# Cosmetic Surgery Consult Funnel

Answers: "How easy is it to book a consultation with us compared with other surgeons nearby?"

The conversion unit is the consultation, not the procedure. The tools do not measure response time or see inquiries. Say so and treat any response time discussion as a question for the owner.

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

Call `scan_business`, then `compare_competitors` with the same `place_id`. Read `booking_readiness`, `website` (booking providers, text channels, tap to call), reputation versus the leaders, and the missed bookings range (which here reads as consultation requests, with the average ticket reflecting the consultation).

## Step 3: narrate

1. Can a prospective patient book a consultation online, text, or tap to call? Use the website signals.
2. Presence and reputation versus nearby surgeons: rank, rating, review volume.
3. The missed consultation range as an ESTIMATE with assumptions. If the user knows their consultation fee or case value, rerun `estimate_missed_bookings` with `avg_ticket`.
4. Questions for the owner about how fast inquiries get a reply.
5. Cite "Data from Google Maps".

## Before and after photos reminder

If the user plans to use before and after photos on the profile or website, remind them: get written consent, follow their state medical board and advertising rules, avoid guarantees or promised results, and keep claims truthful. This is a reminder, not legal advice.

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
