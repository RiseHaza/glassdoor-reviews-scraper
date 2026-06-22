[Glassdoor Reviews Scraper](https://apify.com/knotless_cadence/glassdoor-reviews-scraper?fpr=data)

# Glassdoor Reviews Scraper — Employee Reviews & Ratings in Clean JSON

**Get employee review text and overall ratings from any Glassdoor company page — without an account, with three CSS/HTML fallback parsers so the actor keeps working when Glassdoor re-skins.**

This actor turns a Glassdoor company URL (or a search keyword) into a structured dataset of reviews — title, rating, pros, cons, author, and date — pulled from public Glassdoor company pages. Pagination handled automatically.

## Who uses this

- **Talent-acquisition teams** benchmarking their employer brand against direct competitors before a recruiting push.
- **HR consultants** building workforce-sentiment reports for clients in a specific industry.
- **Market researchers** analyzing how companies in a sector are perceived internally.
- **Job seekers with tooling** who want to compare 20 companies side-by-side instead of clicking through each page.
- **Investment analysts** using employee sentiment as a leading indicator for operational health.

## What you get per review

Real fields actually pushed by the actor (verified against `src/main.js`):

| Field | Type | Example |
| --- | --- | --- |
| `companyName` | string | `"Stripe"` |
| `title` | string | `"Great place for strong engineers"` |
| `rating` | number / null | `4.5` |
| `pros` | text | `"Clear growth paths, thoughtful peers..."` |
| `cons` | text | `"Comp lag for senior ICs since 2024 freeze..."` |
| `text` | text | concatenated `pros + cons` |
| `author` | string | `"Senior Software Engineer"` (when surfaced by Glassdoor) |
| `datePublished` | string | `"2026-03-11"` |
| `scrapedAt` | string (ISO 8601) | `"2026-04-29T05:55:00.000Z"` |

**Up to 9 fields per review (Methods 1 & 2)**, **6 fields if Method 3 fires** (no `rating`, no `author`, no `datePublished` — embedded-JSON regex doesn't extract them). For JSON-LD path (Method 1), `pros` and `cons` are emitted as empty strings — Glassdoor's JSON-LD `Review` schema collapses pros/cons into a single `reviewBody` which lands in the `text` field instead.

No salary data, no interview data, no ratings breakdown by category — those are separate Glassdoor surfaces with their own anti-bot patterns; not in scope for this actor. If you need those, email and we can build a dedicated pipeline (see Custom scraping below).

## Input

```
{
  "companyUrls": [
    "https://www.glassdoor.com/Overview/Working-at-Stripe-EI_IE671932.htm",
    "https://www.glassdoor.com/Overview/Working-at-Notion-EI_IE3641702.htm"
  ],
  "searchQueries": ["fintech startups"],
  "maxReviewsPerCompany": 100
}
```

- `companyUrls` — list of Glassdoor company URLs (Overview or Reviews pages). The actor auto-converts Overview → Reviews and walks pagination.
- `searchQueries` — list of keyword searches. The actor parses the search results page, picks up to 10 distinct employer links per query, then collects reviews for each.
- `maxReviewsPerCompany` — per-company cap. **Default `30`.** Sensible upper bound around 200-500; the underlying `maxRequestsPerCrawl` is 500, so very large totals across many companies can hit that ceiling.

**Honest disclosure on parameters:**

- `sortBy` IS in `input_schema.json` (accepts `date` / `relevance`) and IS read by the actor — but the value is **NOT applied to any URL parameter** in the current implementation. Effectively it is dead at runtime; reviews come back in whatever order Glassdoor's default review page serves. We will wire this up in a future version, or in a custom build if you need it now.
- There is no `country` or `sortReviews` filter. If you want a specific country marketplace, pass that country's Glassdoor URL directly (`www.glassdoor.co.uk/...`, `www.glassdoor.de/...`, etc.).
- Apify proxy is requested via `Actor.createProxyConfiguration()`. On free Apify plans (where proxy isn't included) this resolves to `undefined` and the crawler falls back to **direct fetch** — Glassdoor anti-bot is more likely to fire in that mode. For reliable runs at scale, use a paid Apify plan with proxy enabled, or commission a custom build with residential-proxy routing.
- The actor does NOT proactively detect anti-bot challenges (CAPTCHA, "verify you are human", session-block). If all 3 extraction methods return 0 reviews for a URL, that is the silent indicator — re-run after 30-60 minutes from a fresh IP.

## Python usage example

```
from apify_client import ApifyClient
import statistics

client = ApifyClient("YOUR_APIFY_TOKEN")

run_input = {
    "companyUrls": [
        "https://www.glassdoor.com/Overview/Working-at-Stripe-EI_IE671932.htm",
    ],
    "maxReviewsPerCompany": 100,
}

run = client.actor("knotless_cadence/glassdoor-reviews-scraper").call(run_input=run_input)
reviews = list(client.dataset(run["defaultDatasetId"]).iterate_items())

# Average rating + spread
ratings = [r["rating"] for r in reviews if r.get("rating") is not None]
if ratings:
    print(f"Avg rating: {statistics.mean(ratings):.2f} ({len(ratings)} reviews)")
    if len(ratings) > 1:
        print(f"Median: {statistics.median(ratings):.2f}, stdev: {statistics.stdev(ratings):.2f}")

# Quick keyword scan in pros/cons
for kw in ("layoff", "comp", "burnout", "growth"):
    hits = [r for r in reviews if kw.lower() in (r.get("text") or "").lower()]
    print(f"{kw:>8}: {len(hits)} reviews mention it")
```

## How it works (3 extraction methods)

Glassdoor renders reviews three ways depending on session, country, and anti-bot state. The actor tries all three in order:

1. **JSON-LD `Review`/`EmployerReview` blocks** — fastest path, used when Glassdoor inlines structured data.
2. **HTML review cards** — `[data-test="review-card"]`, `.gdReview`, `.empReview` selectors with `pros`/`cons`/`rating` parsers.
3. **Embedded JSON in `<script>` tags** — regex match on `reviewTitle` + `pros` + `cons` patterns inside Apollo state / `__NEXT_DATA__`.

Method 1 fails → fall through to 2 → fall through to 3. If all three return empty, the page likely served an anti-bot challenge — re-run after 30-60 minutes.

Apify proxy is used when available (`Actor.createProxyConfiguration()`); falls back to direct fetch if the proxy SDK isn't configured. `maxConcurrency=2`, `requestHandlerTimeoutSecs=30`, `maxRequestRetries=3`.

## Common questions

**Q: Do I need a Glassdoor account?**
No. The actor reads publicly visible review pages.

**Q: Does it pull salary or interview data?**
Not in this actor. Salary and interview surfaces have separate anti-bot patterns and a different schema. Available as a custom build — see Custom scraping below.

**Q: How often do reviews refresh?**
Live at run time. Running daily gives you reliable new-review deltas.

**Q: How do I turn raw reviews into actionable insight?**
Common patterns: (1) keyword scan on pros/cons to find recurring themes, (2) rating trend by month to detect culture shifts, (3) date-cohort analysis (e.g. reviews from the 90 days after a re-org), (4) cross-company comparison within a sector.

**Q: Is it compliant with Glassdoor's terms?**
This actor accesses publicly visible pages only, at rate-limited pace (`maxConcurrency=2`). For commercial redistribution of reviews, check Glassdoor's data-use terms with legal counsel. For internal market-research use, public-data precedent (hiQ v. LinkedIn, 9th Circuit 2022) generally applies.

**Q: Why might a run return fewer reviews than expected?**
(a) Glassdoor served an anti-bot challenge — all three extraction methods return empty; re-run after 30-60 min. (b) Page count exceeded `maxRequestsPerCrawl=500`; lower the per-company cap or split the run. (c) Review cards changed CSS class names again — the JSON-LD and embedded-JSON fallbacks usually catch this, but report a broken URL via email and we'll patch the selectors.

## Export integrations

- CSV / JSON / Excel / HTML (native Apify dataset download)
- Google Sheets (via Apify integration)
- Webhooks (fire on each review batch)
- S3 / GCS direct sync
- Zapier / Make.com

## Related scrapers

| Source | Actor | Data |
| --- | --- | --- |
| Glassdoor (this) | Reviews | Workforce sentiment |
| [Trustpilot](https://apify.com/knotless_cadence/trustpilot-review-scraper) | Customer reviews | External brand sentiment |
| [Reddit Discussion](https://apify.com/knotless_cadence/reddit-discussion-scraper) | Discussion threads | Unfiltered opinion |
| [Google News](https://apify.com/knotless_cadence/google-news-scraper) | News articles | Layoff / re-org / leadership signals |
| [MCP Company Researcher](https://apify.com/knotless_cadence/mcp-company-researcher) | Company facts | Headcount, funding, profile |

All 31 published actors free to inspect on [Apify Store](https://apify.com/store?search=knotless_cadence).

---

**Proof of delivery**: This Glassdoor reviews scraper has **39 lifetime production runs** as of May 2026. Author maintains 31 published actors (78 total) and shipped a paid 3-article series in March 2026 ($150, proxy industry). Pilot pricing locked through **May 2026**.

**Sample request?** Reply `sample` to [spinov001@gmail.com](mailto:spinov001@gmail.com) and we'll send 2 published case-study articles within 24 hours.

## Custom scraping — pilot tiers

Need data, not infrastructure. We build, you query. Three tiers:

- **Pilot — $97** · 1 actor, basic config, 7-day support. Good entry point — useful for a single Glassdoor competitor benchmark or a one-off salary surface.
- **Standard — $297** · custom actor + Slack/email alerts on results, 30-day support. Most workforce-analytics projects fit here.
- **Premium — $797** · custom actor + dashboard + 90-day support + 1 modification round. For ongoing pipelines (monthly competitor sentiment, multi-source enrichment with LinkedIn + Blind + Reddit).

**Email:** [spinov001@gmail.com](mailto:spinov001@gmail.com) — drop specs, schema, or target URLs and get a quote within 48h.

**Proof of work:** [31 published Apify scrapers](https://apify.com/knotless_cadence) (78 total in portfolio) — Trustpilot 951 runs, Reddit 82, Google News 45, Glassdoor 39, Email Extractor 107. Recently delivered a paid 3-article series for a client in the proxy industry ($150).

**More tips:** [t.me/scraping_ai](https://t.me/scraping_ai) · [blog.spinov.online](https://blog.spinov.online)

---

## Disclaimer

Designed for market-research, HR-intelligence, and academic use. Respect Glassdoor's Terms of Service, applicable data-protection law (GDPR, CCPA), and scrape publicly visible content only. Not affiliated with Glassdoor, Inc.

*Honest disclosure: this scraper covers public Glassdoor company-page reviews only — no login-walled content, no salary surface, no interview surface. No `country` filter; `sortBy` is in the schema but currently dead at runtime (see Honest disclosure above). Pass a country-localized URL if needed. For any of those, request a custom build.*