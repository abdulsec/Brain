# Elite Web Bug Bounty Roadmap For The AI Era

Last updated: 2026-05-17

Profile assumed:

- Full-time bug bounty hunter.
- 5 years of experience.
- Strongest areas: IDOR, business logic errors, privilege escalation.
- Goal: become elite in web testing, stay competitive in the AI era, and expand into high-value bug classes beyond access control.

Use this only on authorized targets: bug bounty scopes, private labs, local environments, CTFs, or your own infrastructure.

## Strategic Positioning

Your advantage is not starting from zero. IDOR, business logic, and privilege escalation are already the bug classes most scanners fail to find. The goal now is to turn that specialty into a broader research system:

- Stay elite at authorization bugs.
- Add deep API testing.
- Add modern frontend and mobile API analysis.
- Add AI/LLM application testing.
- Add source-code-assisted testing.
- Add automation for coverage, not noisy scanning.
- Add high-quality reporting that makes triage easy.

The best hunters in the AI era will not be the people who just ask AI for payloads. They will be the people who understand application behavior deeply and use AI to move faster through code, documentation, diffs, logs, and test generation.

## What To Master Next

Priority order:

1. Advanced access control and authorization design flaws.
2. API security and object/property/function-level authorization.
3. Business logic in payments, subscriptions, credits, coupons, workflows, and approvals.
4. Race conditions and state-machine bugs.
5. OAuth, SSO, SAML, OIDC, JWT, and session management.
6. GraphQL and BFF APIs.
7. Webhooks, integrations, and third-party trust boundaries.
8. Cloud and SaaS misconfigurations exposed through web apps.
9. AI/LLM application security.
10. Source code review and patch diffing.
11. Request smuggling, cache poisoning, and HTTP parser confusion.
12. Client-side security: DOM XSS, postMessage, prototype pollution, CORS.
13. Deserialization, file processing, SSRF, and internal service pivots.
14. Multi-tenant SaaS isolation testing.

## Core Official References

- PortSwigger Web Security Academy: https://portswigger.net/web-security
- PortSwigger all labs: https://portswigger.net/web-security/all-labs
- PortSwigger Web LLM attacks: https://portswigger.net/web-security/learning-paths/llm-attacks
- OWASP API Security Top 10: https://owasp.org/API-Security/
- OWASP Top 10 for LLM Applications: https://owasp.org/www-project-top-10-for-large-language-model-applications/
- OWASP Web Security Testing Guide: https://owasp.org/www-project-web-security-testing-guide/
- OWASP ASVS: https://owasp.org/www-project-application-security-verification-standard/
- OWASP Cheat Sheet Series: https://cheatsheetseries.owasp.org/
- PortSwigger Research: https://portswigger.net/research

## Elite Hunter Mindset

Think in systems, not payloads.

For every target, ask:

- Who are the users?
- What roles exist?
- What objects belong to whom?
- What state transitions exist?
- What actions cost money?
- What actions create trust?
- What actions affect other users?
- What data should never cross tenant boundaries?
- What internal service does the public app depend on?
- Where does AI, automation, or background processing act on behalf of users?

Your job is to break assumptions.

## Your New Skill Tree

### 1. Authorization Mastery

You already specialize in IDOR, so upgrade from basic object ID changes to full authorization modeling.

Master:

- BOLA / IDOR.
- BFLA: broken function-level authorization.
- BOPLA: broken object property-level authorization.
- Cross-tenant access.
- Organization switching bugs.
- Workspace/project/team boundary bugs.
- Invite and role downgrade bugs.
- Ownership transfer bugs.
- Shared-resource bugs.
- Admin API exposure.
- Internal role confusion.
- Deleted/suspended user access.
- API version authorization drift.
- Mobile API authorization drift.
- GraphQL resolver-level authorization gaps.

Labs:

- PortSwigger access control labs: https://portswigger.net/web-security/access-control
- PortSwigger API testing labs: https://portswigger.net/web-security/api-testing
- crAPI: https://github.com/OWASP/crAPI
- VAmPI: https://github.com/erev0s/VAmPI
- Juice Shop: https://owasp.org/www-project-juice-shop/

Daily drills:

- Create at least 3 accounts.
- Create at least 2 organizations/workspaces.
- Test every object across users and tenants.
- Compare web, mobile, GraphQL, and old API behavior.
- Test read, create, update, delete, export, invite, transfer, share, approve, and billing actions.

