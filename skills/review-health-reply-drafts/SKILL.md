---
name: review-health-reply-drafts
description: Use when someone wants to understand their rating, review volume and recency compared with local competitors, or wants drafted replies to reviews they paste. Includes a health mode for med spa, cosmetic surgery and longevity clinics.
---

# Review Health and Reply Drafts

Answers: "How healthy are my reviews and how should I reply?"

There is no tool that fetches review text. Use the rating, volume and competitor comparison from the scan, and write reply drafts only from review text that the user pastes into the chat.

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

Call `scan_business`, then `compare_competitors` with the same `place_id`. Read `business.rating`, `business.review_count`, the `reputation` factor and its signals (rating, volume, recency when known), and each competitor's `rating_delta` and `review_count_delta`.

## Step 3: narrate review health

1. Rating and review count versus the competitors: who is ahead on each and by how much.
2. Recency: report it only if a reputation signal has `known: true`. Otherwise say recency could not be checked.
3. One or two practical suggestions (for example ask satisfied clients for a review right after a visit, reply to every review within a few days).
4. Cite "Data from Google Maps".

## Step 4: reply drafts (only for pasted reviews)

If the user pastes a review, write a short, warm, professional reply. Keep it to three or four sentences, thank the reviewer, address any concern in general terms and offer to continue by phone or email. Do not invent facts that are not in the pasted text. Ask the user to review before posting; you do not post anything.

### HEALTH MODE (med spa, cosmetic surgery, longevity, IV therapy, any clinical service)

Turn this on when the `vertical` is `med_spa`, `plastic_surgery` or `longevity`, or the review mentions a treatment or a condition. In health mode:
- Never confirm or imply that the reviewer is a patient or client. Do not write "thank you for visiting us" or "we are glad your procedure went well".
- Never name a treatment, procedure, product, condition or outcome, even if the reviewer did.
- Do not repeat any detail from the review (dates, symptoms, staff names, results).
- Use a generic reply: thank them for the feedback, say the team takes feedback seriously, and invite them to contact the business through a private channel (a phone number or email the owner supplies) to talk further.
- Example shape: "Thank you for taking the time to share your feedback. We take every comment seriously. Please contact us directly at [phone or email] so we can talk privately."
- Remind the owner to have their compliance contact review replies before posting.

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
