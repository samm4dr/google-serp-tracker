# Track Your Google Rankings Without Paying $100/Month for a SaaS

Rank trackers charge $50-150/month for what's essentially a Google query on a schedule. Here's the same thing in 30 lines of Python + an Apify actor — pay per SERP, own the pipeline.

> 🤖 **Try it live** — chat with a Poe bot that generates the exact pipeline for your keywords:
> **[SERP Rank Tracker](https://poe.com/serpranktracker)** · [Lead List Generator](https://poe.com/leadlistgenerator) · [Job Postings Miner](https://poe.com/jobpostingsminer) · [Amazon Product Miner](https://poe.com/amazonproductminer) · [Social Lead Finder](https://poe.com/socialleadfinder)

## The stack

- **[Apify](https://console.apify.com/sign-up?fpr=dx06p)** — `apify/google-search-scraper` returns clean SERP JSON. Free monthly credits.
- **Your keywords** — as many as you want, any country/language.

## Step 1 — Token

[Create an Apify account](https://console.apify.com/sign-up?fpr=dx06p) → Settings → Integrations → `export APIFY_TOKEN=...`

## Step 2 — Query the SERPs

```bash
curl -X POST "https://api.apify.com/v2/acts/apify~google-search-scraper/run-sync-get-dataset-items?token=$APIFY_TOKEN" \
  -H "Content-Type: application/json" \
  -d '{
    "queries": "best standing desk\nergonomic chair review\nmotorized desk",
    "maxPagesPerQuery": 2,
    "countryCode": "us",
    "languageCode": "en",
    "mobileResults": false
  }'
```

## Step 3 — Extract your positions

```python
from apify_client import ApifyClient

MY_DOMAIN = "yoursite.com"
client = ApifyClient("APIFY_TOKEN")

run = client.actor("apify/google-search-scraper").call(run_input={
    "queries": "\n".join(["keyword one", "keyword two", "keyword three"]),
    "maxPagesPerQuery": 2,
    "countryCode": "us",
})

for item in client.dataset(run["defaultDatasetId"]).iterate_items():
    term = item["searchQuery"]["term"]
    for r in item.get("organicResults", []):
        if MY_DOMAIN in r.get("url", ""):
            print(f"{term} -> position {r['position']} ({r['url']})")
```

Schedule it weekly → diff the datasets → rank movement report, for free.

## Bonus: keyword research combo

Feed seed keywords into `apify/google-autocomplete-scraper` → hundreds of long-tail suggestions → re-run through the SERP scraper to gauge competition. Poor man's Ahrefs that actually scales.

## Local SEO

`compass/google-maps-scraper` per city audits map-pack presence — who ranks, ratings, review counts. Combine with SERP positions for the full picture.

## Watch your spend

1 query = 1 SERP page billed. `maxPagesPerQuery: 1-2` covers position tracking; go deeper only for competitor research.

Full tooling list: [awesome-web-scraping](https://github.com/g4dr/awesome-web-scraping). Claude Code/Cursor users: `npx skills add g4dr/apify-skills` has an `apify-seo-serp` skill that runs all this from a prompt ([repo](https://github.com/g4dr/apify-skills)).

*What keywords are you tracking?*

---

## Resources

- [Apify](https://console.apify.com/sign-up?fpr=dx06p) — the scraping platform used in this tutorial (free credits on signup)
- [g4dr/apify-skills](https://github.com/g4dr/apify-skills) — Agent Skills that run these pipelines from a prompt
- [g4dr/awesome-web-scraping](https://github.com/g4dr/awesome-web-scraping) — curated list of scraping tools, proxies and actors
