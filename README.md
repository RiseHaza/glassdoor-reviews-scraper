[Glassdoor Reviews Scraper](https://apify.com/sovereigntaylor/glassdoor-reviews-scraper?fpr=data)

The most comprehensive Glassdoor scraper on Apify. Goes far beyond basic review extraction to deliver deep company intelligence including employee reviews with sentiment analysis, detailed rating breakdowns (culture, compensation, management, work-life balance, career opportunities, diversity & inclusion), salary ranges by role and location, interview experiences with questions, benefits data, and company culture metrics.

## What This Scraper Extracts

### Company Profiles

- Company name, overall rating, total review count
- Recommend-to-friend rate, CEO approval rating, CEO name
- Industry, headquarters, company size, revenue, year founded
- Website, company type, mission statement, description
- **Rating breakdown**: Culture & Values, Diversity & Inclusion, Work-Life Balance, Senior Management, Compensation & Benefits, Career Opportunities
- Rating trends over time

### Employee Reviews (Deep)

- Review title, full text, pros, cons, advice to management
- Overall rating + **6-category rating breakdown** per review
- Author name, job title, location, employment status
- Current vs. former employee indicator
- Length of employment, helpful count, featured flag
- **Sentiment analysis** (positive / negative / mixed / neutral)
- Review date, pagination across all pages

### Salary Data

- Job title, median salary, base pay, total pay, additional pay
- Salary range (min-max), number of salary reports
- Currency detection (USD, GBP, EUR, CAD, AUD, JPY, INR, CHF)
- Pay period (yearly, monthly, hourly, weekly)
- Location and years of experience

### Interview Experiences

- Job title, difficulty rating, overall experience
- Outcome (got offer / no offer / declined)
- Application method, interview process description
- Interview questions asked
- Date, duration, location

### Benefits Data

- Overall benefits rating
- Category breakdown (health, retirement, perks, time off, etc.)
- Top highlighted benefits

### Company Culture

- Culture & values rating
- Company values list
- Diversity rating, work-life balance rating, management rating

## How to Use

### Input Options

**Company URLs** — Provide direct Glassdoor URLs (overview, reviews, or salary pages):

```
{
  "companyUrls": [
    "https://www.glassdoor.com/Overview/Working-at-Google-EI_IE9079.htm",
    "https://www.glassdoor.com/Reviews/Apple-Reviews-E1138.htm"
  ]
}
```

**Search Terms** — Search by company name (the actor finds the best match):

```
{
  "searchTerms": ["Google", "Microsoft", "Amazon"]
}
```

**Combined** — Use both for maximum coverage:

```
{
  "companyUrls": ["https://www.glassdoor.com/Overview/Working-at-Tesla-EI_IE43129.htm"],
  "searchTerms": ["Netflix", "Meta"],
  "maxReviews": 500,
  "includeSalaries": true
}
```

### Configuration

| Parameter | Type | Default | Description |
| --- | --- | --- | --- |
| `companyUrls` | array | `[]` | Direct Glassdoor company URLs |
| `searchTerms` | array | `[]` | Company names to search for |
| `maxReviews` | integer | `300` | Max reviews per company (0 = unlimited) |
| `includeSalaries` | boolean | `true` | Also scrape salary data |
| `proxy` | object | — | Proxy configuration (residential recommended) |

## Output Format

### Company Overview Record

```
{
  "type": "company_overview",
  "companyName": "Google",
  "overallRating": 4.3,
  "totalReviewCount": 32456,
  "recommendRate": "85%",
  "ceoApproval": "92%",
  "ceoName": "Sundar Pichai",
  "industry": "Internet & Web Services",
  "headquarters": "Mountain View, CA",
  "companySize": "10000+ Employees",
  "revenue": "$10+ billion (USD)",
  "founded": "1998",
  "website": "https://www.google.com",
  "ratingBreakdown": {
    "culture": 4.2,
    "diversityInclusion": 4.1,
    "workLifeBalance": 4.0,
    "seniorManagement": 3.8,
    "compensationBenefits": 4.5,
    "careerOpportunities": 4.1
  }
}
```

### Review Record

```
{
  "type": "review",
  "companyName": "Google",
  "title": "Great company, high expectations",
  "rating": 4,
  "text": "Working at Google is a dream for many engineers...",
  "pros": "Excellent benefits, smart colleagues, free food, cutting-edge projects",
  "cons": "High pressure, work-life balance can suffer, bureaucracy in large teams",
  "adviceToManagement": "Focus more on employee well-being and reduce meeting culture",
  "author": "Software Engineer",
  "jobTitle": "Senior Software Engineer",
  "location": "Mountain View, CA",
  "date": "2024-11-15",
  "isCurrentEmployee": true,
  "employmentStatus": "Current Employee",
  "lengthOfEmployment": "More than 3 years",
  "helpfulCount": 42,
  "ratingBreakdown": {
    "culture": 4,
    "workLifeBalance": 3,
    "seniorManagement": 4,
    "compensationBenefits": 5,
    "careerOpportunities": 4,
    "diversityInclusion": 4
  },
  "sentiment": "positive"
}
```

### Salary Record

```
{
  "type": "salary",
  "companyName": "Google",
  "jobTitle": "Software Engineer",
  "salary": "$165,000/yr",
  "medianPay": 165000,
  "basePay": 165000,
  "totalPay": 285000,
  "additionalPay": 120000,
  "range": "$130,000 - $220,000",
  "rangeMin": 130000,
  "rangeMax": 220000,
  "count": 1845,
  "currency": "USD",
  "payPeriod": "yr",
  "location": "Mountain View, CA"
}
```

### Interview Record

```
{
  "type": "interview",
  "companyName": "Google",
  "jobTitle": "Software Engineer",
  "difficulty": "Hard",
  "experience": "Positive",
  "outcome": "Accepted Offer",
  "applicationMethod": "Online Application",
  "interviewProcess": "Phone screen followed by 5 on-site interviews...",
  "questions": [
    "Design a URL shortener service",
    "Implement LRU cache",
    "System design: build a notification system"
  ],
  "date": "2024-10-01"
}
```

## Scraping Strategy

This actor uses a multi-layered extraction approach for maximum reliability:

1. ****NEXT_DATA** JSON** (primary) — Glassdoor is built on Next.js and embeds structured page data as JSON. This is the richest data source.
2. **Apollo Cache** (secondary) — GraphQL cache stored in `window.__APOLLO_STATE__` contains review and salary entities.
3. **JSON-LD** — Structured data for search engines, contains company and aggregate rating info.
4. **DOM Parsing** (fallback) — Cheerio-based HTML parsing for when JSON sources are unavailable.

### Anti-Detection Features

- 10+ rotating browser user-agent strings
- Random delays between requests (1.5-4 seconds)
- Referer header spoofing
- Session management and rotation
- Proxy support (residential proxies strongly recommended)

## Use Cases

- **HR Analytics** — Benchmark employer brand, track review sentiment over time
- **Competitive Intelligence** — Compare company ratings, culture, and compensation across competitors
- **Salary Benchmarking** — Build compensation databases by role, location, and company
- **Recruitment Research** — Understand interview processes and candidate experience
- **Employer Branding** — Monitor and improve your Glassdoor presence
- **Due Diligence** — Evaluate company culture before M&A or investment

## Tips for Best Results

1. **Use residential proxies** — Glassdoor blocks datacenter IPs aggressively. Residential proxies are strongly recommended.
2. **Start small** — Test with `maxReviews: 50` first to verify your proxy setup works.
3. **Rate limiting** — The actor has built-in delays. Do not run multiple instances simultaneously on the same proxy pool.
4. **Large companies** — For companies with 10,000+ reviews, set a reasonable `maxReviews` limit to control costs.

## Pricing

This actor uses Pay-Per-Event pricing:

- **$0.005 per review** scraped (company profiles, salaries, interviews, and benefits are included at no extra charge)

## Export Formats

Data can be exported from the Apify dataset in:

- JSON
- CSV
- Excel (XLSX)
- XML
- RSS

## Support

Built by Sovereign AI. For issues or feature requests, contact: [ricardo.yudi@gmail.com](mailto:ricardo.yudi@gmail.com)

## Legal Notice

This actor is provided for research and analytics purposes. Users are responsible for ensuring their use complies with Glassdoor's Terms of Service and applicable laws. The actor respects robots.txt and implements rate limiting to minimize server impact.

## Integration — Python

```
from apify_client import ApifyClient

client = ApifyClient("YOUR_API_TOKEN")
run = client.actor("sovereigntaylor/glassdoor-reviews-scraper").call(run_input={
    "searchTerm": "glassdoor reviews",
    "maxResults": 50
})

for item in client.dataset(run["defaultDatasetId"]).iterate_items():
    print(f"{item.get('title', item.get('name', 'N/A'))}")
```

## Integration — JavaScript

```
import { ApifyClient } from 'apify-client';
const client = new ApifyClient({ token: 'YOUR_API_TOKEN' });

const run = await client.actor('sovereigntaylor/glassdoor-reviews-scraper').call({
    searchTerm: 'glassdoor reviews',
    maxResults: 50
});

const { items } = await client.dataset(run.defaultDatasetId).listItems();
items.forEach(item => console.log(item.title || item.name || 'N/A'));
```