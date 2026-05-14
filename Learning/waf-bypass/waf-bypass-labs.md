# Where to Practice WAF Bypass

Every option below lets you test real WAF bypasses without burning bug bounty targets.

---

## Tier 1: Self-Hosted Labs (Best — full control, free)

### Lab 1: ModSecurity + CRS (OWASP Default Rules)

The most widely deployed WAF. If you can bypass CRS, you can bypass most corporate WAFs.

```bash
# Spin it up
docker pull owasp/modsecurity-crs:apache
docker run -d --name modsec-lab -p 8082:80 \
  -e PARANOIA=4 \
  -e BLOCKING=on \
  -v ~/lab-rules:/etc/modsecurity.d/owlh \
  owasp/modsecurity-crs:apache

# Backend is Apache + a test app with intentional vulns
# Visit http://localhost:8082/ — see the test app
```

Practice targets on this lab:
- SQLi bypass through the built-in DVWA-style app
- XSS bypass through search/comment forms
- Path traversal bypass through file include endpoints
- Command injection through ping/debug endpoints

### Lab 2: Coraza WAF (Go, open source — you can read the parser)

This is the best lab for white-box AI-assisted bypass. Coraza is used in production by many.

```bash
# Clone and build
git clone https://github.com/corazawaf/coraza.git ~/lab/coraza
cd ~/lab/coraza

# Stand up with a vulnerable Go backend
# Create test server that runs Coraza in front
```

**Why Coraza is perfect for AI practice:**
- Full Go source code for the HTTP parser → feed to AI
- You can add logging to see EXACTLY what the WAF sees
- You can modify it to test "what if the WAF did X"
- Compare Coraza parser vs `busboy` parser vs Go `mime/multipart`

### Lab 3: Flask + Custom "WAF" Middleware

Build both sides yourself — this is how you truly understand parser differentials.

```python
# ~/lab/flask-waf-lab/app.py
from flask import Flask, request
import re

app = Flask(__name__)

# --- SIMULATED WAF (strips/checks before Flask sees it) ---
@app.before_request
def fake_waf():
    """A naive WAF that checks raw bytes"""
    raw = request.get_data(cache=True)
    
    # Rule: block :constructor anywhere in body
    if b':constructor' in raw:
        return "WAF: BLOCKED", 403
    
    # Rule: block <script> tags
    if b'<script>' in raw.lower():
        return "WAF: BLOCKED", 403
    
    # Rule: block SELECT/UNION in form body
    if b'SELECT' in raw.upper() or b'UNION' in raw.upper():
        return "WAF: BLOCKED", 403
    
    # ... now what if the request has charset=utf16le?
    # Flask will decode it differently. That's your bypass.

@app.route('/api/upload', methods=['POST'])
def upload():
    # Flask's parser already decoded the multipart form
    # This runs AFTER the "WAF" check above
    data = request.form.get('payload', '')
    return f"Backend received: {data}"

@app.route('/api/search', methods=['GET'])
def search():
    q = request.args.get('q', '')
    # Vulnerable to SQLi but "WAF" above should block it
    return f"Search results for: {q}"

if __name__ == '__main__':
    app.run(port=5001, debug=True)
```

```bash
# Run it
pip install flask
python ~/lab/flask-waf-lab/app.py

# Now practice:
# 1. curl normally → WAF blocks
# 2. curl with utf16le → WAF sees raw bytes, Flask decodes → BYPASS
# 3. Add more WAF rules, find more bypasses
# 4. Feed the WAF code + Flask source to AI → find differentials
```

### Lab 4: Multi-WAF Docker Compose

Run several WAFs side by side, test same payload against all.

```yaml
# ~/lab/waf-lab/docker-compose.yml
version: '3'
services:
  backend:
    image: vulnerables/web-dvwa
    ports: []  # not directly exposed
    
  modsecurity:
    image: owasp/modsecurity-crs:nginx
    ports: ["8082:80"]
    environment:
      - BACKEND=http://backend:80
      - PARANOIA=4
  
  # For a Go-based WAF comparison
  coraza:
    build: ./coraza-proxy
    ports: ["8083:8080"]
    environment:
      - BACKEND=http://backend:80
```

```bash
# Test same payload against both WAFs
curl -H "Content-Type: multipart/form-data; boundary=x; boundary=y" \
  --data-binary @payload.bin \
  http://localhost:8082/api → modsecurity result
  
curl [same request] http://localhost:8083/api → coraza result
```

---

## Tier 2: Online Platforms (Guided, structured)

