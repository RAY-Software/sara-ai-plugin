---
name: online-booking-website-check
description: Use when someone wants to know whether clients can book online from a phone and whether the website offers text or chat. Uses the website signals returned by scan_business.
---

# Online Booking and Website Check

Answers: "Can a client book from their phone in two taps, and can they text or chat?"

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

Call `scan_business`. Read `data.website`: `checked`, `booking_providers`, `chat_providers`, `text_channels` (`sms`, `whatsapp`) and `tap_to_call`, plus the `booking_readiness` factor and its signals.

## Step 3: narrate

1. If `website.checked` is false, say the website could not be checked (no website listed or it was unreachable) and that this alone limits booking.
2. Online booking: name the provider(s) detected, or say none was detected. Say "detected" rather than "does not exist", since a booking tool can be hidden behind a script.
3. Text and chat: which channels were detected, and whether a tap to call link exists.
4. Mention a detected booking provider neutrally. Never suggest the owner should switch providers.
5. Give a short fix list ordered by missing points in `booking_readiness`.
6. Cite "Data from Google Maps".

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
