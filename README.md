# Beauty & Wellness Biz Grader

A checkup for appointment-based beauty and wellness businesses in the United States: hair salons, nail salons, lash and brow studios, waxing and laser hair removal, massage and spas, med spas, aesthetic clinics, cosmetic surgery practices and longevity clinics. Dental practices and businesses without appointments are not supported.

You give the business name and city. The plugin looks up that single business on Google Maps and returns a score for its Google Business Profile, reputation, local competitors, website and online booking signals, plus an estimated range of missed bookings with the assumptions shown. The estimate is a planning figure, not a measurement.

## What it connects to

One remote MCP server, `https://mcp.holasara.ai/mcp`, operated by BotBit Inc. It needs no login and no API key, and every tool is read only. Nothing in this plugin runs code on your machine, uses hooks, or reads your environment.

## Tools

- `find_business`: finds up to five candidates by name and city.
- `scan_business`: scans one business and returns the score, gaps, reviews, competitors and booking signals.
- `compare_competitors`: compares the scanned business with nearby competitors.
- `estimate_missed_bookings`: recalculates the missed bookings range with your own assumptions.
- `get_sara_info_link`: returns an informational link about Sara, a booking assistant, for the scan you just ran.

## Skills

Ten skills guide the checkup: missed bookings audit, Google profile checkup, review health and reply drafts, local competitor snapshot, online booking and website check, after hours coverage, med spa treatment visibility, cosmetic surgery consult funnel, new location scout and weekly monitor info.

## Data and privacy

The plugin uses only public information from Google Maps and the business website. Please do not enter patient or customer data. Scan results are cached for up to seven days, searches for 24 hours, and technical logs are kept up to 30 days. Details: https://holasara.ai/en/privacy#plugin

## Support

Questions or requests: support@rayapp.io. Terms: https://holasara.ai/en/terms#plugin. More information: https://holasara.ai/en/chatgpt
