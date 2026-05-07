# Enterprise Tech Stack — A Bug Hunter's Guide

> Goal: Understand what Fortune 500 / top companies actually run so you know what you're attacking, what's in front, what's behind, and where misconfigurations live.

---

## 1. DNS & Traffic Routing

### Public DNS Providers
| Provider | Who Uses It | Bug Notes |
|----------|------------|-----------|
| **Route 53 (AWS)** | The default for anything on AWS | Zone takeover if CloudFront/S3 origin is deleted but DNS remains |
| **Cloudflare DNS** | Common in tech/SaaS | Always check if origin IP leaks (securitytrails, censys, shodan) |
| **Akamai Edge DNS** | Banking, insurance, enterprise | Same origin-leak concern |
| **Azure DNS** | Microsoft-heavy enterprises | Often paired with Azure Front Door |
| **Google Cloud DNS** | GCP-native companies | Less common externally |

### What matters for recon:
- **DNS zone transfers** — rare but try `dig AXFR target.com @ns1.target.com`
- **Subdomain enumeration** alone finds 30% of attack surface (cert transparency, brute-force, passive sources)
- **Origin IP leaks** — if you find the real IP, you bypass the CDN/WAF entirely (search historical DNS on SecurityTrails)
- **NS record delegation** — a dangling delegation to an unregistered nameserver = subdomain takeover

---

## 2. CDN (Content Delivery Network)

### Tier 1: The Giants
| CDN | Market | How to Fingerprint | Attack Surface |
|-----|--------|-------------------|----------------|
| **Cloudflare** | Dominates tech/SaaS/startups | `cf-ray` header, `server: cloudflare`, cert SAN `*.cloudflare.com` | Cache poisoning, workers abuse, unprotected origins via direct IP |
| **Akamai** | Banking, insurance, Fortune 500 legacy | `x-akamai-request-id`, `server: AkamaiGHost`, edge cookies (`aka_...`) | ESI injection, caching rules abuse |
| **Fastly** | Media, e-commerce, real-time | `x-served-by`, `x-cache: HIT/MISS`, `Fastly-*` headers, `surrogate-control` | Surrogate key manipulation, VCL bypass |
| **Amazon CloudFront** | AWS-native shops | `x-amz-cf-id`, `x-amz-cf-pop`, `via: ... CloudFront` | S3 bucket misconfig, OAI bypass, cache poisoning via query strings |
| **Azure Front Door** | Microsoft-heavy enterprise | `x-azure-ref`, `x-ms-*` headers | Origin bypass, routing rule abuse |

### Tier 2: Others
- **EdgeCast/Verizon** (legacy media), **StackPath** (gaming), **KeyCDN/BunnyCDN** (price-sensitive), **Cloudflare R2** (storage CDN)
- **Imperva/Incapsula** — enterprise WAF + CDN combo, common in financial services

### What matters for recon:
- **Cache poisoning** — does the CDN cache based on unkeyed headers/query params? (`X-Forwarded-Host`, `X-Forwarded-Scheme`, arbitrary query strings)
- **Origin bypass** — find the origin IP via historical DNS, SSRF, or leaked certs → hit it directly, no WAF
- **Request smuggling through CDN** — different HTTP parsing at CDN vs origin = TE.CL / CL.TE

---

## 3. DDoS Protection / Scrubbing

| Provider | Who Uses It | Notes |
|----------|------------|-------|
| **Cloudflare Magic Transit** | Tech/SaaS | L3/L4 protection, BGP-based |
| **Akamai Prolexic** | Banks, exchanges, gambling | The original DDoS filter for finance |
| **AWS Shield Advanced** | AWS-native | Auto WAF rules, cost protection |
| **Google Cloud Armor** | GCP-native | Adaptive protection, reCAPTCHA integration |
| **Radware** | Telcos, hosting providers | Hardware + cloud hybrid |
| **Arbor Networks/NETSCOUT** | ISP-level | Not something you'll see as a bug hunter |

### Bug context:
- DDoS scrubbing means traffic passes through a filtering layer — another hop that can be fingerprinted
- **Rate limiting bypass** — some scrubbing services use weak rate-limit signatures

---

## 4. External Reverse Proxy & Load Balancer

This is the first thing that actually processes your HTTP request at the target's edge.

### On-Prem Hardware (Legacy Fortune 500)
| Appliance | Where It Lives | Bug Context |
|-----------|---------------|-------------|
| **F5 BIG-IP** | Every bank, telco, government DC | **Gold mine.** TMUI RCEs (CVE-2020-5902), iRule bypass, APM/ASM misconfig, cookie persistence (`BIGipServer~...` leaks internal IP) |
| **Citrix ADC (NetScaler)** | Healthcare, gov, finance | CVE-2019-19781 (path traversal), CVE-2022-27518 (auth bypass), `ns_gateway` in responses |
| **A10 Networks Thunder** | Telcos, asian markets | Less target but same class, cookie-based persistence |
| **Barracuda WAF/LB** | Mid-market | Cookie injection, WAF bypass via encoding |

### Software / Cloud-Native
| Software | Where It Lives | Bug Context |
|----------|---------------|-------------|
| **NGINX** (open source & Plus) | **Everywhere.** Web, APIs, K8s ingress, sidecar | Path traversal (`/../` alias misconfig), CRLF injection, SSI injection, `$uri` vs `$request_uri` injection, `proxy_pass` trailing-slash issues |
| **HAProxy** | High-connection-count internal LB, some edge | HTTP request smuggling (CVE-2021-40346), `http-request` rule bypass |
| **Apache httpd** (mod_proxy, mod_proxy_balancer) | Legacy web LB | Path confusion, mod_proxy SSRF, `ProxyPass` misconfig |
| **Envoy** | K8s service mesh edge, API gateways | Header smuggling, path normalization differences from NGINX, CVE-2021-32779 (OAuth filter bypass) |
| **Traefik** | Smaller K8s deployments, Docker | Insecure default dashboard, middleware bypass |
| **Caddy** | Small/medium, automatic HTTPS | Less battle-tested in enterprise, cert management issues |

### How to Fingerprint
- `Server` header (NGINX, Apache, BigIP, etc.)
- Cookie patterns: `BIGipServer~poolname=IP.port.0000` = F5
- Response header ordering (NGINX orders headers differently than Apache)
- Error pages (`/doesnotexist`, 404 body, `/nginx-status` leak)
- TLS fingerprinting (JA3/JA4)

