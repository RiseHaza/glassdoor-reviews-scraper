[Glassdoor Reviews Scraper](https://apify.com/kaix/glassdoor-reviews-scraper?fpr=data)

# Glassdoor Reviews Scraper

Scrape every review from any Glassdoor company page. ~10,000 reviews per minute. Structured JSON output with 10 rating categories, employment details, translations, and employer responses. Collects reviews in all languages.

Optionally fetches comprehensive company data: profile, aggregate ratings, demographic breakdowns, office locations, benefits, photos, and more.

Just paste a Glassdoor company reviews URL and hit Run.

## What you get

### Reviews

Every review includes:

- **Review text**: summary, pros, cons, advice to management
- **10 ratings**: overall, work-life balance, culture, leadership, compensation, career opportunities, diversity, CEO approval, business outlook, recommend to friend
- **Employment**: current/former, job title, tenure, employment status
- **Location**: city and region
- **Engagement**: helpful votes, featured status
- **Translations**: original text for non-English reviews
- **Employer responses**: company replies with job title and timestamps
- **Meta**: review detail URL, division info, language mismatch flag

### Company data

One consolidated object per employer with:

- **Profile**: name, headquarters, revenue, size, website, type, year founded, industry/sector, logos, CEO
- **Aggregate ratings**: overall + 11 sub-ratings with CEO approval and recommend-to-friend ratios
- **Rating distributions**: star breakdown (1-5) for 7 categories + recommend distribution
- **Demographic ratings**: ratings broken down by gender, race/ethnicity, sexual orientation, disability, parent/caregiver, veteran status
- **Review locations**: flat list of all cities/metros where reviews were submitted, with state and country
- **Office locations**: full addresses with postal codes, country/continent, and per-office overall rating
- **Benefits**: 50+ individual benefits with category, rating, review count, and verified status
- **Photos**: company photos with captions
- **Related companies**: parent, subsidiaries, siblings with ratings and counts
- **Awards**: best places to work, best-led companies

## Quick start

Paste a Glassdoor company reviews URL:

```
{
  "urls": ["https://www.glassdoor.com/Reviews/Google-Reviews-E9079.htm"]
}
```

That's it. Default: 20 most recent reviews + full company data.

### More examples

**Multiple companies (200 reviews each):**

```
{
  "urls": [
    "https://www.glassdoor.com/Reviews/Google-Reviews-E9079.htm",
    "https://www.glassdoor.com/Reviews/Meta-Reviews-E40772.htm"
  ],
  "maxReviews": 200
}
```

**Reviews only (skip company data):**

```
{
  "urls": ["https://www.glassdoor.com/Reviews/Google-Reviews-E9079.htm"],
  "includeCompanyData": false
}
```

**All reviews for a company:**

```
{
  "urls": ["https://www.glassdoor.com/Reviews/Google-Reviews-E9079.htm"],
  "maxReviews": 0
}
```

## Input

| Parameter | Type | Default | Description |
| --- | --- | --- | --- |
| **urls** | string[] | required | Glassdoor company reviews URLs (e.g. `glassdoor.com/Reviews/Google-Reviews-E9079.htm`) |
| **maxReviews** | number | 20 | Max reviews per company. Set to 0 for no limit (all reviews). |
| **sort** | select | DATE | `DATE` (newest first) or `RATING` (highest rated first) |
| **includeCompanyData** | boolean | true | Fetch company profile, ratings, demographics, locations, benefits, photos |

## Output

### Reviews dataset (default)

Each review is a single JSON object:

```
{
  "reviewId": 103260164,
  "url": "https://www.glassdoor.com/Reviews/Employee-Review-Nucor-RVW103260164.htm",
  "reviewDateTime": "2026-03-21T23:47:58.430",
  "summary": "Not a good company",
  "pros": "Good benefits and pay",
  "cons": "Long hours, limited growth",
  "advice": "Improve work-life balance",
  "ratings": {
    "overall": 4,
    "workLifeBalance": 3,
    "cultureAndValues": 4,
    "seniorLeadership": 3,
    "compensationAndBenefits": 5,
    "careerOpportunities": 4,
    "diversityAndInclusion": 4,
    "ceo": "APPROVE",
    "businessOutlook": "POSITIVE",
    "recommendToFriend": "POSITIVE"
  },
  "employment": {
    "isCurrentJob": true,
    "status": "REGULAR",
    "lengthOfEmployment": 36,
    "jobEndingYear": null,
    "jobTitle": { "id": 61331, "text": "Software Engineer" }
  },
  "location": { "id": 1127429, "name": "Birmingham, AL", "type": "CITY" },
  "engagement": { "countHelpful": 10, "countNotHelpful": 2, "featured": false },
  "translation": {
    "languageId": "eng",
    "originalLanguageId": null,
    "summaryOriginal": null,
    "prosOriginal": null,
    "consOriginal": null,
    "adviceOriginal": null
  },
  "hasEmployerResponse": false,
  "employer": { "id": 489, "shortName": "Nucor", "name": "Nucor Corporation" },
  "employerResponses": [],
  "meta": {
    "isLegal": true,
    "isCovid19": false,
    "translationMethod": null,
    "flaggingDisabled": null,
    "relatedStructures": [],
    "isLanguageMismatch": false,
    "reviewDetailUrl": "/Reviews/Employee-Review--RVW103260164.htm",
    "divisionName": null,
    "divisionLink": null,
    "topLevelDomainId": 1,
    "scrapedAt": "2026-03-30T10:00:00.000Z"
  }
}
```

### Company dataset (COMPANY)

One object per employer. Available via the named dataset `COMPANY` in Apify console or API.

```
{
  "id": 489,
  "name": "Nucor Corporation",
  "shortName": "Nucor",
  "headquarters": "Charlotte, NC",
  "revenue": "$10+ billion (USD)",
  "size": "10000+ Employees",
  "sizeCategory": "GIANT",
  "website": "www.nucor.com",
  "type": "Company - Public",
  "yearFounded": 1971,
  "primaryIndustry": {
    "industryId": 200074,
    "industryName": "Metal & Mineral Manufacturing",
    "sectorId": 10015,
    "sectorName": "Manufacturing"
  },
  "overview": {
    "description": "Nucor continues to electrify the steel industry...",
    "mission": "Our challenge is to become the world's safest steel company..."
  },
  "ceo": { "id": 942368, "name": "Leon Topalian", "title": "CEO" },
  "ratings": {
    "overallRating": 4,
    "ceoRating": 0.91,
    "ceoRatingsCount": 275,
    "recommendToFriendRating": 0.8,
    "businessOutlookRating": 0.77,
    "cultureAndValuesRating": 3.9,
    "careerOpportunitiesRating": 3.8,
    "workLifeBalanceRating": 3.2,
    "seniorManagementRating": 3.6,
    "compensationAndBenefitsRating": 4,
    "diversityAndInclusionRating": 3.7,
    "ratedCeo": { "id": 942368, "name": "Leon Topalian", "title": "CEO" }
  },
  "ratingCountDistribution": {
    "overall": { "1": 48, "2": 59, "3": 128, "4": 218, "5": 397 },
    "cultureAndValues": { "1": 52, "2": 49, "3": 79, "4": 137, "5": 358 },
    "recommendToFriend": { "RECOMMEND": 494, "WONT_RECOMMEND": 133 }
  },
  "demographicRatings": [
    {
      "category": "gender",
      "categoryRatings": [
        { "categoryValue": "man", "ratings": { "overallRating": 3.9, "reviewCount": 141 } },
        { "categoryValue": "woman", "ratings": { "overallRating": 3.9, "reviewCount": 34 } }
      ]
    }
  ],
  "reviewLocations": [
    { "id": 93, "name": "Birmingham", "type": "METRO", "state": "Alabama", "country": "United States" }
  ],
  "officeLocations": [
    {
      "id": 1088,
      "addressLine1": "701 Bank St Ne",
      "cityName": "Decatur",
      "administrativeAreaName1": "AL",
      "countryName": "US",
      "postalCode": "35601-1609",
      "employerReviews": { "ratings": { "overallRating": 4.3 } }
    }
  ],
  "benefitsOverallRating": 4.2,
  "benefitsTotalReviews": 166,
  "benefits": [
    { "category": "Insurance, Health & Wellness", "name": "Health Insurance", "rating": 3.6, "reviewCount": 33, "verified": true }
  ],
  "benefitCountries": [
    { "id": 1, "name": "United States" }
  ],
  "subsidiaries": [
    { "employer": { "id": 449662, "name": "Harris Rebar", "ratings": { "overallRating": 3.6 } } }
  ],
  "photos": [
    { "caption": "At work", "photoUrl2x": "https://media.glassdoor.com/..." }
  ],
  "counts": { "reviewCount": 850, "salaryCount": 1335, "interviewCount": 199 },
  "scrapedAt": "2026-03-30T13:26:34.106Z"
}
```

*Fields truncated for readability. Actual output includes all sub-ratings in demographic breakdowns, all office locations, all benefits, etc.*

## Performance

- **~10,000 reviews per minute**
- **1 GB default memory** — keeps platform costs low
- Handles pagination automatically — just set `maxReviews`
- Gracefully handles rate limiting — returns whatever was collected
- Company data adds ~2-3 seconds per employer (6 parallel queries)

## Limitations

- `maxReviews` is per company, not total across all URLs
- Some fields may be null for older reviews
- Reviews in all languages are collected; use the `translation.languageId` field to filter by language
- Benefits data defaults to US; other countries available in `benefitCountries`