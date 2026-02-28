# Superclean Actors — Detailed Reference

## Common Patterns

All 7 Superclean actors share:
- **Input**: `items` (string[]) or `item` (string). Objects also accepted: `[{"input": "value"}]`
- **Base output**: `{ id, input, output, confidence }` where `id` is 1-based, `confidence` is 0-1
- **Dual mode**: Batch (run actor with array) or Standby (instant HTTP GET for single items)
- **Standby URL**: `https://superlativetech--{actor-name}.apify.actor?token=TOKEN&input=VALUE`

## Company Names

**Actor ID**: `superlativetech/superclean-company-names`

Normalizes messy company names. AI-powered (LLM).

**Extra input params:**
- `style`: `casual` (default), `formal`, or `legal`
- `model`: LLM model (default `openrouter/auto`)
- `openRouterApiKey`: Optional BYOK for OpenRouter

**Styles:**
| Style | Input | Output |
|-------|-------|--------|
| casual | `Samsung Electronics Co., Ltd.` | `Samsung` |
| formal | `Samsung Electronics Co., Ltd.` | `Samsung Electronics` |
| legal | `Samsung Electronics Co., Ltd.` | `Samsung Electronics Co., Ltd.` |

**Output example:**
```json
{ "id": 1, "input": "ACME CORPORATION, INC.", "output": "Acme", "confidence": 0.95 }
```

**Standby example:**
```
GET https://superlativetech--superclean-company-names.apify.actor?token=TOKEN&input=ACME+CORPORATION&style=casual
```

## Job Titles

**Actor ID**: `superlativetech/superclean-job-titles`

Normalizes messy job titles. AI-powered (LLM).

**Extra input params:**
- `model`: LLM model (default `openrouter/auto`)
- `openRouterApiKey`: Optional BYOK

**Output example:**
```json
{ "id": 1, "input": "Sr. Software Eng. - Platform Team", "output": "Senior Software Engineer", "confidence": 0.95 }
```

## Person Names

**Actor ID**: `superlativetech/superclean-person-names`

Normalizes messy person names. Handles ALL CAPS, titles (Dr., Mr.), credentials (PhD, MD), and special cases (McDonald, O'Brien). AI-powered (LLM).

**Extra input params:**
- `model`: LLM model (default `openrouter/auto`)
- `openRouterApiKey`: Optional BYOK

**Output example:**
```json
{ "id": 1, "input": "DR. JOHN MCDONALD III, PHD", "output": "John McDonald III", "confidence": 0.95 }
```

## Product Names

**Actor ID**: `superlativetech/superclean-product-names`

Cleans product names from e-commerce data. Removes promo text, prices, SKUs, formatting noise. AI-powered (LLM).

**Extra input params:**
- `model`: LLM model (default `openrouter/auto`)
- `openRouterApiKey`: Optional BYOK

**Output example:**
```json
{ "id": 1, "input": "★★★ iPhone 15 Pro Max 256GB SALE $999 FREE SHIPPING ★★★", "output": "iPhone 15 Pro Max 256GB", "confidence": 0.95 }
```

## Places

**Actor ID**: `superlativetech/superclean-places`

Parses location strings into structured city/state/country fields. Rule-based (no LLM).

**Output fields:** `id`, `input`, `output`, `city`, `stateOrProvince`, `country`, `confidence`

**Output example:**
```json
{
  "id": 1,
  "input": "New York, NY",
  "output": "New York, New York, United States",
  "city": "New York",
  "stateOrProvince": "New York",
  "country": "United States",
  "confidence": 0.95
}
```

## URLs

**Actor ID**: `superlativetech/superclean-urls`

Cleans and normalizes URLs. Removes tracking parameters, validates format. Rule-based (no LLM).

**Output fields:** `id`, `input`, `output`, `domain`, `protocol`, `path`, `query`, `valid`, `confidence`

**Output example:**
```json
{
  "id": 1,
  "input": "https://example.com/page?utm_source=google&utm_medium=cpc&ref=abc",
  "output": "https://example.com/page",
  "domain": "example.com",
  "protocol": "https",
  "path": "/page",
  "query": "",
  "valid": true,
  "confidence": 0.95
}
```

## Phone Numbers

**Actor ID**: `superlativetech/superclean-phone-numbers`

Cleans and validates phone numbers. E.164 format, type detection, country code extraction. Rule-based (no LLM).

**Extra input params:**
- `defaultCountry`: Country code for numbers without country prefix (default `US`)
- `outputFormat`: `e164` (default), `international`, or `national`

**Output fields:** `id`, `input`, `output`, `e164`, `isValid`, `type`, `countryCode`, `extension`

**Output example:**
```json
{
  "id": 1,
  "input": "(555) 123-4567",
  "output": "+15551234567",
  "e164": "+15551234567",
  "isValid": true,
  "type": "fixed-line-or-mobile",
  "countryCode": "US",
  "extension": null
}
```
