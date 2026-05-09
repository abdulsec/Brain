# Assetnote Research — 0-Day & N-Day Hunting Cheat Sheet

> Distilled from ~70 Assetnote research posts (assetnote.io/resources/research).
> Goal: a transferable playbook for finding pre-auth criticals in vendor software.

---

## 1. The Assetnote Mindset (How They Find Bugs)

1. **Hunt the gated, the boring, the enterprise.** "Gated enterprise software receives minimal security research attention" — Ivanti, Citrix, VMware, Sitecore, Oracle Opera, Avaya, WatchGuard, Aspera. If it costs $50k/yr and has no demo, it's underaudited.
2. **Get the source.** Publicly downloadable installers (Sitecore, Oracle Opera, Yellowfin, Metabase, MOVEit), license trials, GitHub leaks, extracted firmware (Citrix nsppe, FortiGate FortiOS, Pulse Secure initrd LUKS key), decompiled WAR/JAR/APK/DLL. Every blog starts with `dnSpy / ILSpy / Ghidra / unzip APK / mount initrd`.
3. **Patch-diff is king (N-days → 0-days).** Bindiff/diaphora on patched binaries. Each citrix/fortigate/moveit blog finds the bug from the *fix*. Look for added length checks, added `if` validation, added authorization annotation.
4. **Continuous attack surface monitoring** finds 0-days too — Mozilla AWS WebPageTest, Uber ExactTarget, Zendesk dotfiles, dangling Elastic IPs, Azure DevOps zone takeover.
5. **Sink-to-source code review.** Find a dangerous function (`Runtime.exec`, `BinaryFormatter.Deserialize`, `YAML.load`, `eval`, `entityManager.createQuery`, `file_get_contents`, `Process.Start`, raw subprocess `shell=True`) — then trace backwards to an unauth endpoint. This is the #1 repeated methodology.
6. **Pre-auth obsession.** A post-auth bug + a separate auth bypass / open registration / credential leak / SSRF chain = pre-auth criticality. Always look for the chain.

---

## 2. Vulnerability Class Playbook

### A. Insecure Deserialization (recurring goldmine)
| Pattern | Where to look | Example |
|---|---|---|
| `BinaryFormatter.Deserialize` (.NET) | Custom HTTP modules, .ashx handlers, file upload endpoints, Telerik | Sitecore Report.ashx, WS_FTP MyFileUpload, MOVEit, Sitecore 9.3 |
| `NetDataContractSerializer.ReadObject` (.NET) | XML deserialization handlers | CVE-2021-42237 Sitecore |
| `typeFilterLevel="Full"` in web.config | .NET Remoting, SOAP `.rem` endpoints | Citrix Session Recording CVE-2023-6184 |
| `YAML.load` (Ruby) | Rails controllers w/ user params | Aspera Faspex CVE-2022-47986 |
| `ObjectInputStream.readObject` (Java) | T3, JMX, RMI, custom protocols | (classic) |
| Nested JSON → constructor reflection (PHP) | Magento WebAPI, Laravel | Magento XXE CVE-2024-34102 |
| **Tools:** ysoserial, ysoserial.net, marshalsec, BurpExtension JavaSerialKiller, phpggc | | |

### B. SSRF (with severity multipliers)
- **Where:** image proxies, webhook handlers, OAuth metadata fetchers, render endpoints, "fetch URL" features.
- **Bypasses:** `@` in URL (`https://trusted@evil`), `[::]`, `0.0.0.0`, decimal IPs, DNS rebinding, redirect chain via whitelisted domain (Lotus Domino `?Logout&RedirectTo=`).
- **Multipliers:** 169.254.169.254 (cloud creds), Redis/Memcache via gopher://, internal admin panels, JWT leak via Authorization-forwarded request.
- **Notable patterns:**
  - User-controlled `Host` / `X-Forwarded-Proto` / `X-Forwarded-Host` headers reaching server-side fetch (Next.js SSRF, Netlify IPX, VMware 22972 auth bypass).
  - Hardcoded encryption keys decrypting user-supplied URL into HTTP fetch (VMware UEM CVE-2021-22054, ShareFile).
  - "imageUrl" / "logoUrl" / "url" parameters with no SSRF filter (Jamf Pro, Confluence, Jira).
  - SAML `RetrievalMethod` / xmltooling (Pulse Secure CVE-2024-21893).