### 2. Business Logic And State Machines

Business logic becomes elite when you model application state.

High-value areas:

- Checkout and payment flows.
- Refunds and chargebacks.
- Coupons and promo credits.
- Subscription upgrades and downgrades.
- Usage limits and quotas.
- Free trial abuse.
- Referral systems.
- Marketplace payouts.
- Order cancellation.
- Approval workflows.
- KYC or verification flows.
- Invite and onboarding flows.
- Account recovery.
- Email or phone verification.
- Feature gating.
- Entitlement checks.

Test patterns:

- Skip a step.
- Repeat a step.
- Reverse a step.
- Race two steps.
- Use an old token after state changes.
- Change price, quantity, currency, or plan IDs.
- Replay a webhook.
- Cancel after fulfillment.
- Downgrade after using premium feature.
- Transfer ownership then reuse old privileges.

Labs:

- PortSwigger business logic labs: https://portswigger.net/web-security/logic-flaws
- PortSwigger race condition labs: https://portswigger.net/web-security/race-conditions
- OWASP WSTG business logic testing: https://owasp.org/www-project-web-security-testing-guide/

### 3. API Security

API security is your main expansion lane because IDOR and business logic translate directly.

Master OWASP API risks:

- Broken object-level authorization.
- Broken authentication.
- Broken object property-level authorization.
- Unrestricted resource consumption.
- Broken function-level authorization.
- Unrestricted access to sensitive business flows.
- SSRF.
- Security misconfiguration.
- Improper inventory management.
- Unsafe consumption of APIs.

Testing checklist:

- Inventory all API hosts.
- Find old API versions.
- Compare web vs mobile requests.
- Compare REST vs GraphQL.
- Compare admin vs user endpoints.
- Compare public docs vs hidden endpoints.
- Check object IDs, slugs, UUIDs, and composite IDs.
- Test PATCH/PUT hidden properties.
- Test mass assignment.
- Test filtering and sorting fields.
- Test exports and reports.
- Test async jobs and notifications.

Labs:

- PortSwigger API testing: https://portswigger.net/web-security/api-testing
- crAPI: https://github.com/OWASP/crAPI
- VAmPI: https://github.com/erev0s/VAmPI
- Damn Vulnerable GraphQL Application: https://github.com/dolevf/Damn-Vulnerable-GraphQL-Application

Tools:

- Burp Suite.
- Caido.
- mitmproxy.
- Postman or Insomnia.
- jq.
- httpx.
- ffuf.
- kiterunner.
- arjun.
- nuclei for low-noise custom checks.

### 4. OAuth, SSO, SAML, OIDC, JWT

Many strong hunters are weak here. This is a competitive edge.

Master:

- OAuth authorization code flow.
- PKCE.
- OIDC ID tokens.
- SAML assertions.
- JWT claims.
- Account linking.
- Login CSRF.
- Redirect URI validation.
- State parameter validation.
- Token audience confusion.
- Token substitution.
- Organization membership after SSO.
- SSO domain takeover scenarios.
- SCIM provisioning bugs.

Labs:

- PortSwigger OAuth labs: https://portswigger.net/web-security/oauth
- PortSwigger JWT labs: https://portswigger.net/web-security/jwt
- SAML Raider: https://portswigger.net/bappstore/efabb620b7494c5995e91de2f55c2f5c
- Auth0 docs for conceptual learning: https://auth0.com/docs
- OpenID Connect specs: https://openid.net/developers/how-connect-works/

High-value tests:

- Can I link my account to victim identity?
- Can I change email after SSO verification?
- Can I reuse an invite after identity changes?
- Can I join an organization by controlling email domain?
- Can I confuse tenant, audience, issuer, or client ID?
- Can I bypass MFA through an alternate login flow?

### 5. GraphQL

GraphQL is an IDOR hunter's playground when authorization is implemented inconsistently.

Master:

- Introspection.
- Query batching.
- Aliases.
- Fragments.
- Nested object access.
- Resolver-level authorization.
- Mutation authorization.
- Cost/depth limits.
- Hidden fields.
- Node/global IDs.
- GraphQL subscriptions.

Labs:

- PortSwigger GraphQL labs: https://portswigger.net/web-security/graphql
- Damn Vulnerable GraphQL Application: https://github.com/dolevf/Damn-Vulnerable-GraphQL-Application

Testing checklist:

- Enumerate schema if allowed.
- Compare fields returned for different roles.
- Query nested objects owned by another user.
- Test mutations with another user's IDs.
- Test batch requests for rate-limit bypass.
- Test aliases to repeat expensive actions.
- Test hidden admin fields.

### 6. Race Conditions

Race conditions are a natural next step from business logic.

Master:

- Single-endpoint races.
- Multi-endpoint races.
- Limit overrun.
- Coupon double spend.
- Balance double spend.
- Invite acceptance races.
- Role update races.
- File replacement races.
- Session invalidation races.
- Webhook replay races.

Labs:

- PortSwigger race condition labs: https://portswigger.net/web-security/race-conditions

Tools:

- Burp Turbo Intruder.
- Caido replay workflows.
- Custom Python async scripts.
- curl parallel runs.

Test targets:

- Payments.
- Wallets.
- Credits.
- Coupons.
- Trials.
- Orders.
- Limit counters.
- Approvals.
- Password reset.
- Email verification.

### 7. HTTP Parser Bugs, Cache, And Smuggling

This area has lower volume but high impact.

Master:

- HTTP request smuggling.
- HTTP/2 downgrade issues.
- Cache poisoning.
- Web cache deception.
- Host header attacks.
- CORS trust mistakes.
- Reverse proxy behavior.
- CDN behavior.

Labs:

- Request smuggling: https://portswigger.net/web-security/request-smuggling
- Web cache poisoning: https://portswigger.net/web-security/web-cache-poisoning
- Web cache deception: https://portswigger.net/web-security/web-cache-deception
- Host header attacks: https://portswigger.net/web-security/host-header
- CORS: https://portswigger.net/web-security/cors

Tools:

- Burp HTTP Request Smuggler.
- Burp Param Miner.
- h2csmuggler.
- custom raw HTTP clients.

### 8. Client-Side Bugs

Do not ignore frontend. Modern SaaS apps put logic in the browser.

Master:

- DOM XSS.
- postMessage bugs.
- OAuth token leakage in frontend routes.
- CORS misconfiguration.
- CSP bypass concepts.
- Prototype pollution.
- Client-side path traversal.
- Source map review.
- Frontend feature flag leakage.
- Exposed GraphQL/API endpoints.

Labs:

- DOM XSS: https://portswigger.net/web-security/cross-site-scripting/dom-based
- Prototype pollution: https://portswigger.net/web-security/prototype-pollution
- CORS: https://portswigger.net/web-security/cors
- Content Security Policy: https://portswigger.net/web-security/cross-site-scripting/content-security-policy

Tools:

- DevTools.
- Burp DOM Invader.
- LinkFinder.
- SecretFinder.
- sourcemapper.
- js-beautify.
- ripgrep.

### 9. SSRF, File Processing, And Internal Integrations

SSRF and file processing bugs often become high-impact when linked to cloud metadata, internal APIs, or AI tools.

Master:

- Classic SSRF.
- Blind SSRF.
- Webhook SSRF.
- PDF/HTML renderer SSRF.
- Image fetch SSRF.
- URL preview SSRF.
- Cloud metadata access.
- Internal admin panels.
- File upload and conversion.
- ZIP/TAR extraction.
- XXE.
- CSV injection.
- PDF generation bugs.

Labs:

- SSRF: https://portswigger.net/web-security/ssrf
- File upload: https://portswigger.net/web-security/file-upload
- XXE: https://portswigger.net/web-security/xxe
- Path traversal: https://portswigger.net/web-security/file-path-traversal

### 10. AI And LLM Application Security

This is where web hunters need to adapt.

Important: basic jailbreaks are usually low value. High-value AI bugs cross real security boundaries.

Master:

- Prompt injection.
- Indirect prompt injection.
- Insecure output handling.
- Excessive agency.
- Sensitive information disclosure.
- Tool/plugin abuse.
- Retrieval augmented generation data exposure.
- Vector database authorization bugs.
- Tenant isolation in AI search.
- AI-generated action abuse.
- Agent permission bypass.
- AI workflow SSRF.
- AI connector OAuth mistakes.
- Model/file upload parsing.

Official references:

- OWASP Top 10 for LLM Applications: https://owasp.org/www-project-top-10-for-large-language-model-applications/
- PortSwigger Web LLM attacks: https://portswigger.net/web-security/learning-paths/llm-attacks
- OWASP GenAI project: https://genai.owasp.org/

Labs and practice:

