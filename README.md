[README.md — Phishing Infrastructure Investigation.md](https://github.com/user-attachments/files/32621077/README.md.Phishing.Infrastructure.Investigation.md)
# 🔎 Phishing Infrastructure Investigation

## Overview

This project documents the technical investigation of a suspicious promotional landing page associated with the domain:

```text
sweepquest[.]shop
```

The objective was to analyze the web infrastructure, identify tracking mechanisms, investigate DNS and domain registration information, examine JavaScript behavior, and document indicators of compromise (IOCs).

The investigation was performed using passive OSINT, DNS/RDAP analysis, Certificate Transparency and static analysis of the landing page.

> ⚠️ **Disclaimer:** This project is intended for cybersecurity research and educational purposes. No credentials or personal information were submitted to the investigated infrastructure.

---

## 🎯 Objectives

The investigation focused on:

- Identifying the infrastructure behind the landing page.
- Investigating DNS and nameservers.
- Examining domain registration information.
- Analyzing TLS certificates.
- Identifying CDN infrastructure.
- Analyzing JavaScript behavior.
- Identifying tracking mechanisms.
- Identifying anti-analysis techniques.
- Extracting relevant IOCs.
- Establishing potential infrastructure relationships.
- Assessing whether publicly available information could identify the operator.

---

# 🧪 Methodology

The investigation followed this workflow:

```text
Suspicious URL
      │
      ▼
Initial Web Analysis
      │
      ├── HTML
      ├── JavaScript
      └── Resources
      │
      ▼
Infrastructure Analysis
      │
      ├── DNS
      ├── Nameservers
      ├── RDAP
      ├── TLS
      └── Certificate Transparency
      │
      ▼
Tracking Analysis
      │
      ├── Campaign identifiers
      ├── Tracking endpoints
      ├── Device telemetry
      └── Timing information
      │
      ▼
Anti-Analysis Analysis
      │
      ├── DevTools detection
      ├── Ctrl+S blocking
      └── Context-menu blocking
      │
      ▼
IOC Collection
      │
      ▼
Infrastructure Correlation
```

---

# 🌐 Domain Analysis

### Domain

```text
sweepquest[.]shop
```

The domain was registered on:

```text
2026-08-07 01:45:04 UTC
```

The registrar identified through RDAP was:

```text
Spaceship, Inc.
```

The domain uses Cloudflare nameservers:

```text
hayes.ns.cloudflare.com
nova.ns.cloudflare.com
```

---

# 🌍 DNS Analysis

A DNS query returned:

```text
172.67.139.155
104.21.70.211
```

These addresses correspond to Cloudflare infrastructure.

Therefore, they should **not** be considered confirmed origin-server IP addresses.

### Nameservers

```text
hayes.ns.cloudflare.com
nova.ns.cloudflare.com
```

---

# 🔐 TLS / Certificate Analysis

The observed TLS certificate contained:

```text
Subject:
CN=sweepquest.shop

Issuer:
Google Trust Services
WE1

Not Before:
2026-08-07 01:42:07 UTC

Not After:
2026-11-05 02:40:37 UTC
```

Certificate Transparency records were also observed for:

```text
sweepquest.shop
*.sweepquest.shop
```

Additional certificates were issued by:

```text
Google Trust Services
Sectigo
```

An interesting timeline correlation was observed:

```text
01:42:07 UTC → TLS certificate validity begins
01:45:04 UTC → Domain registration recorded
```

The approximately three-minute difference is noteworthy but **does not constitute evidence of attribution**.

---

# 📡 CDN Infrastructure

The HTML contained:

```html
<base href="https://gogotracker.b-cdn.net/bonus/all-cc/lp1/">
```

This identified:

```text
gogotracker[.]b-cdn[.]net
```

as infrastructure used by the landing page.

The hostname is associated with Bunny CDN infrastructure.

The name `gogotracker` should be considered an infrastructure indicator rather than evidence of the identity of an individual operator.

---

# 🧩 Landing Page Analysis

The landing page contained several promotional products/offers, including references to:

- Samsung S25
- iPhone 17
- PayPal
- Walmart
- Gas

The page also contained:

```html
<meta name="robots" content="noindex, nofollow">
<meta name="googlebot" content="noindex">
```

This requests that search engines do not index the page.

---

# 📊 Tracking Analysis

The JavaScript contained a tracking object named:

```javascript
PK
```

The script extracts parameters from the URL:

```javascript
PK.c = PK.getToken("c"),
PK.k = PK.getToken("k"),
PK.e = PK.getToken("e")
```

The observed URL contained campaign/session-related parameters.

For publication, sensitive or potentially reusable values have been redacted.

---

# 📡 Tracking Mechanism

One particularly interesting finding was the tracking implementation.

The script does not require:

```text
fetch()
XMLHttpRequest
```

Instead, it dynamically creates an image:

```javascript
var n = document.createElement("img");
n.src = e + "&t=" + Math.random();
```

This causes the browser to make an HTTP request to the specified tracking endpoint.

---

# 🎯 Tracking Endpoints

The following endpoints were identified in the JavaScript:

```text
/ctrack.php

/lib/ajax/campdata.php

/lib/ajax/lp_timing.php

/lib/ajax/lp_engage.php
```

The endpoints appear to be associated with campaign tracking and visitor telemetry.

---

# 🖥️ Device Telemetry

The JavaScript collects information including:

```text
screen.width
screen.height
window.innerWidth
window.innerHeight
document.documentElement.scrollWidth
document.documentElement.scrollHeight
```

This allows the system to record information about:

- Screen resolution.
- Browser viewport.
- Page content dimensions.

---

# ⏱️ Timing Telemetry

The script calculates page-load timing information.

It also attempts to obtain DNS lookup timing through:

```javascript
domainLookupEnd - domainLookupStart
```

The resulting information is sent to:

```text
/lib/ajax/lp_timing.php
```

---

# 🔗 Offer / Redirect Infrastructure

The JavaScript defines:

```javascript
PK.buildoffer()
```

and:

```javascript
PK.buildoffer_random()
```

These functions can construct URLs pointing to:

```text
/click.php
```

with parameters including:

```text
c
k
offer
productname
producturl
```

However, static analysis of the examined `index.php` did not identify a direct call to these functions.

Therefore:

> The existence of `click.php` functionality is confirmed in the JavaScript, but its use in the specific page flow examined was not conclusively established.

---

# 🛡️ Anti-Analysis Techniques

The file:

```text
site-protect2.0.js
```

contains several anti-analysis mechanisms.

## DevTools detection

The script periodically checks browser window dimensions and other conditions to detect developer tools.

The check runs approximately every:

```text
500 ms
```

When the defined conditions are detected, the page redirects to:

```text
https://www.google.com/
```

---

## Ctrl+S blocking

The script attempts to prevent:

```text
Ctrl + S
```

from being used.

---

## Context-menu blocking

The script also disables the browser context menu:

```text
contextmenu
```

This prevents normal right-click interaction.

---

# 🔍 Infrastructure Map

```text
                    ┌──────────────────────┐
                    │  Suspicious Landing  │
                    │   sweepquest[.]shop  │
                    └──────────┬───────────┘
                               │
                ┌──────────────┼──────────────┐
                │              │              │
                ▼              ▼              ▼
             Cloudflare     Tracking       CDN
                │              │              │
                │         ctrack.php       b-cdn.net
                │         campdata.php     gogotracker
                │         lp_timing.php
                │         lp_engage.php
                │
                ▼
          DNS / Proxy Layer
```

---

# 🚨 Indicators of Compromise

| Type | Indicator |
|---|---|
| Domain | `sweepquest[.]shop` |
| CDN hostname | `gogotracker[.]b-cdn[.]net` |
| Nameserver | `hayes.ns.cloudflare.com` |
| Nameserver | `nova.ns.cloudflare.com` |
| Endpoint | `/ctrack.php` |
| Endpoint | `/click.php` |
| Endpoint | `/lib/ajax/campdata.php` |
| Endpoint | `/lib/ajax/lp_timing.php` |
| Endpoint | `/lib/ajax/lp_engage.php` |

Campaign/session identifiers observed during the investigation have intentionally been omitted from this public repository.

---

# 🕒 Timeline

| Date / Time UTC | Event |
|---|---|
| 2026-08-07 01:42:07 | TLS certificate validity begins |
| 2026-08-07 01:45:04 | Domain registration recorded |
| 2026-08-07 | Certificate Transparency records observed |
| 2026-09-10 04:36:44 | RDAP last-change event |
| 2026-09-24 | Technical investigation performed |

---

# 🧠 Key Findings

The investigation identified:

### 1. Recently registered infrastructure

The domain was registered on 7 August 2026.

### 2. Cloudflare infrastructure

Cloudflare is used for DNS/proxy infrastructure.

### 3. CDN infrastructure

The landing page references:

```text
gogotracker[.]b-cdn[.]net
```

### 4. Campaign tracking

The JavaScript implements a dedicated tracking system.

### 5. Visitor telemetry

The system records browser/window dimensions and timing information.

### 6. Anti-analysis mechanisms

The page contains mechanisms designed to interfere with common inspection techniques.

### 7. Potential offer-routing infrastructure

The JavaScript contains functionality capable of constructing `click.php` offer URLs.

---

# ⚠️ Attribution Limitations

The investigation did not identify sufficient public evidence to attribute the infrastructure to a specific individual.

The following were **not identified**:

```text
Confirmed operator name
Confirmed operator phone number
Confirmed personal email
Confirmed physical address
Confirmed origin-server IP
```

Infrastructure providers such as:

```text
Cloudflare
Spaceship
Bunny CDN
```

should not be interpreted as participants in the phishing operation simply because their infrastructure was used.

Similarly, the hostname:

```text
gogotracker
```

does not establish the identity of the operator.

---

# 📌 Conclusion

The investigation identified a suspicious web infrastructure consisting of a recently registered domain, Cloudflare DNS/proxy services, CDN resources hosted under `b-cdn.net`, JavaScript-based campaign tracking and anti-analysis mechanisms.

The JavaScript provides evidence of visitor telemetry and multiple tracking endpoints, while additional functions appear capable of constructing tracked offer URLs.

The available evidence is sufficient to document the technical infrastructure and generate useful IOCs, but it is insufficient to attribute the operation to a specific individual.

The next phase of the investigation would be infrastructure clustering: identifying other domains that reuse the same JavaScript, tracking endpoints, URL structures, CDN hostnames or other technical fingerprints.

---

# 📚 Tools Used

- `dig`
- `curl`
- RDAP
- Certificate Transparency / `crt.sh`
- `grep`
- Static HTML analysis
- JavaScript analysis
- Browser Developer Tools
- OSINT techniques

---

# ⚖️ Disclaimer

This repository documents cybersecurity research performed for educational and defensive purposes.

The objective is to understand phishing infrastructure, identify technical indicators and improve defensive detection capabilities.

No attempt was made to access private accounts, obtain credentials, compromise systems or identify private individuals through unauthorized means.