### C. Path / Auth Bypass via Path Confusion
| Trick | Example |
|---|---|
| Double URL encoding `%252e%252e/` between proxies | PAN-OS CVE-2025-0108 (Nginx→Apache) |
| Prefix-only `strncmp` validation | Pulse Secure CVE-2023-46805 (`/api/v1/totp/user-backup-code/../`) |
| Fragment `#` strips after extension check | Sitecore order-of-ops `?f=...web.config%23.js` |
| Server.Execute() reuses authenticated context | Sitecore 9.3 |
| Header-based auth (`X-pan-AuthCheck: off`) leaking through proxies | PAN-OS |
| H2C upgrade smuggling past WAF | h2c smuggling research |
| Trailing slash, `;`, `..;/`, `/./` differential parsing | (universal) |

### D. Order-of-Operations Bugs
**Sanitize-before-decrypt vs decrypt-before-sanitize. Validate-before-strip-fragment. `&&` vs `||`.**
- Oracle Opera CVE-2023-21932: sanitization before decryption → bypass with encrypted payload.
- Sitecore CVE-2024-46938: extension check before fragment removal.
- Dynamicweb CVE-2022-25369: `&&` instead of `||` in setup re-run check.
- ShareFile CVE-2023-24489: decryption success ≠ authentication (no MAC) — brute PKCS#7 padding.
- Lesson: **whenever you see encrypt+sanitize+validate, ask "what order, and what happens when one step throws?"**

### E. Hardcoded Secrets / Default Keys
Grep installer for: `private`, `secret`, `BEGIN RSA`, `key=`, `salt=`, hex strings >32 chars, base64 blobs >40 chars, `MachineKey`, `validationKey`, `decryptionKey`.
- Yellowfin BI (3 hardcoded keys), VMware UEM, SolarWinds WHD creds, Metabase setup token, Sitecore machineKey from web.config backup.
- **JWT secret hardcoded** = forge admin tokens.
- **MachineKey leaked** = ViewState RCE.

### F. Memory Corruption (binary, less common but critical)
- **`snprintf` return-value misuse** → over-read leak (Citrix Bleed CVE-2023-4966): oversized Host header.
- **Unbounded `strcat`** → heap overflow into adjacent SAX struct (WatchGuard CVE-2022-26318).
- **Missing bounds check on parsed length field** (FortiGate CVE-2024-21762 — 2-byte chunked encoding overflow; Citrix CVE-2024-8534 RDP TPKT length).
- **Stack overflow, no canaries/ASLR/NX** (Citrix NetScaler CVE-2023-3519 — `ns_aaa_saml_url_decode`).
- Workflow: BinDiff → spot added length check → fuzz that input → crash → exploit.

### G. Argument / Command Injection
- Null-byte injection in NuProcessBuilder (Bitbucket CVE-2022-36804 — `prefix=x%00--exec=...`).
- `subprocess(... shell=True)` with user input (Pulse Secure license, Cloudflare Pages build).
- Azure DevOps logging command injection (`##vso[task.setvariable]`).

### H. Server-side Template Injection / Eval
- AMPScript `LookupRows` (Uber ExactTarget — $23k).
- Jelly two-phase `${}` then `$[]` (ServiceNow).
- Twig filter chain bypass (`['system','id']|sort('call_user_func')`) — Craft CMS.
- HQL injection via `entityManager.createQuery` (SolarWinds WHD `/assetReport/rawHQL`).
- H2 SQL JDBC `TRACE_LEVEL_SYSTEM_OUT` for RCE (Metabase CVE-2023-38646).