- PortSwigger Web LLM attack labs.
- Build a local toy RAG app.
- Build a local agent with file, browser, and shell tools.
- Test tool permission boundaries.
- Test tenant isolation in vector search.
- Test whether AI summaries leak documents from another user.
- Test whether AI actions can send emails, create tickets, change settings, or access private integrations.

Targets to study locally:

- Chroma: https://github.com/chroma-core/chroma
- LiteLLM: https://github.com/BerriAI/litellm
- Ollama: https://github.com/ollama/ollama
- LangChain: https://github.com/langchain-ai/langchain
- LlamaIndex: https://github.com/run-llama/llama_index

High-value AI-era test questions:

- Can one user's AI search retrieve another user's documents?
- Can prompt injection cause the app to call a privileged tool?
- Can an uploaded document influence actions outside the chat?
- Can AI output become HTML, SQL, shell, template, Markdown, or code without validation?
- Can a connector token be abused across tenants?
- Can the model be tricked into revealing hidden system data that has real confidentiality impact?
- Can an AI workflow be used as SSRF, CSRF, or stored XSS delivery?
- Can the app trust AI classification for authorization, fraud, or billing decisions?

## Source-Code-Assisted Hunting

This is how you compete with other full-time hunters.

When source is available, do not just run the app. Read it.

Search for:

- Auth middleware.
- Role checks.
- Object ownership checks.
- `isAdmin`, `isOwner`, `tenantId`, `orgId`, `workspaceId`.
- `TODO`, `FIXME`, `temporary`, `legacy`.
- `eval`, `exec`, `Function`, `child_process`, `subprocess`.
- `pickle`, `yaml.load`, deserialization.
- File paths and archive extraction.
- Webhook signature validation.
- Payment state transitions.
- Feature flags.
- Admin routes.
- Background jobs.
- GraphQL resolvers.
- AI tool calls and agent permissions.

AI prompts:

```text
Map this repository's authorization model. List roles, tenants, object ownership fields, middleware, route guards, and places where authorization is missing or inconsistent.
```

```text
Find business logic state machines in this codebase: billing, subscription, invites, approvals, credits, refunds, account recovery, and workspace membership.
```

```text
Review these API handlers for BOLA, BFLA, BOPLA, mass assignment, and tenant isolation bugs. Give me exact files, functions, and test ideas.
```

```text
Find all places where AI/model output reaches tools, HTML, Markdown, SQL, shell, filesystem, network requests, or privileged actions.
```

```text
Generate regression tests for authorization boundaries across two users, two organizations, admin, member, and read-only roles.
```

## Bug Class Expansion Checklist

Focus on these until you are comfortable reporting all of them:

- [ ] IDOR/BOLA.
- [ ] BFLA.
- [ ] BOPLA/mass assignment.
- [ ] Business logic abuse.
- [ ] Race conditions.
- [ ] OAuth/OIDC bugs.
- [ ] SAML bugs.
- [ ] JWT bugs.
- [ ] GraphQL authorization bugs.
- [ ] SSRF.
- [ ] Webhook abuse.
- [ ] Request smuggling.
- [ ] Cache poisoning/deception.
- [ ] CORS misconfig.
- [ ] DOM XSS.
- [ ] postMessage bugs.
- [ ] Prototype pollution.
- [ ] File upload and processing bugs.
- [ ] Path traversal.
- [ ] Deserialization.
- [ ] XXE.
- [ ] Cloud metadata exposure.
- [ ] Multi-tenant SaaS isolation.
- [ ] AI prompt injection with real impact.
- [ ] AI tool/agent permission bypass.
- [ ] RAG/vector database data leakage.
- [ ] AI connector OAuth/token bugs.

## Weekly Training Plan

### Monday: Target Mapping

- Pick one program or app.
- Build user/role/tenant matrix.
- Inventory web, mobile, GraphQL, admin, and old APIs.
- Identify all high-value workflows.

### Tuesday: Authorization Day

- Test BOLA, BFLA, BOPLA.
- Test cross-tenant access.
- Test exports, reports, attachments, comments, notifications.
- Test old API versions.

### Wednesday: Business Logic Day

- Test payments, subscriptions, coupons, invites, approvals, account recovery.
- Try skip, repeat, reverse, replay, race, and stale-token scenarios.

### Thursday: Advanced Web Day

- Rotate between OAuth, GraphQL, SSRF, file upload, CORS, cache, request smuggling, prototype pollution.