---

## 5. WAF (Web Application Firewall)

| WAF | Common Deployment | Bypass Difficulty | Notes |
|-----|------------------|-------------------|-------|
| **Cloudflare WAF** | Everywhere on CF plans | Medium | Managed rules + custom rules. OWASP core ruleset base. Bypass via encoding tricks, HTTP/2, newline smuggling |
| **AWS WAF** | AWS-native | Medium | Rules you write + managed rules. Bypass via oversized body, encoding |
| **Akamai Kona** | Enterprise/finance | Medium-High | Custom rules per client, strict |
| **F5 Advanced WAF (ASM)** | On-prem enterprise | Medium | Cookie signing, CSRF tokens, learning mode gaps |
| **Imperva SecureSphere** | Finance, healthcare | High | Hard WAF, but HTTP/2 smuggling works |
| **Fortinet FortiWeb** | Mid-market enterprise | Medium | Regex bypasses in custom rules |
| **ModSecurity + CRS** | Open-source, many hosting providers | Low-Medium | Default OWASP CRS has known bypasses (encoding, multipart, JSON). Widespread false positives → many run in detection-only mode. |
| **NAXSI** (NGINX WAF) | Small/medium | Low | Score-based, not signature-based. Easy to stay under threshold. |

### WAF Bypass Fundamentals:
1. **Encoding**: URL-encode, double-encode, Unicode normalize, UTF-8 overlong
2. **Parser differentials**: CDN parses differently from origin, WAF differently from app server
3. **HTTP/2 → HTTP/1.1 downgrade**: WAF on HTTP/1.1 might not see what origin receives
4. **Request smuggling**: TE.CL / CL.TE to smuggle payloads past WAF
5. **Parameter pollution**: `?id=1&id=1'+OR+'1'='1` — WAF sees first, app parses last
6. **Content-Type tricks**: Send JSON in a multipart form, WAF doesn't parse deeply
7. **Line folding**: `\n\t` in headers, obsolete HTTP features that WAFs miss

---

## 6. API Gateway

### The Gateways
| Gateway | Where | Attack Surface |
|---------|-------|---------------|
| **Kong** | Cloud-native, K8s | Plugin misconfig (auth, rate-limit), admin API exposure (`:8001`), JWT key confusion |
| **Apigee (Google)** | Traditional enterprise, banking | OAuth flow misconfig, proxy path traversal, developer portal leaks |
| **AWS API Gateway** | AWS-native | Lambda cold-start timing side channels, custom authorizer bypass, API key leak in client code |
| **Azure API Management** | Azure-native | Policy misconfig, subscription key leak, CORS over-permissive |
| **Tyk** | Smaller cloud-native | Gateway admin API exposure, plugin bypass |
| **Ambassador / Emissary** | K8s, Envoy-based | Mapping config exposure, auth bypass |
| **Gloo Edge** | K8s, solo.io | VirtualService abuse, auth misconfig |
| **NGINX as API Gateway** | Common DIY | Same NGINX issues + JWT validation bypass |

### API-Specific Bug Classes:
- **BOLA/IDOR** — the #1 API bug. Gateway adds auth, but app doesn't enforce ownership
- **Mass assignment** — API silently binds fields the UI doesn't show
- **JWT algo confusion** — `alg: none`, RS256 → HS256 key confusion
- **GraphQL** — introspection leak, deep query DoS, field-level auth gaps
- **gRPC** — reflection service leak, no auth on internal methods
- **OpenAPI/Swagger leak** — `/swagger.json`, `/api-docs`, `/v2/api-docs`, `/v3/api-docs`, `/openapi.json`
- **API versioning** — `/v1/...` has auth but `/v2/...`, `/internal/`, `/beta/`, `/legacy/` don't

---

## 7. Web Servers & App Servers

### Web Servers (serve static, reverse proxy to app)
| Server | Fingerprint | Key Bugs |
|--------|------------|----------|
| **NGINX** | `Server: nginx`, default 404 page | Path traversal (`alias` directive), CRLF in `$uri`, `proxy_pass` SSRF, `X-Accel-Redirect` internal access |
| **Apache httpd** | `Server: Apache`, default directory listing style | Path traversal (`mod_alias`), `mod_proxy` SSRF (CVE-2021-40438), `.htaccess` leaks, `mod_status` leak |
| **IIS (Microsoft)** | `Server: Microsoft-IIS/X.X`, ASP.NET headers | `..;` path traversal (CVE-2023-1397), `web.config` leak, Tilde shortname enumeration (~1 technique), MS15-034 HTTP.sys |
| **LiteSpeed** | `Server: LiteSpeed`, LSWS headers | Less targeted research = maybe more undiscovered bugs |
| **Caddy** | `Server: Caddy`, JSON config | Auto-HTTPS misconfig, reverse proxy config exposure |
| **OpenResty** | NGINX + LuaJIT, `X-Powered-By: OpenResty` | Lua code injection, `ngx.location.capture` SSRF |
| **Tomcat** (Java) | `X-Powered-By: Servlet/...`, default error pages | `/manager/`, `/host-manager/` weak creds, CVE-2025-xxx deserialization, AJP exposure (`:8009`) |
| **Jetty** (Java) | `Server: Jetty` | Less targeted, path traversal in specific versions |
| **Gunicorn / uWSGI** | Python app servers behind NGINX | uWSGI protocol abuse via `uwsgi_pass` SSRF |
| **Node.js (Express)** | `X-Powered-By: Express` | SSRF via `http-proxy-middleware` bypass, prototype pollution |
| **PHP-FPM** | Behind NGINX | `fastcgi_pass` SSRF → code execution (if you can inject into `SCRIPT_FILENAME`) |

---

## 8. Internal Load Balancing & Service Mesh

