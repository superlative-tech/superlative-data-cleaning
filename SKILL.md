---
name: superlative-data-cleaning
description: Clean and normalize messy data using Superlative's Apify actors. Handles company names, job titles, person names, product names, places, URLs, phone numbers, plus DNS lookups, WHOIS, domain health scoring, and ICP lead scoring. Use when cleaning scraped data, normalizing CRM exports, or building data pipelines.
license: MIT
metadata:
  author: superlative-tech
  version: "1.0"
---

# Superlative Data Cleaning

Clean messy data at scale using Apify actors. All actors run on [Apify](https://apify.com) and support batch processing and instant single-item API calls.

## Authentication

All actors require an Apify API token. Set the `APIFY_TOKEN` environment variable or pass `token` as a query parameter for Standby API calls.

## Actor Catalog

| Actor | Actor ID | What It Does | Cost/1K | Type |
|-------|----------|-------------|---------|------|
| [Company Names](https://apify.com/superlativetech/superclean-company-names) | `superlativetech/superclean-company-names` | Normalize company names (legal suffixes, ALL CAPS, brand formatting) | $1.00 | LLM |
| [Job Titles](https://apify.com/superlativetech/superclean-job-titles) | `superlativetech/superclean-job-titles` | Normalize job titles (abbreviations, levels, team names) | $1.00 | LLM |
| [Person Names](https://apify.com/superlativetech/superclean-person-names) | `superlativetech/superclean-person-names` | Normalize person names (ALL CAPS, titles, credentials, McDonald/O'Brien) | $1.00 | LLM |
| [Product Names](https://apify.com/superlativetech/superclean-product-names) | `superlativetech/superclean-product-names` | Clean product names (remove promo text, prices, SKUs) | $1.00 | LLM |
| [Places](https://apify.com/superlativetech/superclean-places) | `superlativetech/superclean-places` | Parse locations into city/state/country fields | $0.50 | Rule |
| [URLs](https://apify.com/superlativetech/superclean-urls) | `superlativetech/superclean-urls` | Clean URLs (remove tracking params, normalize format) | $0.50 | Rule |
| [Phone Numbers](https://apify.com/superlativetech/superclean-phone-numbers) | `superlativetech/superclean-phone-numbers` | Clean phone numbers (E.164 format, validate, detect type) | $0.50 | Rule |
| [DNS Lookup](https://apify.com/superlativetech/dns-lookup) | `superlativetech/dns-lookup` | Look up DNS records (MX, SPF, DMARC, A, AAAA, etc.) | $0.10 | DNS |
| [Domain Health](https://apify.com/superlativetech/supernet-domain-health) | `superlativetech/supernet-domain-health` | Score email domain health 0-100 (SPF, DKIM, DMARC, blacklists) | $2.50/$5.00 | DNS |
| [WHOIS Lookup](https://apify.com/superlativetech/supernet-whois-lookup) | `superlativetech/supernet-whois-lookup` | Look up WHOIS registration data (registrar, dates, nameservers) | $0.50 | WHOIS |
| [HTTP API](https://apify.com/superlativetech/http-api) | `superlativetech/http-api` | Make HTTP requests from Apify (GET/POST/PUT/DELETE) | $0.20 | Utility |
| [OpenRouter BYOK](https://apify.com/superlativetech/open-router) | `superlativetech/open-router` | Route LLM requests through OpenRouter with your own API key | $0.20 | LLM Proxy |
| [ICP Scorer](https://apify.com/superlativetech/superlead-icp-scorer) | `superlativetech/superlead-icp-scorer` | Score leads against your Ideal Customer Profile using AI | $1.00 | LLM |

## Quick Start

### Node.js (batch)

```javascript
import { ApifyClient } from 'apify-client';
const client = new ApifyClient({ token: process.env.APIFY_TOKEN });

const run = await client.actor('superlativetech/superclean-company-names').call({
  items: ['ACME CORPORATION, INC.', 'GOOGLE LLC', 'walmart inc'],
  style: 'casual'
});
const { items } = await client.dataset(run.defaultDatasetId).listItems();
// [{ id: 1, input: 'ACME CORPORATION, INC.', output: 'Acme', confidence: 0.95 }, ...]
```

### Python (batch)

```python
from apify_client import ApifyClient
client = ApifyClient(token=os.environ['APIFY_TOKEN'])

run = client.actor('superlativetech/superclean-company-names').call(run_input={
    'items': ['ACME CORPORATION, INC.', 'GOOGLE LLC', 'walmart inc'],
    'style': 'casual'
})
items = client.dataset(run['defaultDatasetId']).list_items().items
```

### Standby API (instant, single item)

Every actor has a Standby URL for sub-second, single-item requests:

```bash
curl "https://superlativetech--superclean-company-names.apify.actor?token=TOKEN&input=ACME+CORPORATION"
# {"id":1,"input":"ACME CORPORATION","output":"Acme","confidence":0.95}
```

## When to Use Which Actor

**Cleaning scraped lead data?**
- Company names with legal suffixes, ALL CAPS → **Company Names** (set `style`: casual/formal/legal)
- Job titles with abbreviations, team names → **Job Titles**
- Person names with titles, credentials → **Person Names**
- Product names with promo text, prices → **Product Names**
- Location strings → **Places** (returns city, stateOrProvince, country)
- Messy URLs with tracking params → **URLs**
- Phone numbers in mixed formats → **Phone Numbers** (set `defaultCountry`, `outputFormat`)

**Domain/email intelligence?**
- DNS records for any domain → **DNS Lookup** (MX, SPF, DMARC, TXT, A, AAAA, NS, SOA)
- Email deliverability audit → **Domain Health** (0-100 score, `mode`: sending/receiving, `depth`: basic/advanced)
- Domain registration data → **WHOIS Lookup** (registrar, dates, nameservers, 20 fields)

**Lead scoring?**
- Score leads against your ICP → **ICP Scorer** (any JSON lead objects + plain-English ICP description)

## Common Patterns

### Input formats

All Superclean actors accept `items` (string array) or `item` (single string):
```json
{ "items": ["value1", "value2"] }
{ "item": "single value" }
```

Objects also work: `{ "items": [{"input": "value1"}] }`

Supernet actors use `domains`/`domain` instead of `items`/`item`.
ICP Scorer uses `leads`/`lead` (JSON objects, not strings).

### Error handling

Actors return HTTP 400 for invalid input and HTTP 402 when spending limits are reached. In batch mode, individual item failures don't stop the run — check `confidence` < 0.5 for uncertain results.

### Volume discounts

All actors have automatic PPE tiered pricing:
- **Bronze** (100+ items): 10% off
- **Silver** (1,000+ items): 20% off
- **Gold** (10,000+ items): 30% off

## Pipeline Examples

Actors compose into data cleaning pipelines. See [references/pipeline-examples.md](references/pipeline-examples.md) for recipes.

Common pipeline: **Scrape** → **Clean** (Superclean actors) → **Enrich** (DNS/WHOIS/Domain Health) → **Score** (ICP Scorer)

## Output Schemas

All Superclean actors return: `{ id, input, output, confidence }`

Actors with additional fields:
- **Places**: adds `city`, `stateOrProvince`, `country`
- **Phone Numbers**: adds `e164`, `isValid`, `type`, `countryCode`
- **URLs**: adds `domain`, `protocol`, `path`, `valid`
- **Domain Health**: `checkId`, `domain`, `score`, `band`, `summary`, `issues[]`, `diagnostics[]`
- **WHOIS**: `domain`, `registrar`, `createdDate`, `expiryDate`, `domainAge`, `nameServers[]`, etc.
- **ICP Scorer**: `score` (0-100), `band`, `summary`, `confidence`

See detailed schemas in [references/superclean-actors.md](references/superclean-actors.md), [references/supernet-actors.md](references/supernet-actors.md), and [references/superlead-actors.md](references/superlead-actors.md).
