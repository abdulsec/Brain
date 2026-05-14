# AI-Assisted WAF Bypass Discovery

Practical methods to use AI for finding parser differentials that bypass WAFs.

---

## The Setup You Need

Before AI helps, you need to answer **three questions**:

1. **What WAF?** — Cloudflare? Vercel? AWS? ModSecurity? Custom nginx rules?
2. **What backend?** — Next.js? Express? Flask? Rails? Spring Boot?
3. **What parser does each use?** — This is the key. Find the exact library.

```
Example:
WAF: Cloudflare → uses custom Go HTTP parser (partially documented)
Backend: Next.js 15.x → uses Node.js http-parser + busboy 1.6.x for multipart
```

**If you can't answer #3, AI can't do much.** Garbage in, garbage out.

---

## Method 1: White-Box — Feed AI Both Parsers (Strongest)

This is what the React2Shell researchers did with Gemini 2.5 Pro. It reproduced all 5 bypasses.

### Step 1: Get the WAF parser source

```bash
# If WAF is open source (ModSecurity, Coraza)
git clone https://github.com/corazawaf/coraza.git

# If WAF is Cloudflare/CloudFront — you don't have source
# But you DO have: blog posts, engineering talks, RFC compliance docs
# Search: "Cloudflare HTTP parser" "how Cloudflare parses multipart"
```

### Step 2: Get the backend parser source

```bash
# For Next.js / Node.js backends
# Find the multipart parser
grep -r "busboy\|formidable\|multer\|multipart" node_modules/

# For Python backends
# Django: django/http/multipartparser.py
# Flask: uses werkzeug/formparser.py

# For Go backends
# Check mime/multipart stdlib
```

### Step 3: Feed both to AI

Use this exact prompt template:

```
I'm looking for HTTP parser differentials between two implementations.

## WAF Parser Code (reads request first):

[paste the WAF's multipart boundary parsing function here]
[paste the WAF's header parsing code here]
[paste the WAF's content-type normalization here]

## Backend Parser Code (reads request after WAF):

[paste the backend's multipart boundary parsing function here]
[paste the backend's header parsing code here]
[paste the backend's content-type normalization here]

## Task
Find inputs where these two parsers produce DIFFERENT results for the same HTTP request.
Focus on:
1. Boundary parameter handling (quotes, whitespace, special chars, duplicates)
2. Content-Type charset handling (what charsets are accepted, how invalid ones fail)
3. Header duplicate handling (which value wins when duplicated?)
4. Edge cases in Content-Disposition parsing
5. How each parser handles invalid/malformed multipart framing
6. Null byte behavior
7. Trailing garbage after boundaries

For each finding, give me:
- The exact HTTP request that triggers the differential
- What the WAF sees vs what the backend sees
- Why they differ (specific line of code or RFC interpretation)
```

### Real example: React2Shell Bypass 3 (UTF-16LE)

If you fed the WAF and busboy source to AI, it would spot:

```
WAF: scans raw bytes, looks for ":constructor" string
Busboy: Content-Type: text/plain; charset=utf16le → decodes from UTF-16LE first

Differential: WAF sees raw bytes (no ":constructor"), busboy sees decoded string (":constructor" appears)
```

---

## Method 2: Spec-Based — Feed AI the RFCs + Docs (Medium)

When you don't have source but have documentation.

### Step 1: Collect spec/documents

```
- RFC 7578 (multipart/form-data)
- RFC 2046 §5.1 (MIME multipart)
- The WAF's public documentation on how it handles requests
- The backend framework's HTTP parsing docs
- Any engineering blog posts about either parser
```

### Step 2: The prompt

```
I'm looking for parsing differential gaps between two HTTP implementations.

## What we know about the WAF parser:
[documentation snippets, blog posts, known behaviors]

## What we know about the backend parser:
[documentation, RFC compliance statements, code snippets if available]

## Task
Based on what's known about each parser, identify:
1. Edge cases the WAF docs don't mention that the RFC allows
2. Differences in how they handle ambiguous RFC language
3. Features the backend supports that the WAF probably doesn't parse

For each hypothesis, generate test payloads to verify.
```