### I. File Upload / Path Traversal
- Multipart `filename="../../../webapps/ROOT/x.jsp"` — DotCMS, Aspera, classic.
- ZIP extraction without canonicalization (WebPageTest, WebSphere Portal).
- Symlinks in archive.
- Race condition: extract → execute → before-cleanup (WebPageTest Mozilla — 200 Burp threads).

### J. XXE / Nested Deserialization
- `SimpleXMLElement` reachable through nested JSON object instantiation (Magento CVE-2024-34102).
- Default `xml:space` injection in Citrix StoreFront SAML.
- Unsafe XML signature `RetrievalMethod` external dereference (Pulse Secure xmltooling).

### K. Setup / Install Wizard Re-trigger
Always test `/setup`, `/install`, `/admin/access/setup`, `/initial`, `/wizard` post-deploy.
- Dynamicweb (`Action=createadministrator`).
- Metabase exposed setup-token via `/api/session/properties` after install.

### L. Cloud / Infra Class
- Dangling Elastic IPs / DNS zones → Ghostbuster, Azure DevOps `project-cascade.visualstudio.com` takeover.
- Subdomain takeover via Fastly / cPanel claim.
- DNS poisoning (Great Firewall pool of "censored" IPs claimable on Fastly).
- Kubelet API on :10255 from build pod (Cloudflare Pages part 3).
- React Native `index.android.bundle` grep for `apiKey` / Firebase creds.
- GitHub dotfile leaks (Zendesk).

### M. GraphQL
- Introspection always-on, schema suggestions in errors → Clairvoyance.
- Batched queries bypass rate-limit → BatchQL ("10k pin attempts in one request").
- CSRF via GET / `application/x-www-form-urlencoded`.

### N. Open Redirect / OAuth Reply-To
- Permissive `*.visualstudio.com` matching → Azure DevOps account takeover.
- CRLF in `post_logout_redirect_uri` → Citrix Gateway XSS CVE-2023-24488.

---

## 3. Hunting Workflow (the Assetnote loop)

```
1. PICK TARGET → enterprise vendor software, gated, costly, internet-facing
2. GET CODE    → trial download, license leak, firmware extract, GitHub, decompile WAR/DLL/APK
3. MAP SURFACE → web.xml / RouteConfig / @Path / urls.py / Startup.cs / proxy_pass /
                 servlet mappings, [AllowAnonymous], skipFilters, AuthorizeAttribute exclusions
4. SINK HUNT   → grep dangerous APIs (deserialize, eval, exec, query, file_get_contents,
                 ProcessBuilder, BinaryFormatter, fetch/HttpClient, MapPath, fopen)
5. SOURCE TRACE→ find unauth path → controllable parameters → reach sink
6. DIFF        → if N-day, bindiff patched vs unpatched; look for added validation
7. PoC         → minimal request, then weaponize chain to pre-auth
8. SCAN AT SCALE → Shodan/Censys/internetdb → scan with high-signal probe
                  (error oracle, version banner, harmless canary)
```

### High-signal pre-flight checks per target
- [ ] Does it ship with default creds / hardcoded keys?
- [ ] Any `/setup` re-run, `/install`, install-token leak?
- [ ] Any HTTP module / servlet filter with custom auth (often broken)?
- [ ] Any deserialization endpoint? (file upload, session, view-state)
- [ ] Reverse proxy in front? (Nginx→Apache, IIS→ARR, Cloudflare→origin) → path confusion
- [ ] Encrypted parameters with static key? (decrypt-before-validate trick)
- [ ] SAML / OAuth / SSO endpoint? (XML signature, RetrievalMethod, redirect_uri)
- [ ] Any image/URL fetch / webhook / proxy endpoint?
- [ ] GraphQL / batch endpoint?
- [ ] WebDAV / file PUT?

---

## 4. Toolkit (mentioned across the corpus)