### East-West Traffic (service-to-service inside the DC/K8s)
| Technology | Usage | Bug Context |
|-----------|-------|-------------|
| **HAProxy** | Internal TCP LB, very high connections | Connection exhaustion attacks, `http-request` rule bypass |
| **NGINX (internal)** | Sidecar, internal proxy | Same class as external but often less hardened |
| **Envoy (via Istio)** | K8s service mesh standard | Sidecar bypass via direct pod IP, mTLS enforcement gaps, RBAC filter bypass |
| **Linkerd** | Lightweight K8s mesh | mTLS policy gaps |
| **Consul Connect** | HashiCorp ecosystem | Intent-based auth bypass, gossip encryption weak |
| **MetalLB / kube-proxy** | Bare-metal K8s LB, iptables/IPVS | NodePort exposure, BGP hijack in L2 mode |
| **AWS ELB / ALB / NLB** | AWS-native | ALB request smuggling, NLB source IP spoofing, ALB auth bypass |
| **GCP Cloud LB** | GCP-native | NEG misconfig, IAP bypass |
| **Azure Load Balancer** | Azure-native | NSG rule misconfig, health probe exposure |

### Service Mesh Attack Surface:
- **Sidecar bypass** — talk to the pod IP directly, skipping Envoy/mTLS
- **AuthorizationPolicy gaps** — Istio `ALLOW` policies that miss endpoints
- **mTLS downgrade** to plaintext where PERMISSIVE mode is set
- **Control plane exposure** — Istiod/Pilot API accessible by compromise one service = compromise the mesh

---

## 9. Backend Frameworks & Languages

### By Language, What's Deployed in Enterprise:

