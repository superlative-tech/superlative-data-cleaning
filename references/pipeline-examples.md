# Pipeline Examples

Superlative actors compose into data cleaning and enrichment pipelines. Each actor does one thing well — chain them together for complex workflows.

## Lead Data Cleaning Pipeline

Clean scraped lead data before it enters your CRM or outreach tool.

```javascript
import { ApifyClient } from 'apify-client';
const client = new ApifyClient({ token: process.env.APIFY_TOKEN });

// Step 1: Clean company names
const companyRun = await client.actor('superlativetech/superclean-company-names').call({
  items: rawLeads.map(l => l.company),
  style: 'casual'
});
const companies = (await client.dataset(companyRun.defaultDatasetId).listItems()).items;

// Step 2: Clean person names
const nameRun = await client.actor('superlativetech/superclean-person-names').call({
  items: rawLeads.map(l => l.name)
});
const names = (await client.dataset(nameRun.defaultDatasetId).listItems()).items;

// Step 3: Normalize job titles
const titleRun = await client.actor('superlativetech/superclean-job-titles').call({
  items: rawLeads.map(l => l.title)
});
const titles = (await client.dataset(titleRun.defaultDatasetId).listItems()).items;

// Step 4: Merge cleaned data back
const cleanedLeads = rawLeads.map((lead, i) => ({
  ...lead,
  company: companies[i].output,
  name: names[i].output,
  title: titles[i].output,
}));
```

## Domain Intelligence Pipeline

Audit prospect domains before email outreach.

```javascript
// Step 1: Check domain health
const healthRun = await client.actor('superlativetech/supernet-domain-health').call({
  domains: prospects.map(p => p.domain),
  mode: 'receiving',
  depth: 'basic'
});
const health = (await client.dataset(healthRun.defaultDatasetId).listItems()).items;

// Step 2: WHOIS for company verification
const whoisRun = await client.actor('superlativetech/supernet-whois-lookup').call({
  domains: prospects.map(p => p.domain)
});
const whois = (await client.dataset(whoisRun.defaultDatasetId).listItems()).items;

// Step 3: Filter out bad domains
const validProspects = prospects.filter((p, i) =>
  health[i].score > 50 && health[i].hasMx && !health[i].isBlacklisted
);
```

## ICP Scoring Pipeline

Clean lead data first, then score against your ICP.

```javascript
// Step 1: Clean all fields (run in parallel)
const [companyRun, titleRun, nameRun] = await Promise.all([
  client.actor('superlativetech/superclean-company-names').call({ items: rawLeads.map(l => l.company) }),
  client.actor('superlativetech/superclean-job-titles').call({ items: rawLeads.map(l => l.title) }),
  client.actor('superlativetech/superclean-person-names').call({ items: rawLeads.map(l => l.name) }),
]);

// Step 2: Merge cleaned data into lead objects
const cleanedLeads = rawLeads.map((lead, i) => ({
  company: (await client.dataset(companyRun.defaultDatasetId).listItems()).items[i].output,
  title: (await client.dataset(titleRun.defaultDatasetId).listItems()).items[i].output,
  name: (await client.dataset(nameRun.defaultDatasetId).listItems()).items[i].output,
  industry: lead.industry,
  employees: lead.employees,
}));

// Step 3: Score against ICP
const scoreRun = await client.actor('superlativetech/superlead-icp-scorer').call({
  leads: cleanedLeads,
  icp: 'B2B SaaS companies with 50-500 employees. VP+ in Engineering or Product.'
});
const scores = (await client.dataset(scoreRun.defaultDatasetId).listItems()).items;

// Step 4: Prioritize
const hotLeads = scores.filter(s => s.band === 'Excellent' || s.band === 'Strong');
```

## Instant API (Standby) Pipeline

For real-time integrations (Clay, Make, n8n), use Standby mode for sub-second responses:

```javascript
const BASE = 'https://superlativetech--superclean-company-names.apify.actor';
const token = process.env.APIFY_TOKEN;

// Single-item cleaning — no Actor run needed
const response = await fetch(`${BASE}?token=${token}&input=${encodeURIComponent(rawCompanyName)}`);
const result = await response.json();
// { id: 1, input: "ACME CORP", output: "Acme", confidence: 0.95 }
```

Each Superclean actor's Standby URL follows the same pattern:
- `superlativetech--superclean-company-names.apify.actor`
- `superlativetech--superclean-job-titles.apify.actor`
- `superlativetech--superclean-person-names.apify.actor`
- `superlativetech--superclean-product-names.apify.actor`
- `superlativetech--superclean-places.apify.actor`
- `superlativetech--superclean-urls.apify.actor`
- `superlativetech--superclean-phone-numbers.apify.actor`
- `superlativetech--dns-lookup.apify.actor`
- `superlativetech--supernet-domain-health.apify.actor`
- `superlativetech--supernet-whois-lookup.apify.actor`
