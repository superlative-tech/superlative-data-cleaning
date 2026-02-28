# Superlead Actors — Detailed Reference

Lead intelligence and scoring tools.

## ICP Scorer

**Actor ID**: `superlativetech/superlead-icp-scorer`

Scores sales leads against your Ideal Customer Profile (ICP) using AI. Submit any JSON lead objects with a plain-English ICP description. Returns 0-100 fit scores with band labels and explanations.

**Input:**
- `leads`: object[] — lead objects with any fields (company, title, industry, employees, etc.)
- `lead`: object — single lead (API shorthand)
- `icp`: string (required) — plain-English ICP description
- `model`: string — LLM model (default `openrouter/auto`)
- `openRouterApiKey`: string — optional BYOK for OpenRouter

**Example input:**
```json
{
  "leads": [
    {"company": "Acme Corp", "title": "VP of Engineering", "industry": "SaaS", "employees": 150},
    {"company": "MegaRetail Inc", "title": "Marketing Intern", "industry": "Retail", "employees": 50000}
  ],
  "icp": "B2B SaaS companies with 50-500 employees. Series A to C. Decision makers: VP+ in Engineering or Product. North America."
}
```

**Standby:**
```
POST https://superlativetech--superlead-icp-scorer.apify.actor?token=TOKEN
Content-Type: application/json

{"lead": {"company": "Acme Corp", "title": "VP Engineering"}, "icp": "B2B SaaS, 50-500 employees"}
```

**Output fields:** `score` (0-100), `band`, `summary`, `confidence`, plus original lead fields

**Bands:** Excellent (90-100), Strong (75-89), Good (60-74), Moderate (40-59), Weak (20-39), Poor (0-19)

**Output example:**
```json
{
  "score": 87,
  "band": "Strong",
  "summary": "Strong ICP fit: VP-level engineering leader at mid-market SaaS company within target employee range.",
  "confidence": 0.9,
  "lead": {"company": "Acme Corp", "title": "VP of Engineering", "industry": "SaaS", "employees": 150}
}
```