### Step 3: Verify with actual requests

AI gives hypotheses. You test them against the real WAF.

```
AI hypothesis: "Vercel WAF might not handle quoted boundary parameters"
Test: Content-Type: multipart/form-data; boundary="x" vs boundary=x
Result: Both work → hypothesis wrong, move to next
```

---

## Method 3: Black-Box Pattern Analysis (Weakest, but sometimes works)

When you know nothing about either parser.

### Step 1: Build a probe set

Send these requests and record: blocked or allowed.

```
1. Normal request (baseline — should be allowed)
2. Obviously malicious (baseline — should be blocked)
3. Request with duplicate Content-Type headers
4. Request with boundary=x; boundary=y
5. Request with charset=utf-16le
6. Request with null byte in body
7. Request with trailing space after boundary marker
8. Request with unicode in header names
9. Request with Transfer-Encoding: chunked AND Content-Length
10. Request with Content-Type: text/html on a JSON endpoint
... (more probes from the technique catalog)
```

### Step 2: Feed the pattern to AI

```
I'm fingerprinting a WAF. Here's what I observed:

| Test | Payload | Response | Status |
|------|---------|----------|--------|
| 1 | Normal POST | 200 | allowed |
| 2 | POST with <script> in body | 403 + "X-Cloudflare: blocked" | blocked |
| 3 | Duplicate Content-Type headers | 200 | allowed |
| 4 | charset=utf-16le in Content-Type | 500 | error (reached backend?) |
| 5 | boundary with trailing space | 200 | allowed |
| ... | ... | ... | ... |

## Task
Based on this pattern, what WAF technology does this look like?
What parsing logic explains these results?
What payloads should I try next that are likely to bypass?
```

This is the least reliable but can give you direction.

---

## Method 4: AI-Assisted Fuzzing Loop (Automated)

Automate the cycle: generate → test → analyze → refine.

### Script: `waf-fuzz-loop.sh`

```bash
#!/bin/bash
# AI-assisted WAF fuzzing loop
# Concept: generate payloads with AI, test them, feed results back

TARGET="$1"
ENDPOINT="$2"

# Step 1: AI generates payload list
# (you run this manually or via script with API)
echo "Generate 20 WAF bypass payloads for multipart boundary parsing" | \
  # your AI API call here → payloads.txt

# Step 2: Test each payload
while read PAYLOAD; do
  RESPONSE=$(curl -s -o /dev/null -w "%{http_code}" \
    -H "Content-Type: multipart/form-data; boundary=x" \
    --data-binary "$PAYLOAD" \
    "$TARGET$ENDPOINT")
  echo "$PAYLOAD → $RESPONSE"
  
  # Step 3: If response different from 403 (WAF block), flag it
  if [ "$RESPONSE" != "403" ]; then
    echo "INTERESTING: $RESPONSE for payload: $PAYLOAD" >> interesting.txt
  fi
done < payloads.txt

# Step 4: Feed interesting results back to AI
# "These 3 payloads bypassed. Generate 50 more variations on these patterns."
```

### Caido Automate Version

Even better — use Caido's fuzzing engine:

1. Create a base request in Caido Replay
2. Mark payload positions (boundary value, charset, header values)
3. Load AI-generated wordlists as payloads
4. Run the Automate session
5. Filter results by response code/body length anomalies
6. Feed anomalies back to AI for analysis

---

## Method 5: The "Both Sides" Technique (Most Reliable)

This is the React2Shell researchers' actual workflow. It's the most reliable approach.

### The workflow

```
For a given HTTP feature (e.g., multipart boundary parsing):

1. Read how the RFC defines it
   ↓
2. Find 2+ implementations of it (WAF's lib, backend's lib)
   ↓
3. Feed BOTH implementations to AI side by side
   ↓
4. AI spots the differential
   ↓
5. Test it against the real WAF
```

### Concrete example for Cloudflare + Next.js