### PortSwigger Web Security Academy (Free)
```
https://portswigger.net/web-security
```
Specific labs for what you want:
- **HTTP Request Smuggling** — 20+ labs (CL.TE, TE.CL, H2.CL)
- **Authentication** — WAF-like rate limiting bypass
- **SQL Injection** — blind SQLi with WAF evasion
- **File Upload** — Content-Type restriction bypass
- All free, all have hints and solutions to learn from

### HackTheBox (Paid, ~$14/mo)
```
Filter by tags: "WAF", "request smuggling", "filter evasion"
Best machines:
- "Catch" — custom WAF bypass
- "Armageddon" — Drupal filter bypass
- "Forge" — SSRF with URL parser differential
```

### TryHackMe (Free tier available)
```
Rooms:
- "WAF" — basics of WAF bypass
- "OWASP Top 10" — practice with ModSecurity in place
- "HTTP Request Smuggling" — CL.TE and TE.CL hands-on
```

### PentesterLab (Paid, ~$20/mo)
```
Exercises tagged "WAF" and "filter bypass"
Has guided exercises specifically for:
- Encoding bypasses
- Protocol-level attacks
- Signature evasion
```

---

## Tier 3: Bug Bounty Targets (Real, but careful)

Programs where you can ethically test WAF bypass — these have known WAFs:

| Program | WAF | Backend | Good for practicing |
|---------|-----|---------|---------------------|
| **HackerOne itself** (disclosed targets) | Varies | Varies | Read disclosed reports to reverse-engineer bypasses |
| **Vercel customers** | Vercel WAF | Next.js | Multipart boundary tricks |
| **Cloudflare customers** | Cloudflare | Varies | Request smuggling, header tricks |
| **Shopify** (high reputation needed) | Custom | Rails | Encoding differentials |
| **GitHub** | Custom (Go) | Rails | Parser differentials |

**The playbook for bug bounty practice:**

1. Pick a program with a known WAF (fingerprint it first)
2. Find an endpoint that accepts user input (upload, search, profile edit)
3. Send benign payloads → confirm they reach backend
4. Send malicious payloads → confirm WAF blocks them
5. NOW start testing bypasses — you have a live WAF to beat
6. If you find a bypass, report it (even a WAF bypass alone can be a finding on some programs)

**Important**: Don't spray random bypass payloads at production. Be surgical. One test at a time. Watch for rate limiting.

---

## Tier 4: CTF Challenges

### Current CTFs with WAF bypass challenges
```
- CTFtime.org → search "WAF" in past challenges
- HackerOne CTF (every year, always has WAF challenges)
- Web Security CTF by PortSwigger (annual)
```

### Classic CTF challenges to replay
```
- "Bypass WAF" on Root-Me.org (free)
- "NoSQLi" challenges (WAF blocking SQL but not NoSQL)
- Any challenge tagged "filter" or "blacklist" or "parser"
```

---

## Your Practice Path

Recommended order:

```
Week 1-2: ModSecurity Docker lab
  → Learn the feel of a WAF blocking you
  → Reproduce all Category A-D techniques from the catalog
  → Goal: 10 verified bypasses

Week 3: Flask WAF lab (build your own WAF)
  → Understand what a WAF actually does internally
  → Practice charset tricks (utf16le, encoding)
  → Goal: bypass your own WAF 5 different ways

Week 4: PortSwigger Smuggling Labs
  → Master CL.TE, TE.CL, H2.CL
  → Goal: complete all 20+ labs

Week 5: Coraza + AI
  → Feed Coraza parser + busboy to AI
  → Generate novel differentials
  → Test them in the Coraza lab
  → Goal: 3 AI-discovered bypasses

Week 6+: Bug bounty targets
  → Apply everything to real programs
  → Start with low-severity endpoints
  → Combine bypass + vulnerability
```

---

## Quick Commands to Start Today

```bash
# Spin up ModSecurity right now
docker pull owasp/modsecurity-crs:nginx
docker run -d --name waf-practice -p 8082:8080 owasp/modsecurity-crs:nginx

# Visit http://localhost:8082
# Default test: curl 'http://localhost:8082/?q=<script>alert(1)</script>'
# Should get 403 Forbidden

# Now try your first bypass:
# curl 'http://localhost:8082/?q=%3C%73%63%72%69%70%74%3Ealert(1)%3C%2F%73%63%72%69%70%74%3E'
# (URL-encoded <script> — does it bypass?)

# Try unicode normalization:
# curl 'http://localhost:8082/?q=%EF%BC%9Cscript%EF%BC%9Ealert(1)%EF%BC%9C/script%EF%BC%9E'
# (fullwidth angle brackets — does ModSecurity normalize them?)
```