**Static / RE**
- ILSpy, dnSpy, JetBrains dotPeek (.NET)
- jadx, JD-GUI, Bytecode Viewer, Procyon, CFR (Java)
- Ghidra + BinDiff, IDA + Diaphora, radare2 (binary)
- PHP: psalm, manual + Burp
- Ruby on Rails: routes.rb, manual

**Dynamic / Debug**
- gdbserver, lldb, Frida (FortiGate, WatchGuard, Citrix)
- pspy (process monitoring — Bitbucket null-byte discovery)
- attach debugger to JVM / IIS w3wp / dotnet

**Recon / Surface**
- Kiterunner (contextual API discovery from 67k Swagger files)
- IIS-ShortName-Scanner + BigQuery GitHub regex (find real names from 8.3)
- Ghostbuster (dangling IPs)
- gitrob / trufflehog (dotfile leaks)
- httpx, naabu, dnsx, subfinder

**Exploit**
- ysoserial, ysoserial.net, marshalsec
- Gopherus (SSRF→Redis/MySQL/SMTP gadgets)
- BatchQL, Clairvoyance (GraphQL)
- Burp Intruder (race conditions, padding oracle)

**Comms / Canaries**
- Burp Collaborator, interactsh, requestbin
- DNS / HTTP / SMB / LDAP callbacks for blind detection

---

## 5. Pattern Library — "If you see X, try Y"

| If you see... | Try... |
|---|---|
| `getServerName()` / `Host:` reaching server-side URL build | SSRF via Host injection, auth-bypass via redirected validation |
| Encrypted parameter + static key (config/DLL strings) | Decrypt → forge → bypass auth |
| `[AllowAnonymous]` on API controller | Sink hunt the controller's chain |
| `BinaryFormatter` / `NetDataContractSerializer` | ysoserial.net payload, look for any reachable input |
| `web.config` `typeFilterLevel="Full"` | .NET Remoting `.rem` deser RCE |
| `<%=` in JSP, `{{ }}` in Twig/Jinja, AMPScript tags | XSS / SSTI |
| YAML in Rails params | YAML.load gadget chain |
| H2/Postgres JDBC URL controllable | TRACE_LEVEL or COPY FROM PROGRAM |
| Two reverse proxies (Nginx+Apache, F5+IIS) | Path confusion, double-decode, ambiguous routing |
| Setup/install endpoint exists in prod | Re-trigger, create admin |
| `subprocess.run(... shell=True)` | Command injection via metachar in any input |
| Image/file upload with `filename` from multipart | Path traversal, polyglot, ZIP slip |
| `snprintf`/`strcat`/`memcpy` w/ length from header | Memory corruption (over-read or overflow) |
| OAuth/SAML `redirect_uri`/`reply_to` | Open redirect, suffix-match bypass, subdomain takeover |
| Hardcoded `MachineKey` in config | ViewState RCE |
| `getResourceAsStream` w/ user path | Java path traversal, classpath disclosure |
| `URI.create(base + userInput)` | `@` injection → SSRF |
| `Server.Execute()` in .NET | IIS auth bypass via internal redispatch |
| `mod_dav_fs` / WebDAV PUT | Direct shell upload |
| H2C upgrade allowed | Request smuggling past WAF |
| Frontend bundle (JS/RN) | grep `apiKey`, JWT, S3 URLs, internal hosts |

---

## 6. Recurring Lessons (the meta-rules)

1. **Pre-auth attack surface is bigger than the docs say.** Anonymous controllers, install wizards, error pages, `.well-known`, image proxies, OAuth flows.
2. **Encryption ≠ Authentication.** Without a MAC, encrypted parameters are forgeable / paddable / replayable.
3. **Defense layers must agree on path normalization.** Differential parsers = auth bypass.
4. **Order matters.** Sanitize→decode→validate must be in the right sequence.
5. **`snprintf` return value, prefix-only `strncmp`, suffix-only domain match, regex anchors** — all repeatedly broken.
6. **Patches are exploit roadmaps.** Add a length check → there was a memory bug. Add escaping → there was injection. Add authz → there was bypass.
7. **Vendor software in your bug bounty scope is rarely audited.** If `xyz.target.com` = WebSphere Portal / Sitecore / WhatsUp Gold / Avaya — pull the installer.
8. **Continuous monitoring beats one-shot pentests.** New subdomain → new vendor → new CVE class → re-run scanner.
9. **Chain post-auth → pre-auth.** Open signup, default creds, low-priv → admin via missing authz, admin → RCE via deser/upload.
10. **Don't guess — instrument.** `var_dump`, debugger, network trace will always beat pure static reading on complex frameworks (Magento, Sitecore, Citrix).