```bash
# 1. Find Cloudflare's multipart handling
# Cloudflare uses workerd (open source runtime)
git clone https://github.com/cloudflare/workerd.git
# Look at src/workerd/api/http.c++ for header parsing

# 2. Find Next.js/Node.js multipart handling  
# Next.js uses busboy
# Source: https://github.com/mscdex/busboy

# 3. Feed both to AI
```

**Key insight**: Many WAFs use the Go standard library `mime/multipart`. Many backends use busboy (Node), cgi.FieldStorage (Python), or Spring's MultipartResolver (Java). These all implement RFC 7578 slightly differently. AI will find the gaps.

---

## Quick-Start: Copy-Paste Material

### AI prompt for multipart boundary differentials

```
Compare the multipart boundary parsing in these two codebases:

CODEBASE A (WAF): [paste boundary extraction code]
CODEBASE B (Backend): [paste boundary extraction code]

Specific edge cases to check:
1. What if boundary parameter appears twice? (boundary=x; boundary=y)
2. What if boundary has surrounding quotes? (boundary="x y")
3. What if boundary has leading/trailing whitespace?
4. What if boundary contains special characters like = or ;?
5. What if Content-Type header appears twice with different boundaries?
6. What if boundary= is followed by nothing?
7. What if boundary parameter has UTF-8 or non-ASCII characters?

OUTPUT FORMAT:
For each codebase, explain exactly what boundary value it would extract.
Flag where they differ.
```

### AI prompt for charset differentials

```
I have a WAF that scans raw HTTP body bytes for malicious patterns.
The backend uses these Content-Type handlers:
- multipart/form-data → busboy 1.6.x
- application/json → JSON.parse
- application/x-www-form-urlencoded → qs 6.x

Question: How can I encode my payload so the WAF sees harmless bytes 
but the backend decodes it into a malicious payload?

Consider:
- charset=utf16le → WAF sees ASCII, backend sees UTF-16
- charset=utf7 → 
- Content-Transfer-Encoding: base64 within multipart parts
- Unicode normalization (NFKC collapsing lookalike chars)
- Gzip/brotli Content-Encoding at different points in the chain
```

### AI prompt for header duplication

```
How do these technologies handle duplicate HTTP headers?

1. Cloudflare WAF
2. NGINX
3. Node.js http module
4. Go net/http
5. Python werkzeug

For each pair, what happens with:
- Duplicate Content-Type → which value wins?
- Duplicate Content-Length → which value wins?
- Duplicate Transfer-Encoding → which value wins?
- What about duplicate headers with different casing? (content-type vs Content-Type)

I want to construct requests where the WAF sees one value and the backend sees another.
```

---

## What AI Can't Do (Yet)

From the React2Shell research and practical experience:

| Capability | AI Strength | AI Weakness |
|------------|-------------|-------------|
| Find differential between two known parsers | Excellent | — |
| Generate test payloads for a known differential | Excellent | — |
| Identify which RFC edge cases are ambiguous | Good | — |
| Guess the WAF technology from response patterns | Medium | Unreliable |
| Black-box bypass without parser info | Poor | Often wrong |
| Test payloads itself (needs sandbox) | Poor | Needs you to run tests |
| Creativity in novel attack chains | Medium | Still needs human intuition |

**Bottom line**: AI is a force multiplier for the analysis, not a replacement for the recon. Your job is to map the territory. AI's job is to find the cracks.

---

## Your Workflow

```
RECON (you do this)
  ↓
Find exact WAF technology
Find exact backend framework + version
Find parser libraries used by both
  ↓
ANALYSIS (AI does this)
  ↓
Feed both parsers to AI
AI identifies differentials
AI generates test payloads
  ↓
VERIFICATION (you do this)
  ↓
Test payloads against real target
Record which bypass
  ↓
ITERATION (you + AI)
  ↓
Feed results back to AI
AI refines and generates more
  ↓
WEAPONIZATION (you do this)
  ↓
Combine bypass + actual vulnerability
Submit report
```