**Java (still the #1 enterprise backend)**
- **Spring Boot** — actuator endpoints leak (`/actuator/heapdump`, `/actuator/env`, `/actuator/mappings`), SpEL injection, `spring.datasource` config leak
- **Jakarta EE (J2EE legacy)** — deserialization everywhere
- **Struts** — OGNL injection (CVE-2017-5638 class). Legacy but still in banks
- **JSF (JavaServer Faces)** — viewstate deserialization

**.NET / C# (Microsoft enterprise)**
- **ASP.NET Core** — `swagger.json` leak, model binding over-posting, ViewState deserialization in legacy ASP.NET
- **SharePoint** — it's an entire extra attack surface (SSRF, deserialization, auth bypass)

**Python**
- **Django** — admin panel exposure (`/admin/`), debug mode info leak (`DEBUG=True` + invalid params → traceback + settings dump), `SECRET_KEY` in git → session forgery
- **Flask** — debug mode RCE (Werkzeug debugger PIN bypass), SSTI in Jinja2 templates, `SECRET_KEY` → flask-unsign
- **FastAPI** — `/docs`, `/openapi.json` leak, Pydantic validation bypass

**Node.js / JavaScript**
- **Express** — prototype pollution → RCE, `node-serialize` / `serialize-javascript` RCE, `eval()` / `Function()` injection
- **Next.js** — SSRF via image optimization (`/_next/image`), middleware bypass via `x-middleware-subrequest`, server actions auth gap

**PHP**
- **Laravel** — `.env` leak, debug mode RCE (CVE-2021-3129), `APP_KEY` → unserialize
- **Symfony** — profiler leak (`/_profiler`), secret in `APP_SECRET`
- **WordPress** — plugin hell, `xmlrpc.php`, REST API user enumeration

**Go**
- **net/http** — SSRF with redirect following, header injection via `\n` in values
- **Gin, Echo, Fiber** — path parameter injection, binding issues

**Ruby**
- **Ruby on Rails** — mass assignment (protected attrs bypass), `secret_key_base` → RCE (Marshal deser), SSTI in ERB

---

## 10. Databases & Caching

### Relational
| DB | Where | Attack Surface |
|----|-------|---------------|
| **PostgreSQL** | Tech, startups, modern enterprise | `COPY ... FROM PROGRAM` if superuser, SQL injection → `lo_import` file read, RCE via extensions |
| **MySQL / MariaDB** | Everywhere legacy | `LOAD_FILE()`, `INTO OUTFILE` webshell, UDF RCE if plugin dir writable |
| **Oracle DB** | Banking, insurance | TNS poisoning, Java stored procedure RCE, massive attack surface, hard to test without creds |
| **MS SQL Server** | Microsoft-heavy shops | `xp_cmdshell` RCE, `OPENROWSET` SSRF (link servers), `xp_dirtree` UNC path SMB relay |
| **SQLite** | Mobile, edge, embedded | Local file read via ATTACH, but usually not over network |

### NoSQL
| DB | Where | Attack Surface |
|----|-------|---------------|
| **MongoDB** | Startup, mid-market | **NoSQL injection** (`$gt`, `$ne`, `$regex` operators), unauthenticated default (port 27017, shodan it) |
| **Elasticsearch** | Logging, search | Unauthenticated access → dump entire index, grok script RCE, CVE-2014-3120 MVEL RCE |
| **Redis** | Caching, session store, queue | Unauthenticated default → write crontab/SSH key for RCE. `SLAVE OF` to set up rogue master. Very commonly exposed. |
| **Cassandra** | Big data, time-series | JMX exposure, auth disabled by default on old versions |
| **DynamoDB** (AWS) | Serverless apps | IAM policy over-permission, NoSQL injection via `FilterExpression` |
| **CouchDB** | Legacy | Unauthenticated HTTP API → Fauxton admin, replication to attacker-controlled instance |

### Caching Layer
| Cache | Attack Surface |
|-------|---------------|
| **Redis** | See above — often exposed. If it stores sessions → session forgery. If it stores serialized objects → deserialization RCE |
| **Memcached** | UDP amplification DDoS (not your bug), unauthenticated read → dump all keys/values, session hijack |
| **Varnish Cache** | Cache poisoning via unkeyed headers, ESI injection |
| **CDN edge caches** | Cache deception (tricking CDN into caching authenticated pages), cache poisoning |

---

## 11. Message Queues & Event Streaming

| Technology | Where | Attack Surface |
|-----------|-------|---------------|
| **Apache Kafka** | Big data pipeline, event-driven microservices | No auth on ZK/Kafka (port 9092/2181), ACL bypass, schema registry data leak |
| **RabbitMQ** | Traditional messaging | Management plugin exposure (`:15672`), guest/guest default creds, vhost enumeration |
| **Amazon SQS / SNS** | AWS event-driven | IAM policy over-permission, cross-account queue injection, SNS topic public subscribe |
| **Google Pub/Sub** | GCP | Service account permission escalation |
| **NATS** | Cloud-native, lightweight | No auth by default, account JWT confusion |
| **Apache Pulsar** | Kafka alternative | Auth token leak, function worker RCE |

### Bug Context:
- **SSRF hitting queue internals** — if you can SSRF to `localhost:15672` (RabbitMQ) or `localhost:9092` (Kafka), interesting things happen
- **Message injection** — if you can publish to a queue, what subscribers trust that input?
- **Dead letter queue poisoning** — messages that fail processing go to a DLQ; if you can trigger reprocessing, you might get bypasses on validation that only runs on first attempt

---

## 12. Authentication & Identity

| Technology | Where | Attack Surface |
|-----------|-------|---------------|
| **Okta** | Enterprise SSO #1 | OAuth/OIDC misconfig (redirect_uri bypass, state param missing → CSRF on login), SAML signature exclusion, SCIM provisioning abuse |
| **Azure AD (Entra ID)** | Microsoft ecosystem | OAuth consent phishing (illegal grant), device code flow abuse, guest account privilege escalation, cross-tenant access |
| **Auth0 (Okta subsidiary)** | Tech/SaaS | OIDC misconfig, `audience` validation bypass, custom DB connection code injection, management API key leak in client-side JS |
| **AWS Cognito** | AWS mobile/web apps | Misconfigured identity pools (unauthenticated → authenticated role escalation), user pool ID leak, JWT kid injection |
| **Keycloak (open source)** | Self-hosted identity | Auth bypass (CVE-2020-10770), XSS in admin console, token replay, `redirect_uri` validation bypass, outdated instances (check `/auth/`) |
| **PingIdentity / PingFederate** | Legacy enterprise, banking | SAML signature wrapping, older protocols (WS-Federation) with weaker security |
| **ForgeRock** | Enterprise IAM | Directory traversal (CVE-2021-35464), auth bypass |
| **LDAP / Active Directory** | Internal identity | LDAP injection (still works in legacy apps), anonymous bind enabled, `userPassword` attribute readable |
| **Kerberos** | Internal auth (Windows) | Kerberoasting (weak SPN service account passwords), AS-REP roasting, Silver/Golden ticket abuse |
| **SAML** (protocol) | Enterprise federation everywhere | Signature exclusion (comment out `<ds:Signature>`), signature wrapping, token recipient confusion, XML signature algorithm downgrade |
| **OAuth 2.0 / OIDC** (protocol) | Web/mobile auth | `redirect_uri` validation bypass, CSRF (missing `state`), PKCE bypass on mobile, `response_type` / `response_mode` confusion, implicit flow abuse, scope upgrade |

### Key Auth Bug Classes:
- **OAuth redirect_uri manipulation** — `/callback?code=xyz&redirect_uri=https://evil.com` — app redirects there
- **JWT attacks**: `alg: none`, RS256→HS256, `kid` injection, `jku`/`jwk` header injection
- **SAML signature exclusion** — just remove or comment out the `<ds:Signature>` element
- **Race conditions in 2FA** — complete login before 2FA is enforced
- **Session fixation** — set your session cookie then trick victim into using it

---

## 13. Cloud Providers — Native Stacks

### AWS (Largest Market Share)
| Service | Purpose | Equivalent To |
|---------|---------|---------------|
| **CloudFront** | CDN | Cloudflare/Akamai |
| **ALB / NLB** | L7/L4 load balancer | NGINX/HAProxy/F5 |
| **WAF** | WAF | ModSecurity/Cloudflare WAF |
| **API Gateway** | API management | Kong/Apigee |
| **Shield** | DDoS protection | Prolexic |
| **Cognito** | Identity | Okta/Keycloak |
| **Route 53** | DNS | — |
| **Lambda** | Serverless compute | — |
| **ECS / EKS** | Containers / K8s | — |
| **S3** | Object storage | MinIO/Ceph |
| **RDS / Aurora** | Managed DB (MySQL/PG) | — |
| **DynamoDB** | NoSQL | MongoDB/Cassandra |
| **ElastiCache** | Redis/Memcached | — |
| **SQS / SNS / EventBridge** | Message queues | Kafka/RabbitMQ |
| **Secrets Manager** | Secrets | Vault |

### GCP
- Cloud CDN, Cloud Load Balancing, Cloud Armor (WAF/DDoS), Apigee, Cloud Run, GKE, Cloud Storage, Pub/Sub, Secret Manager, IAP (Identity-Aware Proxy)

### Azure
- Azure Front Door (CDN + WAF), Azure Load Balancer / Application Gateway, Azure AD, Azure API Management, Functions, AKS, Blob Storage, Service Bus, Key Vault

### Bug Context:
- **S3 bucket misconfig** is the classic but still found: `bucket-name.s3.amazonaws.com` → list objects, write, read
- **Cloud metadata SSRF** → `http://169.254.169.254/latest/meta-data/` — steal IAM credentials
- **Lambda environment variables** leak secrets if you get read access
- **IAM privilege escalation** — a low-privilege role can often escalate via specific API calls

---

## 14. Storage & File Handling

| Technology | Attack Surface |
|-----------|---------------|
| **AWS S3** | Public bucket, writable bucket (upload malicious content served from same origin), versioning leak, cross-account access, CloudFront+S3 OAI bypass |
| **Azure Blob Storage** | Public container, SAS token with excessive permissions, anonymous read access |
| **GCP Cloud Storage** | Public bucket, uniform bucket-level access misconfig |
| **MinIO** | Self-hosted, default creds (`minioadmin:minioadmin`), console exposure, bucket policy bypass |
| **Ceph / RADOS** | Self-hosted, S3 API compatible, same bucket misconfig issues |
| **NFS** | Internal, no-auth shares exposing source code / secrets |
| **FTP / SFTP** | Anonymous login, cleartext creds, path traversal in file path |
| **Nextcloud / ownCloud / Seafile** | Self-hosted file sharing, plugin RCE, share link enumeration |

### File Upload Bugs (universal):
- Unrestricted extension → webshell
- SVG upload → XSS, SSRF via `<use>`/`<image>` tags
- Zip slip / path traversal in archive extraction
- ImageMagick/Ghostscript RCE via crafted images (ImageTragick)
- FFmpeg SSRF via HLS playlist in video upload
- Office document XXE → SSRF in docx/xlsx processing
- PDF → SSRF via PDF generators (wkhtmltopdf, PrinceXML)

---

## 15. Containerization & Orchestration

| Technology | Where | Attack Surface |
|-----------|-------|---------------|
| **Docker** | Everywhere | Docker socket exposure (`/var/run/docker.sock`) → host escape, insecure daemon TCP (`:2375`/`:2376`), privileged container escape, `--cap-add=SYS_ADMIN` and friends |
| **Kubernetes** | Cloud-native enterprise | **Service account token** in `/var/run/secrets/kubernetes.io/serviceaccount/token`, `kubectl` proxy exposed, etcd open (`:2379`), RBAC misconfig (list pods → exec → escape), admission controller bypass |
| **Helm** (K8s package manager) | K8s shops | Tiller (Helm v2) no-auth on `:44134`, chart injection |
| **Podman** | RHEL/Fedora shops | Similar socket exposure concerns |
| **containerd** | Underlying runtime | Less likely exposed but CRI socket access = container escape |

### K8s Specific Recon:
- `/.well-known/` on K8s-hosted apps may leak internal service names
- `kubectl get pods -o wide` often leaks internal IPs in error messages
- `/metrics` endpoints (Prometheus) — if unauthenticated, they enumerate every service in the cluster
- K8s API server accessible at `kubernetes.default.svc.cluster.local` from inside a pod
- **Istio sidecar proxy** — try port 15000, 15001, 15004 locally within an SSRF context

---

## 16. CI/CD & Source Control

| Technology | Attack Surface |
|-----------|---------------|
| **GitHub** | `.git` exposure → source code leak, GitHub Actions injection (pull_request_target + checkout), branch protection bypass |
| **GitLab** | `.git` leak, CI/CD pipeline injection via `.gitlab-ci.yml` in MR, registry credential leak, runner escape |
| **Bitbucket** | Pipeline injection, repository variable leak |
| **Jenkins** | Groovy script console RCE, unauthenticated access (`/script`, `/job/.../build`), credential binding leak, pipeline sandbox escape |
| **ArgoCD** | Dashboard unauthenticated, API key leak, Helm chart injection, config drift |
| **Drone CI** | Pipeline secret exposure, runner misconfig |
| **CircleCI / Travis / Buildkite** | Pipeline env variable leak in build logs |
| **Terraform Cloud / HCP** | State file with secrets, plan output leak |
| **Ansible Tower / AWX** | API exposure, credential type injection |
| **Spinnaker** | Pipeline expression injection, cloud account credential leak |

### Bug Context:
- **.git exposed on webroot** — `wget -r target.com/.git/` dumps full source + secrets
- **CI/CD pipeline injection** — if you can modify a file that's read during CI → steal CI secrets
- **Docker image in public/private registry** — `docker pull` and extract layers for secrets
- **Build artifact leaks** — `.env`, `credentials.json`, `.npmrc` in built JS bundles
- **Webhook secrets** visible in repository settings → forge webhook events

---

## 17. Artifact Registries & Package Repositories

| Technology | Attack Surface |
|-----------|---------------|
| **JFrog Artifactory** | Anonymous repo browsing, mis-scoped API keys, SSRF in integrations, build-info leaks with credentials and package URLs |
| **Sonatype Nexus** | Default/admin creds, repository exposure, proxying internal packages, old RCE/deserialization bugs |
| **GitHub Packages / GHCR** | Token over-permission, private package leakage through Actions or config files |
| **AWS ECR** | IAM over-permission, leaked `docker login` tokens in CI logs |
| **Azure Container Registry (ACR)** | Admin user enabled, pull/push perms too broad, task/webhook exposure |
| **GCP Artifact Registry / GCR** | Service account over-permission, image/package metadata leakage |
| **Private npm / PyPI / Maven proxies** | Dependency confusion, internal package name disclosure, token leakage in `.npmrc`, `pip.conf`, `settings.xml` |

### Bug Context:
- **Dependency confusion** still works when companies leak internal package names or use weak namespace controls
- Artifact stores often contain **build metadata**, **docker layers**, and **old package versions** with secrets left inside
- If you can read package manager config, you often get reusable tokens for code, packages, or CI

---

## 18. Secrets Management & Configuration

| Technology | Attack Surface |
|-----------|---------------|
| **HashiCorp Vault** | Unauthenticated health/metrics endpoints, overly broad tokens, AppRole secret ID leakage, SSRF to local Vault agent |
| **AWS Secrets Manager / SSM Parameter Store** | IAM over-permission, Lambda/env var leakage exposing secret names and ARNs |
| **Azure Key Vault** | Mis-scoped managed identities, public network access, secret enumeration via leaked app permissions |
| **GCP Secret Manager** | Service account over-permission, secret version enumeration |
| **Kubernetes Secrets / ConfigMaps** | Base64-encoded secrets readable from pod compromise, mounted files in containers, etcd exposure |
| **Consul** | KV secrets exposure, ACL misconfig, anonymous service discovery |
| **etcd** | Plaintext cluster state including secrets if directly exposed |

### Bug Context:
- **Secrets managers are rarely the entry point**; they are the payoff once you get SSRF, pod access, CI access, or IAM drift
- **Leaked read-only tokens still matter** — they often expose DB creds, API keys, signing secrets, and internal hostnames
- **Config stores** frequently reveal service names, feature flags, callback URLs, and environment separation mistakes

---

## 19. Enterprise Middleware & Business Platforms

| Technology | Where | Attack Surface |
|-----------|-------|---------------|
| **Oracle WebLogic** | Legacy enterprise Java | `/console` exposure, deserialization RCE, T3 protocol abuse, JNDI issues |
| **JBoss / Red Hat EAP / WildFly** | Enterprise Java apps | JMX console exposure, deserialization, management interface leaks |
| **IBM WebSphere** | Large legacy orgs | Admin console exposure, SOAP admin endpoints, old deserialization bugs |
| **SAP NetWeaver / SAP Portal** | Manufacturing, finance, gov | Default paths, ICM misconfig, file disclosure, auth bypasses, RFC exposure |
| **Adobe Experience Manager (AEM)** | Marketing-heavy enterprises | Dispatcher bypass, CRX package manager exposure, SSRF, content repo leaks |
| **ServiceNow** | ITSM and internal workflows | Misconfigured guest access, API token leakage, ACL bypass on custom apps |
| **Oracle E-Business Suite** | Large legacy enterprise | Old auth bypass chains, exposed admin endpoints, weak SSO integrations |

### Recon Notes:
- Look for highly recognizable paths: `/console`, `/ibm/console`, `/webdynpro/`, `/irj/portal`, `/crx/`, `/system/console`, `/servicenow/`
- These platforms often sit behind SSO, but the **management endpoint** or **legacy virtual host** is what gets left behind
- Legacy enterprise middleware is disproportionately valuable because it bridges auth, internal systems, and sensitive workflows

---

## 20. Email, Collaboration & Workspace Platforms

| Technology | Where | Attack Surface |
|-----------|-------|---------------|
| **Microsoft Exchange / OWA / ECP** | Large enterprise standard | OWA/ECP exposure, legacy auth, SSRF/auth bypass/RCE classes, Autodiscover leaks |
| **Microsoft SharePoint** | Internal portals, document workflows | Deserialization, auth bypass, file upload abuse, search/index leakage, exposed admin paths |
| **Atlassian Confluence** | Engineering and IT teams | Public spaces, attachment/file disclosure, old OGNL/template injection classes, admin panel exposure |
| **Atlassian Jira** | Engineering, support, ITSM | Anonymous project browsing, issue data leakage, API token misuse, SSRF in integrations |
| **Teams / Slack** | Enterprise messaging | Webhook/token leakage, file link exposure, bot mis-scoping, tenant metadata disclosure |

### Recon Notes:
- These systems leak **usernames, project names, ticket IDs, email patterns, internal hostnames, and attached files**
- Even when the app itself is in scope indirectly, public collaboration artifacts can dramatically improve recon and chaining
- Search for exposed help centers, public spaces, attachments, export endpoints, and guest access flows

---

## 21. Remote Access, VPN & Zero Trust Edge

| Technology | Where | Attack Surface |
|-----------|-------|---------------|
| **Palo Alto GlobalProtect** | Enterprise remote access | Portal/gateway exposure, auth bypass chains, SAML misconfig, clientless portal leaks |
| **Fortinet FortiGate / FortiProxy** | Mid-market and enterprise | SSL VPN auth bypass/RCE class bugs, admin panel exposure, weak split-tunnel policy |
| **Cisco ASA / AnyConnect** | Legacy enterprise standard | WebVPN exposure, path traversal/file read class bugs, outdated appliances |
| **Ivanti Connect Secure / Pulse Secure** | Gov, healthcare, enterprise | Repeated auth bypass/RCE history, admin endpoints, session theft |
| **Citrix Gateway / ADC** | Enterprise VDI and remote access | Same NetScaler class bugs, session token theft, auth bypass |
| **Zscaler / Cloudflare Access / Netskope / Prisma Access** | Zero-trust edge | OIDC/SAML trust misconfig, policy bypass through alternate hostnames, exposed apps behind weak posture checks |

### Why it matters:
- These products are **internet-facing by design** and often terminate auth before traffic reaches the app
- They frequently expose version clues, HTML assets, and JavaScript configs that reveal tenants, IdPs, and internal app names
- If the target uses ZTNA/remote access, the exposed portal may be a bigger attack surface than the main website

---

## 22. PKI, Certificate Services & Federation Infrastructure

| Technology | Where | Attack Surface |
|-----------|-------|---------------|
| **Active Directory Certificate Services (AD CS)** | Microsoft-heavy enterprise | Misconfigured enrollment templates, ESC-style privilege escalation paths, certificate-based auth abuse |
| **ADFS** | Legacy Microsoft federation | Token-signing trust issues, auth bypass/misconfig, exposed metadata and login endpoints |
| **Microsoft Entra App Proxy** | Publishing internal apps externally | Mis-scoped app exposure, header trust issues, backend auth assumptions |
| **Venafi / enterprise PKI managers** | Large enterprise certificate lifecycle | Admin/API exposure, certificate inventory leakage, private key/workflow misconfig |
| **SCEP / NDES / certificate enrollment portals** | Device and VPN cert issuance | Enrollment abuse, weak challenge protection, device trust bypass |

### Why it matters:
- Certificate infrastructure defines **who the enterprise trusts**, not just who can log in
- These systems often bridge AD, VPN, Wi-Fi, MDM, and internal apps, so one misconfig can unlock several layers at once
- Public metadata, enrollment endpoints, and template misconfigurations are high-value recon targets

---

## 23. VDI, Virtualization & Hypervisors

| Technology | Where | Attack Surface |
|-----------|-------|---------------|
| **Citrix Virtual Apps / StoreFront** | Enterprise remote desktops/apps | StoreFront exposure, gateway trust issues, session token leakage, legacy auth paths |
| **VMware vCenter / ESXi** | Datacenter standard | vCenter auth bypass/RCE classes, exposed management UI/API, datastore browsing, plugin bugs |
| **Horizon / Omnissa Horizon** | Enterprise VDI | HTML Access exposure, broker/gateway misconfig, old auth bypass chains |
| **Proxmox VE** | SMB to mid-market | Web UI exposure, API token leakage, backup storage mounts |
| **Hyper-V / SCVMM** | Microsoft-heavy shops | Management portal exposure, delegated admin abuse, integration with AD and backups |

### Why it matters:
- These platforms often expose **high-privilege management planes** rather than just one application
- Even read-only access can reveal VM names, internal networks, templates, snapshots, and credentials in mounted ISOs/scripts
- VDI portals also leak usernames, tenant names, IdPs, and app catalogs useful for recon

---

## 24. Endpoint Management, MDM & EDR

| Technology | Where | Attack Surface |
|-----------|-------|---------------|
| **Microsoft Intune / Endpoint Manager** | Microsoft enterprise fleet management | Enrollment profile leakage, device compliance policy abuse, app config exposure |
| **VMware Workspace ONE / AirWatch** | Enterprise MDM | Enrollment portal exposure, staging credentials, API mis-scoping |
| **Jamf Pro** | Apple fleet management | Enrollment invitations, package/script distribution abuse, token leakage |
| **SCCM / MECM** | Legacy Windows fleet management | Distribution point exposure, package/script leakage, NTLM relay opportunities |
| **Tanium** | Large enterprise endpoint ops | Console/API exposure, client package distribution, sensitive inventory data |
| **CrowdStrike / SentinelOne / Defender for Endpoint** | EDR platforms | API token leakage, webhook/integration secrets, host inventory and detection metadata exposure |

### Recon Notes:
- MDM/EDR systems expose **device names, usernames, software inventory, internal domains, VPN configs, and deployment scripts**
- They matter less as a direct internet RCE target and more as a **source of credentials, internal topology, and trusted scripts**
- Search CI, mobile configs, public docs, and support portals for enrollment URLs, bootstrap tokens, and agent configs

---

## 25. Network Management & Security Appliances

| Technology | Where | Attack Surface |
|-----------|-------|---------------|
| **Palo Alto PAN-OS** | Enterprise firewall standard | Admin UI/API exposure, auth bypass/RCE class bugs, config export leakage |
| **Cisco ISE** | NAC / enterprise access control | Guest portal exposure, admin API, RADIUS/TACACS integrations, certificate handling issues |
| **Cisco FMC / Firepower** | Enterprise firewall management | Central management UI exposure, policy export, vulnerable plugins/services |
| **FortiManager / FortiAnalyzer** | Fortinet shops | Admin exposure, backup/config leakage, token/session handling bugs |
| **Aruba ClearPass** | NAC / guest access | Guest portal misconfig, admin panel exposure, certificate and onboarding profile leakage |
| **SonicWall SMA / NSM** | SMB to enterprise | Appliance web UI exposure, historical auth bypass/RCE classes |
| **Forescout / NAC platforms** | Large enterprise network control | Appliance exposure, credentialed scan data, internal asset inventory |

### Bug Context:
- These tools often know **every device, VLAN, certificate, and policy** in the environment
- Guest portals and onboarding flows are particularly valuable because they are internet-facing or semi-public by design
- Config backups from these systems can contain shared secrets, SNMP strings, VPN settings, and internal addressing

---

## 26. Data Warehouses, BI & Analytics Platforms

| Technology | Where | Attack Surface |
|-----------|-------|---------------|
| **Snowflake** | Modern enterprise analytics | Mis-scoped roles, shared data exposure, external stage credential leakage |
| **Databricks** | Data engineering / ML | Workspace token leakage, notebook secrets, cluster init scripts, DBFS data exposure |
| **BigQuery** | GCP analytics | Service account over-permission, dataset exposure, scheduled query credential drift |
| **Redshift** | AWS analytics warehouse | IAM/auth drift, snapshot exposure, SQL-based data exfiltration |
| **Power BI** | Microsoft-heavy enterprise | Public report links, embedded tokens, dataset/API over-sharing |
| **Tableau** | BI dashboards | Public workbooks, extract downloads, service account credentials in connectors |
| **Looker** | SaaS analytics | Embed secret misuse, model/repo leakage, SQL runner over-permission |

### Recon Notes:
- Analytics platforms leak **customer data, internal business metrics, partner names, user identifiers, and service credentials**
- Public dashboards and embedded analytics often reveal hidden APIs, object IDs, and backend query structure
- Notebook and connector configs are especially valuable because they bridge cloud storage, databases, and SaaS APIs

---

## 27. Backup, Disaster Recovery & Managed File Transfer

| Technology | Where | Attack Surface |
|-----------|-------|---------------|
| **Veeam** | Enterprise backup standard | Console/API exposure, backup metadata, credential storage, mounted backup browsing |
| **Commvault** | Large enterprise backup | Web console exposure, agent auth issues, backup job metadata |
| **Rubrik** | Modern enterprise backup | API token leakage, snapshot exposure, cloud archive trust relationships |
| **Cohesity** | Backup / secondary storage | Admin API exposure, backup search data, restore workflow abuse |
| **MOVEit Transfer** | Managed file transfer | Internet-facing file transfer, auth bypass/RCE classes, sensitive file stores |
| **GoAnywhere MFT** | Managed file transfer | Admin console exposure, workflow abuse, old RCE class bugs |
| **Accellion / Kiteworks / similar MFT** | B2B file exchange | Legacy appliance exposure, file disclosure, external sharing misconfig |

### Why it matters:
- Backup systems are a **treasure map**: they know where the sensitive data lives and often store copies of it
- MFT platforms are commonly internet-facing and frequently hold payroll, HR, finance, and partner data
- Restore/export workflows can become a path to large-scale data access even without code execution

---

## 28. Monitoring & Observability

| Technology | Attack Surface |
|-----------|---------------|
| **Prometheus** | `/metrics` or `:9090` unauthenticated → full service/endpoint enumeration, scrape targets, alert rules, internal hostnames |
| **Grafana** | Dashboard public share link, CVE-2021-43798 path traversal (file read), default admin/admin, alert webhook SSRF |
| **Datadog** | Agent API key leak in source/env, log forwarding config |
| **ELK Stack (Elasticsearch + Logstash + Kibana)** | ES unauthenticated (see DB section), Kibana RCE (CVE-2019-7609 prototype pollution), Logstash pipeline injection |
| **Splunk** | REST API exposure, search head auth bypass, HEC token leak |
| **New Relic** | API key in source, browser agent data leak (client-side secrets) |
| **Sentry** | DSN exposure in client JS → inject fake errors, event data leak, internal host leak |
| **Nagios / Icinga** | Unauthenticated status pages revealing infrastructure, CGI script RCE |
| **Zabbix** | Default creds (Admin:zabbix), `latest.php` SQL injection |
| **Jaeger / Zipkin** (tracing) | Unauthenticated → trace data exposes internal API calls and parameters |

---

## 29. Putting It All Together — Recon Strategy

### Phase 1: Passive Fingerprinting
1. **Browsing normally**: `Server`, `X-Powered-By`, `Set-Cookie` headers, `Via` header
2. **TLS cert** (crt.sh) — SANs enumerate subdomains, issuer tells you if it's Cloudflare/AWS
3. **SecurityTrails / Shodan / Censys** — historical DNS, open ports, tech fingerprint
4. **BuiltWith / Wappalyzer** — guess framework from JS/CSS signatures
5. **Error pages** — `/doesnotexist`, `/%00`, `/..;/` — the error format tells you the server stack

### Phase 2: Active Mapping
1. **Subdomain enumeration** — amass, subfinder, cert spotter
2. **Port scan** non-standard ports on discovered hosts
3. **Directory brute-force** — look for `/actuator`, `/swagger.json`, `/api-docs`, `/.git`, `/metrics`, `/env`
4. **Technology-specific probes** — SAP NetWeaver paths, Oracle WebLogic console, JBoss JMX

### Phase 3: Attack Surface Prioritization
1. **Edge layer** — Can you bypass CDN/WAF? Is origin IP leaking?
2. **Auth layer** — OAuth/SAML/JWT misconfigurations
3. **API layer** — IDOR, mass assignment, insecure direct object references
4. **Application layer** — Framework-specific vulnerabilities (Spring actuators, Express prototype pollution, Laravel debug mode)
5. **Infrastructure layer** — Exposed services (Redis, Elasticsearch, Docker API, K8s dashboard)
6. **CI/CD layer** — `.git` exposed, pipeline injection, leaked secrets

---

### Quick Reference: Headers That Tell You What You're Hitting

| Header Pattern | Technology |
|----------------|-----------|
| `cf-ray`, `cf-cache-status`, `__cf_bm` | Cloudflare |
| `x-akamai-request-id`, `X-Akamai-Edge-*` | Akamai |
| `x-served-by`, `x-cache-hits`, `x-timer` | Fastly |
| `x-amz-cf-id`, `x-amz-cf-pop` | AWS CloudFront |
| `x-azure-ref`, `x-ms-*` | Azure Front Door |
| `Server: BigIP`, Cookie: `BIGipServer~...` | F5 BIG-IP |
| `Server: nginx` | NGINX |
| `Server: Apache` or `Server: Apache/2.x` | Apache httpd |
| `Server: Microsoft-IIS` | IIS |
| `X-Powered-By: Express` | Node.js Express |
| `X-Powered-By: PHP/x.x` | PHP |
| `X-Powered-By: ASP.NET` | .NET |
| `X-AspNet-Version`, `X-AspNetMvc-Version` | ASP.NET MVC |
| `x-envoy-*`, `x-request-id` | Envoy proxy |
| `X-Kong-*`, `via: kong` | Kong API Gateway |
| `x-waf-*`, `x-cdn-*` | Generic CDN/WAF |
| `server: Kestrel` | ASP.NET Core |
| `X-Runtime`, `X-Request-Id: ...` (UUID, no proxy indicator) | Ruby on Rails |
| `Set-Cookie: laravel_session` | Laravel |
| `Set-Cookie: JSESSIONID` | Java/Jakarta |
| `Set-Cookie: PHPSESSID` | PHP |
| `Set-Cookie: ASPSESSIONID` | Classic ASP |
| `Set-Cookie: .AspNetCore.*` | ASP.NET Core |

---

## 30. Recon Checklist / Cheat Sheet

### 60-Second First Pass
- Check response headers: `Server`, `Set-Cookie`, `Via`, `X-Powered-By`, CDN/WAF headers
- Check TLS cert SANs on `crt.sh` for subdomains and wildcard patterns
- Check if the host sits behind Cloudflare/Akamai/Fastly/CloudFront and ask: can the origin be found?
- Check whether the app smells like enterprise: SSO, VPN, `/api/`, `/admin/`, `/actuator`, `/swagger`, `/docs`, `/metrics`
- Check for obvious platform clues in HTML, JS, favicon hashes, and error pages

### Core Recon Workflow
1. Enumerate subdomains from CT logs, passive sources, and brute-force
2. Fingerprint CDN/WAF/reverse proxy/load balancer
3. Hunt for origin IP leakage and direct-access hosts
4. Scan common alternate ports and admin panels
5. Brute-force high-value paths for framework, middleware, and observability leaks
6. Prioritize auth, API, admin, file-handling, and exposed-infra surfaces
7. Chain what you find: SSRF → metadata/localhost, read access → secrets, CI leak → package/token abuse

### High-Value Paths To Probe
- `/.git/`, `/.env`, `/server-status`, `/nginx-status`, `/metrics`, `/health`, `/debug`
- `/swagger.json`, `/openapi.json`, `/api-docs`, `/graphql`, `/graphiql`
- `/actuator`, `/actuator/env`, `/actuator/heapdump`, `/actuator/mappings`
- `/admin/`, `/login`, `/console`, `/manager`, `/host-manager`, `/jmx-console`
- `/owa/`, `/ecp/`, `/autodiscover/`, `/vpn/`, `/remote/`, `/citrix/`
- `/crx/`, `/system/console`, `/servicenow/`, `/ibm/console`, `/irj/portal`

### If You See This, Think This
- CDN/WAF headers: cache poisoning, origin bypass, parsing differentials
- SSO/OAuth/SAML: redirect handling, token validation, `state`, audience, relay/ACS confusion
- `swagger`/`openapi`: undocumented endpoints, hidden params, old versions
- `metrics`/`trace`/`logs`: internal hostnames, tokens, queue names, service map
- Java middleware: deserialization, admin consoles, JMX, old endpoints
- K8s/cloud clues: metadata SSRF, service account tokens, IAM drift, dashboard/API exposure
- Package registries/artifacts: dependency confusion, leaked tokens, docker layers, build secrets
- Backup/MFT/collab suites: mass data exposure, guest access, exported files, workflow abuse

### Best Enterprise Pivot Points
- **SSRF** into `localhost`, metadata endpoints, admin panels, Prometheus, queues, Vault agents
- **Read-only access** to config, metrics, docs, artifacts, or dashboards
- **Exposed admin/UI** with weak auth, SSO gaps, or legacy endpoints
- **Token leakage** in JS, CI logs, mobile configs, package manager files, exported reports
- **Guest/public portals** for Jira, Confluence, SharePoint, ServiceNow, NAC, VPN, MFT

### Prioritization Rules
- Prioritize assets that terminate trust: CDN, WAF, SSO, VPN, API gateway, reverse proxy
- Prioritize assets that reveal internal topology: metrics, tracing, dashboards, certs, backup consoles
- Prioritize assets that store reusable secrets: CI/CD, artifact registries, Vault, cloud IAM, MDM
- Prioritize legacy enterprise middleware because it is often forgotten and highly privileged
- Prioritize internet-facing portals that are not the main app because they are often weaker

### Stop And Ask
- Can I hit the origin directly?
- Can I reach a different parser than the WAF sees?
- Can this low-impact leak become credentials, tokens, or internal hostnames?
- Can this admin or support platform pivot me into the real environment?
- Can I turn visibility into control by chaining auth drift, SSRF, or secret leakage?