### Friday: AI And Integrations Day

- Test AI features, assistants, copilots, RAG search, document upload, automations, connectors, webhooks.
- Focus on real boundaries: tenant, role, token, file, tool, billing, email, and admin action.

### Saturday: Code Review And Automation

- Use Codex/Claude to map code.
- Write small scripts.
- Create Burp/Caido workflows.
- Create custom nuclei templates only for confirmed patterns.
- Build checklists for the target.

### Sunday: Reports And Review

- Finish reports.
- Reproduce findings cleanly.
- Improve impact explanation.
- Review duplicates and rejected reports.
- Update personal methodology.

## 90-Day Upgrade Plan

### Days 1-15: API Dominance

- Complete PortSwigger API labs.
- Complete crAPI.
- Build an API authorization checklist.
- Practice BOLA, BFLA, BOPLA daily.

### Days 16-30: OAuth, JWT, SSO

- Complete PortSwigger OAuth and JWT labs.
- Learn OIDC and SAML basics.
- Test account linking, invite, organization membership, and MFA bypass flows.

### Days 31-45: GraphQL And Race Conditions

- Complete PortSwigger GraphQL labs.
- Complete PortSwigger race condition labs.
- Practice with Turbo Intruder or Caido replay.

### Days 46-60: Advanced Web

- Request smuggling.
- Cache poisoning/deception.
- SSRF.
- File upload.
- Prototype pollution.
- DOM XSS and postMessage.

### Days 61-75: AI/LLM Security

- Complete PortSwigger Web LLM labs.
- Read OWASP LLM Top 10.
- Build a local RAG app.
- Test tenant isolation, tool use, connector tokens, and insecure output handling.

### Days 76-90: Source-Code-Assisted Hunting

- Pick one open-source SaaS-style app.
- Map authorization.
- Write tests.
- Use Semgrep/CodeQL basics.
- Practice patch diffing.
- Produce 2 full research reports from lab findings.

## Competitive Hunting Workflow

For each program:

1. Read policy and scope.
2. Identify business model.
3. Create user matrix.
4. Map all assets and APIs.
5. Map roles and tenants.
6. Map high-value workflows.
7. Test authorization systematically.
8. Test business logic state changes.
9. Test integrations and webhooks.
10. Test AI features.
11. Test mobile and old APIs.
12. Test frontend source maps and JS.
13. Run selective automation.
14. Write clean reports with strong impact.

## User Matrix Template

Create:

- User A: owner of Org A.
- User B: member of Org A.
- User C: read-only member of Org A.
- User D: outsider.
- User E: owner of Org B.
- User F: suspended or removed user.
- User G: invited but not accepted.
- User H: SSO user.
- User I: mobile-only session.

Test every sensitive action across these users.

## Object Matrix Template

Track:

- User ID.
- Organization ID.
- Workspace ID.
- Project ID.
- Team ID.
- Document ID.
- File ID.
- Invoice ID.
- Subscription ID.
- Payment method ID.
- Invite ID.
- Role ID.
- API key ID.
- Integration ID.
- Webhook ID.
- AI conversation ID.
- Vector collection ID.
- Connector token ID.

For each object:

- Can outsider read it?
- Can member update it?
- Can read-only user mutate it?
- Can removed user access it?
- Can Org B user access Org A object?
- Can old API access it?
- Can export/report endpoint leak it?
- Can AI search retrieve it?

## Report Quality Upgrade

Triage teams reward clarity.

Every report should include:

- Short title with impact.
- Affected endpoint or feature.
- Exact role/precondition.
- Clean reproduction steps.
- Two-user or two-tenant proof when relevant.
- Screenshots or HTTP requests.
- Security impact, not just technical behavior.
- Business impact.
- Suggested fix.
- Why it is not intended functionality.

Weak:

- "I can change user_id."

Strong:

- "A read-only member of Organization A can modify billing contact details for Organization B by changing `organization_id` in the invoice settings API. This enables cross-tenant billing data tampering and invoice misdirection."

## AI Assistant Operating Model

Use Codex/Claude as:

- Code reader.
- Diff reviewer.
- Test generator.
- Report editor.
- Endpoint mapper.
- Checklist generator.
- Log analyzer.
- Learning tutor.

Do not use them as:

- Your proof of vulnerability.
- Your only source of truth.
- A replacement for manual reproduction.
- A reason to send low-quality reports.

Best prompts:

```text
I am testing an authorized bug bounty target. Based on these endpoints, build a role/object/tenant authorization test matrix. Prioritize high-impact IDOR, BFLA, BOPLA, and business logic cases.
```

```text
I found this request. Suggest variations to test authorization and business logic safely. Avoid destructive actions and keep the scope bug-bounty appropriate.
```

```text
Analyze this JavaScript bundle for hidden API routes, feature flags, role checks, GraphQL operations, and AI-related endpoints.
```

```text
Rewrite this bug bounty report to be concise, high-impact, and easy for triage to reproduce. Do not exaggerate impact.
```

```text
From this app flow, identify state transitions and race-condition opportunities. Focus on payments, credits, invites, approvals, and role changes.
```

## Automation Stack

Core:

- Burp Suite or Caido.
- browser DevTools.
- ffuf.
- httpx.
- nuclei with custom templates.
- jq.
- Python.
- mitmproxy.
- Postman/Insomnia.

API:

- kiterunner.
- arjun.
- graphql-voyager.
- GraphQLmap for labs.
- jwt_tool.

JS/frontend:

- LinkFinder.
- SecretFinder.
- sourcemapper.
- js-beautify.
- ripgrep.

Race:

- Turbo Intruder.
- Caido replay.
- Python asyncio/httpx scripts.

Code review:

- Semgrep.
- CodeQL basics.
- ripgrep.
- git diff.

AI:

- Claude Code.
- OpenAI Codex.
- Local notes and checklists.
- Use AI to generate tests and summarize code, not to make final security decisions.

## Personal Metrics

Track weekly:

- Number of programs deeply mapped.
- Number of workflows modeled.
- Number of high-quality reports sent.
- Accepted/duplicate/informative/N/A ratio.
- Time from target selection to first serious hypothesis.
- Number of new bug classes practiced.
- Number of reports improved after rejection.
- Number of reusable checklists/scripts created.

Your goal is not more requests. Your goal is better hypotheses.

## Reading List

High signal:

- PortSwigger Research: https://portswigger.net/research
- PortSwigger Web Security Academy: https://portswigger.net/web-security
- OWASP API Security: https://owasp.org/API-Security/
- OWASP ASVS: https://owasp.org/www-project-application-security-verification-standard/
- OWASP LLM Top 10: https://owasp.org/www-project-top-10-for-large-language-model-applications/
- GitHub Security Lab: https://github.blog/security/vulnerability-research/
- Assetnote Research: https://www.assetnote.io/resources/research
- SonarSource Research: https://www.sonarsource.com/blog/
- Project Zero: https://projectzero.google/
- ZDI Blog: https://www.zerodayinitiative.com/blog

Bug bounty writeups:

- HackerOne Hacktivity: https://hackerone.com/hacktivity
- Bugcrowd blog: https://www.bugcrowd.com/blog/
- Intigriti blog: https://www.intigriti.com/researchers/blog

## Labs Checklist

- [ ] PortSwigger access control.
- [ ] PortSwigger business logic.
- [ ] PortSwigger API testing.
- [ ] PortSwigger OAuth.
- [ ] PortSwigger JWT.
- [ ] PortSwigger GraphQL.
- [ ] PortSwigger race conditions.
- [ ] PortSwigger SSRF.
- [ ] PortSwigger file upload.
- [ ] PortSwigger request smuggling.
- [ ] PortSwigger cache poisoning.
- [ ] PortSwigger web cache deception.
- [ ] PortSwigger CORS.
- [ ] PortSwigger DOM XSS.
- [ ] PortSwigger prototype pollution.
- [ ] PortSwigger Web LLM attacks.
- [ ] OWASP Juice Shop.
- [ ] OWASP crAPI.
- [ ] VAmPI.
- [ ] Damn Vulnerable GraphQL Application.
- [ ] Local toy RAG app.
- [ ] Local toy agent with tools and permissions.

## What Makes You Hard To Compete With

- You understand business workflows better than scanners.
- You build user/role/tenant matrices fast.
- You know API, mobile, GraphQL, and AI features.
- You can read source code when available.
- You can use AI to generate test cases and summarize code.
- You write reports that triage can reproduce quickly.
- You specialize but do not stay one-dimensional.

The direction is clear: keep your IDOR/business-logic edge, then add API depth, AI-era workflows, SSO/OAuth, race conditions, and source-code-assisted testing. That combination is hard for ordinary hunters and automated scanners to match.
