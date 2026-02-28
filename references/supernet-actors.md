# Supernet Actors — Detailed Reference

Domain and network intelligence tools. Use `domains` (string[]) or `domain` (string) for input.

## DNS Lookup

**Actor ID**: `superlativetech/dns-lookup`

Looks up DNS records for any domain. Supports MX, SPF, DMARC, TXT, A, AAAA, CNAME, NS, SOA, and reverse DNS (PTR).

**Input:**
- `domains`: string[] (required) — domain names or IP addresses
- `dnsRecordTypes`: string[] — specific record types to query (default: all)
- `reverseLookup`: boolean — set true for reverse DNS (IP → hostname)

**Standby:**
```
GET https://superlativetech--dns-lookup.apify.actor?token=TOKEN&domain=example.com
GET https://superlativetech--dns-lookup.apify.actor?token=TOKEN&domain=example.com&types=MX,TXT
```

**Output example:**
```json
{
  "domain": "example.com",
  "A": [{"value": "93.184.216.34", "ttl": 300}],
  "MX": [{"priority": 10, "value": "mail.example.com", "ttl": 3600}],
  "TXT": [{"value": "v=spf1 include:_spf.example.com ~all", "ttl": 3600}],
  "NS": [{"value": "ns1.example.com", "ttl": 86400}]
}
```

## Domain Health

**Actor ID**: `superlativetech/supernet-domain-health`

Scores email domain health 0-100. Checks SPF, DKIM, DMARC, MX, blacklists, SSL, WHOIS, BIMI, MTA-STS, TLSRPT.

**Input:**
- `domains`: string[] — domains to check
- `domain`: string — single domain (API shorthand)
- `mode`: `sending` (default) or `receiving` — sending audits your domain, receiving validates prospect domains
- `depth`: `basic` (default) or `advanced` — advanced adds blacklists, BIMI, MTA-STS, TLSRPT

**Pricing:** $2.50/1K (basic), $5.00/1K (advanced)

**Standby:**
```
GET https://superlativetech--supernet-domain-health.apify.actor?token=TOKEN&domain=example.com
GET https://superlativetech--supernet-domain-health.apify.actor?token=TOKEN&domain=example.com&mode=receiving&depth=advanced
```

**Output example:**
```json
{
  "checkId": "abc123",
  "domain": "example.com",
  "score": 85,
  "band": "Good",
  "summary": "Strong email authentication with minor improvements available.",
  "hasMx": true,
  "hasSpf": true,
  "hasDkim": true,
  "hasDmarc": true,
  "dmarcPolicy": "reject",
  "isBlacklisted": false,
  "issuesCritical": 0,
  "issuesWarning": 2,
  "issuesInfo": 1,
  "issuesList": ["DKIM key rotation recommended", "Missing BIMI record"],
  "mode": "sending",
  "depth": "basic",
  "durationMs": 1234,
  "issues": [{"module": "dkim", "severity": "warning", "message": "DKIM key rotation recommended"}],
  "diagnostics": [{"module": "spf", "status": "pass", "detail": "SPF record found and valid"}],
  "scoreBreakdown": {"mx": 10, "spf": 15, "dkim": 15, "dmarc": 20},
  "details": {}
}
```

## WHOIS Lookup

**Actor ID**: `superlativetech/supernet-whois-lookup`

Looks up WHOIS registration data for any domain. Returns 20 structured fields.

**Input:**
- `domains`: string[] — domains to look up
- `domain`: string — single domain (API shorthand)

**Standby:**
```
GET https://superlativetech--supernet-whois-lookup.apify.actor?token=TOKEN&domain=example.com
```

**Output example:**
```json
{
  "domain": "example.com",
  "registrar": "GoDaddy",
  "createdDate": "1997-09-15T00:00:00Z",
  "updatedDate": "2024-01-15T00:00:00Z",
  "expiryDate": "2030-09-15T00:00:00Z",
  "domainAge": 10358,
  "registrant": "Example Inc.",
  "registrantOrg": "Example Inc.",
  "registrantCountry": "US",
  "registrantState": "CA",
  "nameServers": ["ns1.example.com", "ns2.example.com"],
  "status": ["clientTransferProhibited"],
  "dnssec": true,
  "whoisServer": "whois.godaddy.com",
  "rawWhois": "..."
}
```

## HTTP API

**Actor ID**: `superlativetech/http-api`

Makes HTTP requests from Apify infrastructure. Building block for automation workflows.

**Input:**
- `url`: string — target URL
- `method`: `GET`, `POST`, `PUT`, `DELETE`
- `headers`: object — custom headers
- `body`: string — request body
- `auth`: object — authentication config

**Output:** Returns `status`, `headers`, `body` from the response.
