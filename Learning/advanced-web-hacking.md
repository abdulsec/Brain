# Advanced Web Hacking Techniques

Where to learn, who to follow, and what to practice.

---

## The Techniques Worth Learning in 2026

### Tier 1: High Impact, Widely Applicable

#### HTTP Request Smuggling / Desync
Attack the connection between frontend (proxy/CDN/WAF) and backend.

| Variant | What It Exploits | Learn From |
|---------|-----------------|------------|
| **CL.TE** | Content-Length vs Transfer-Encoding confusion | PortSwigger Research |
| **TE.CL** | Reverse — TE vs CL confusion | PortSwigger Research |
| **H2.CL** | HTTP/2 downgrade injects Content-Length | James Kettle — "HTTP/2: The Sequel" |
| **H2.TE** | HTTP/2 downgrade + chunked confusion | James Kettle — Black Hat 2021 |
| **Browser-Powered** | Victim's browser smuggles request via HTML | PortSwigger — "Browser-Powered Desync" |
| **Client-Side Desync** | Browser ↔ frontend mismatch | PortSwigger — "Client-Side Desync" |

**Best resource**: [PortSwigger Web Security Academy — Request Smuggling](https://portswigger.net/web-security/request-smuggling) — 20+ free labs.

#### Web Cache Poisoning & Deception
Poison the CDN cache so other users get your malicious response.

| Technique | What | Key Researcher |
|-----------|------|---------------|
| **Cache Poisoning** | Poison cached response via unkeyed inputs | James Kettle |
| **Cache Deception** | Trick cache into storing sensitive data as static | Omer Gil |
| **CP-DoS** | Denial of Service via cache poisoning | PortSwigger Research |

**Best resource**: [PortSwigger — Web Cache Poisoning](https://portswigger.net/web-security/web-cache-poisoning)

#### OAuth / SSO Attacks
Every SaaS has SSO. Every SSO implementation has bugs.

| Technique | Impact |
|-----------|--------|
| **redirect_uri bypass** | Steal authorization codes |
| **state parameter leak** | CSRF on OAuth login |
| **PKCE downgrade** | Auth code interception |
| **scope upgrade** | Gain admin privileges |
| **OAuth to Account Takeover** | Full account compromise |
| **SSO parser differentials** | SAML sig bypass, XML signature wrapping |

**Best resource**: [PortSwigger — OAuth Labs](https://portswigger.net/web-security/oauth) + [SAML Raider Burp extension](https://github.com/portswigger/saml-raider)

#### Race Conditions
The fastest-growing class of high-severity bugs.

| Technique | Example |
|-----------|---------|
| **Single-packet attack** | Send multiple requests in one TCP packet |
| **TOCTOU** | Time-of-check-time-of-use on file/resource access |
| **Limit-overrun** | Race against rate limits (coupon codes, invites) |
| **Multi-endpoint** | Race two different endpoints simultaneously |
| **Turbo Intruder** | Burp extension for sub-millisecond race windows |

**Best resource**: [PortSwigger — Race Conditions](https://portswigger.net/web-security/race-conditions)

#### Server-Side Prototype Pollution
Poison `Object.prototype` in Node.js to control application logic.

**Best resource**: [PortSwigger — Prototype Pollution](https://portswigger.net/web-security/prototype-pollution)

#### SSTI (Server-Side Template Injection)
Inject template syntax that gets evaluated server-side.

| Engine | Syntax | Impact |
|--------|--------|--------|
| Jinja2 (Python) | `{{7*7}}` | RCE via `__subclasses__` |
| Twig (PHP) | `{{7*7}}` | RCE via `_self` |
| Freemarker (Java) | `${7*7}` | RCE via `Execute` |
| Velocity (Java) | `#set($x=7*7)` | RCE |
| ERB (Ruby) | `<%= 7*7 %>` | RCE via system() |
| Pug/Jade (Node) | `#{7*7}` | RCE via process.mainModule |

**Best resource**: [PortSwigger — SSTI](https://portswigger.net/web-security/server-side-template-injection)

---

### Tier 2: Specialized, High Skill Ceiling

#### Insecure Deserialization
Full RCE when apps deserialize untrusted input.

| Language | Gadget Chains |
|----------|---------------|
| **Java** | ysoserial, CommonsCollections, Spring beans |
| **PHP** | phpgcc, phar:// deserialization |
| **Python** | pickle, PyYAML, ruamel.yaml |
| **Node.js** | node-serialize, js-yaml, funcster |
| **Ruby** | Marshal.load, YAML.load |
| **.NET** | BinaryFormatter, DataContractSerializer |

**Best resource**: [PortSwigger — Deserialization](https://portswigger.net/web-security/deserialization)

#### GraphQL Attacks
Not SQLi on GraphQL — the API structure itself.

| Technique | Impact |
|-----------|--------|
| **Introspection abuse** | Discover all types, fields, mutations |
| **Alias-based batching** | Bypass rate limits, brute-force auth |
| **Field-level suggestions** | Steal schema even with introspection off |
| **Depth/circular queries** | DoS via `a { b { a { b { ... } } } }` |
| **IDOR via GraphQL** | Access other user's data via mutations |
| **Persisted query abuse** | Replay valid queries with modified variables |

**Best resources**: [PortSwigger — GraphQL](https://portswigger.net/web-security/graphql) + [InQL Burp extension](https://github.com/doyensec/inql)

#### JWT Attacks
Almost every API uses JWTs. Most implementations have bugs.

| Technique | How |
|-----------|-----|
| **alg:none** | Set algorithm to "none", skip signature |
| **Key confusion (HS256/RS256)** | Sign with public key using HMAC |
| **JWKS injection** | Inject your own JWK via jku/x5u headers |
| **kid injection** | SQLi/Path traversal in kid header |
| **Empty signature** | Remove signature entirely |
| **weak HMAC secret** | Brute-force HS256 secret |
| **JWT to JWK cross-realm** | Use one realm's JWT in another |
| **Embedded JWK bypass** | Provide your own public key in the token |

**Best resources**: [PortSwigger — JWT](https://portswigger.net/web-security/jwt) + `jwt_tool` + `jwt-forgery.py`

#### SSRF (Advanced)
Beyond `http://169.254.169.254/latest/meta-data`.

| Technique | Bypass |
|-----------|--------|
| **DNS rebinding** | Change DNS mid-request to hit internal IP |
| **302 redirect** | Point SSRF to your server, 302 to internal |
| **IPv6/IPv4 confusion** | `http://[::ffff:169.254.169.254]/` |
| **URL parser differential** | WAF vs backend parse URL differently |
| **DNS pinning** | Resolve once to external, second to internal |
| **localhost aliases** | 127.0.0.1 → 2130706433, 0x7f000001, 017700000001 |
| **CRLF in URL** | Inject headers via SSRF target URL |
| **Gopher/other protocols** | `gopher://`, `dict://`, `file://` |

**Best resource**: [Assetnote — SSRF Bible](https://blog.assetnote.io/2021/01/13/blind-ssrf-chains/)

---

### Tier 3: Cutting Edge (2024–2026)

#### Next.js / React Server Components Attacks
The React2Shell era — server actions are attack surface.

| Attack | Target |
|--------|--------|
| **Server Action pollution** | Send unexpected action IDs |
| **Flight protocol abuse** | React Flight serialization tricks |
| **RSC payload injection** | Inject into server component props stream |
| **Router cache poisoning** | Poison Next.js client-side router cache |
| **Middleware bypass** | Skip Next.js middleware via HPP/parser tricks |

**Best resources**: [Next.js Security Docs](https://nextjs.org/docs/app/building-your-application/configuring/content-security-policy) + disclosed H1 reports tagged "next.js"

#### HTTP/2 & HTTP/3 Attacks

| Technique | Key Insight |
|-----------|-------------|
| **H2 smuggling** | HTTP/2 → HTTP/1.1 downgrade injects headers |
| **H2 multiplexing abuse** | Multiple streams confuse connection-scoped state |
| **H2 rapid reset** | DoS via RST_STREAM flood (CVE-2023-44487) |
| **QPACK header bomb** | Compress large headers via QPACK |
| **HTTP/3 smuggling** | QUIC → HTTP/1.1 translation edge cases |

**Best resource**: Jake Miller (BishopFox) — "HTTP/2 Attacks" research

#### Browser-Powered Attacks (XS-Leaks)

| Technique | Leaks |
|-----------|-------|
| **Frame counting** | `window.length` reveals number of frames |
| **CSS injection** | `@import` + timing = data exfiltration |
| **Pixel stealing** | Canvas + timing + SVG filters |
| **ID attribute timing** | `#fragment` causes scroll → timing side channel |
| **Performance API** | `performance.getEntriesByType('resource')` leaks URLs |
| **CORB/CORP bypass** | Read cross-origin data via side channels |

**Best resource**: [XS-Leaks Wiki](https://xsleaks.dev/)

---

## The People to Follow

### Researchers (read everything they publish)

| Researcher | Known For | Where |
|-----------|-----------|-------|
| **James Kettle (albinowax)** | Request smuggling, cache poisoning, browser-powered attacks | [PortSwigger Research](https://portswigger.net/research) |
| **Orange Tsai** | SSRF, parser differentials, Exchange/SharePoint RCE | [Orange's Blog](https://blog.orange.tw/) |
| **Nagli** | HTTP request smuggling, WAF bypass | [Nagli's Blog](https://n4gli.github.io/) |
| **d3d** | Regular expression DoS, HTTP parsing | [Twitter](https://x.com/d3delse) |
| **Shubham Shah (Assetnote)** | SSRF, SSTI, cloud metadata | [Assetnote Blog](https://blog.assetnote.io/) |
| **Rhino Security Labs** | AWS IAM, cloud security | [Rhino Blog](https://rhinosecuritylabs.com/) |
| **SonarSource** | Source code analysis, novel vuln classes | [Sonar Blog](https://blog.sonarsource.com/) |
| **WatchTowr** | WAF bypass, exploit chains | [WatchTowr Blog](https://labs.watchtowr.com/) |
| **Soroush Dalili** | WAF bypass, encoding tricks | [Soroush's Blog](https://soroush.me/) |
| **Ala Alaajaj** | Race conditions, business logic | [Twitter](https://x.com/alaAlaajaj) |

### Teams/Orgs (their research feeds are gold)

- **PortSwigger Research** — portswigger.net/research (James Kettle, Michael Stepankin)
- **Assetnote** — blog.assetnote.io
- **BishopFox** — bishopfox.com/blog
- **SonarSource** — blog.sonarsource.com
- **WatchTowr Labs** — labs.watchtowr.com
- **Doyensec** — doyensec.com (Electron, GraphQL)
- **Include Security** — includesecurity.com (SSRF, Cloud)
- **Hacktivity** — hackerone.com/hacktivity (filter by "disclosed" + sort by bounty)

---

## The Path: Where to Start

### If you're comfortable with basic web vulns and want to level up:

```
Month 1: Request Smuggling
  → Complete all 20+ PortSwigger smuggling labs
  → Read every James Kettle paper on desync
  → Then: apply to real CDN/WAF targets

Month 2: JWT + OAuth
  → PortSwigger JWT labs + OAuth labs
  → Build a JWT manipulation script
  → Then: test every API you hunt that uses tokens

Month 3: Cache Poisoning
  → PortSwigger cache poisoning labs
  → Learn to identify unkeyed inputs
  → Then: test Cloudflare/Akamai-fronted targets

Month 4: Race Conditions
  → PortSwigger race condition labs
  → Master Turbo Intruder / single-packet attack
  → Then: every limit/coupon/one-time-use endpoint

Month 5: Prototype Pollution + SSTI
  → PortSwigger labs for both
  → Server-side PP: find Node.js targets
  → SSTI: any app with templates + user input

Month 6+: Pick your specialty
  → GraphQL (every modern API is GraphQL)
  → SSRF (every cloud app has internal metadata)
  → Next.js (the new attack surface)
```

---

## All Free Labs (PortSwigger Web Security Academy)

```
https://portswigger.net/web-security/all-labs

Request Smuggling:      20+ labs
JWT:                    8 labs
OAuth:                  6 labs
GraphQL:                5 labs
SSRF:                   7 labs
SSTI:                   7 labs
Prototype Pollution:    10 labs
Race Conditions:        6 labs
Web Cache Poisoning:    9 labs
Deserialization:        8 labs
SQL Injection:          16 labs
XSS:                    30 labs
CORS:                   3 labs
Clickjacking:           5 labs
WebSockets:             3 labs
DOM-based:              7 labs
File Upload:            7 labs
Path Traversal:         7 labs
OS Command Injection:   4 labs
Business Logic:         11 labs
Information Disclosure: 5 labs
Authentication:         14 labs
Access Control:         13 labs

Total: 200+ free labs
```

---

## Tools You Need

```bash
# Request Smuggling
# Burp Extension: HTTP Request Smuggler (PortSwigger)
# Turso Intruder (race conditions)
# Turbo Intruder (high-speed bruteforce)

# JWT
# jwt_tool (python)
# Portal for ArcGIS (JWT analysis)

# SSRF
# interactsh (out-of-band, from projectdiscovery)
# Burp Collaborator (built in)

# GraphQL
# InQL (Burp extension)
# GraphQL Raider
# Clairvoyance (schema extraction even with introspection off)

# Prototype Pollution
# ppfuzz (scan for PP gadgets)
# ppscan

# Deserialization
# ysoserial (Java)
# phpgcc (PHP)
# ysoserial.net (.NET)

# General
# Caido (your main proxy)
# nuclei (automated scanning with WAF bypass templates)
# ffuf (fuzzing with encoding transforms)
```

---

## How to Read Research Papers (Get the Most From Them)

When a paper drops (e.g., "Browser-Powered Desync Attacks" by James Kettle):

1. **Read the abstract** → understand the attack class
2. **Read the "How We Found It" section** → methodology matters more than the bug
3. **Reproduce ONE example** → spin up a lab and do it yourself
4. **Read the exploit code** → understand every line
5. **Ask: "What else could this apply to?"** → generalize the technique

The difference between reading about a technique and being able to use it is **reproduction**. Always reproduce.