---

## 7. CVE Index (all blogs covered, by class)

**Deserialization RCE:** CVE-2021-42237 Sitecore, CVE-2022-47986 Aspera, CVE-2023-6184 Citrix Session Recording, CVE-2023-34362 MOVEit, CVE-2023-40044 WS_FTP, Sitecore 9.3 Telerik chain, Magento CVE-2024-34102.

**Pre-auth RCE (logic/injection):** CVE-2022-26352 dotCMS, CVE-2022-25369 Dynamicweb, CVE-2022-26318 WatchGuard, CVE-2022-36804 Bitbucket, CVE-2023-21932 Oracle Opera, CVE-2023-29489 cPanel XSS, CVE-2023-38646 Metabase, CVE-2024-21887 + 46805 Pulse Secure, CVE-2024-21762 FortiGate, CVE-2024-21893 Pulse SAML, Yellowfin BI (47882/47884/47885), Craft CMS PHP footgun, Avaya AADS WebDAV.

**Auth bypass:** CVE-2022-22972 VMware WS1 Access (Host hdr), CVE-2025-0108 PAN-OS path confusion, CVE-2025-29927 Next.js middleware, Sitecore 9.3 Server.Execute, Pulse Secure prefix-only strncmp.

**SSRF:** CVE-2021-22054/22056 VMware, CVE-2021-27748 WebSphere, CVE-2021-39303/40809 Jamf, CVE-2022-26135 Jira, CVE-2024-34351 Next.js, Netlify IPX, Confluence/WebLogic glossary, ServiceNow Jelly.

**Memory corruption:** CVE-2023-3519 Citrix (3 parts), CVE-2023-4966 Citrix Bleed, CVE-2024-8534 Citrix RDP, CVE-2024-21762 FortiGate, CVE-2022-26318 WatchGuard.

**Information / takeover:** Mozilla AWS WebPageTest, Uber AMPScript ($23k), Zendesk dotfile, Azure DevOps zone takeover, Cloudflare Pages 1/2/3, dangling EIPs, IIS shortname + BigQuery, GFW DNS poisoning.

**Templating / SSTI:** ServiceNow Jelly, Uber AMPScript, Craft CMS Twig.

**XSS / Client-side:** CVE-2023-24488 Citrix CRLF, CVE-2023-29489 cPanel, CVE-2023-5914 Citrix StoreFront, Avaya AADS, Netlify SVG.

---

## 8. Daily Drill (build the muscle)

- **Mon:** Pick one CVE from the corpus. Reproduce in a lab. Patch-diff if applicable.
- **Tue:** Pick a vendor in your scope. Pull installer/firmware. Decompile.
- **Wed:** Sink-grep one dangerous API across the codebase. Trace one source-to-sink path.
- **Thu:** Pick one class (e.g., XXE). Find every parser instantiation in target. Test each with malicious DTD.
- **Fri:** Surface scan: new subdomains, new TLS certs, JS bundle diffs. Re-run httpx + nuclei tag for last week's CVEs.
- **Weekend:** Read 2 new Assetnote / ProjectDiscovery / WatchTowr blogs. Add to this cheat sheet.

---

*Source corpus: every public post on https://www.assetnote.io/resources/research as of 2026-05-09. Original write-ups credit Shubham Shah, Adam Kues, Adam Crosser, Dylan Pindur, Max Garrett, Nathan Wilson, et al. — read the originals for code-level detail.*
