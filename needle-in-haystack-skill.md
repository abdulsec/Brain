# Needle in the Haystack — LLM-Assisted Vulnerability Discovery

## Overview
Systematic methodology for finding real vulnerabilities in any codebase using LLM-specific techniques. Based on the principle that LLMs perform worst with broad prompts and best when given thin slices, adversarial framing, and invariant-violation tasks. Proven across Parse Server, HonoJS, ElysiaJS, Harden Runner, BullFrog, and Better-Hub.

**Core principle:** Minimal persistent scaffolding, maximal targeted exploration. If your scaffolding becomes the haystack, the vulnerability becomes the needle.

## Trigger
Use when the user:
- Wants to audit a new target/codebase for vulnerabilities
- Says "needle in haystack", "audit this", "find bugs in this", "hunt on this"
- Asks to apply the methodology to a specific target
- Wants to find vulnerabilities beyond running automated scanners

## When NOT to Use
- The user just wants to run Nuclei/semgrep/automated scanners (use bug-hunting skill instead)
- The user wants to test a single specific bug class on a known endpoint (use idor skill, etc.)
- The target has no source code available (methodology depends on source access)

---

## Phase 0: Intake (5% of effort)

Ask the user these questions in one shot:

1. **Target:** What's the codebase/app? (path to source, repo URL, or container image)
2. **Attack surface:** Web API? Mobile backend? Desktop app? CLI tool? Library?
3. **Source access:** Do we have full source, decompiled JARs, webpack bundles, or container-extracted code?
4. **Live instance:** Is there a running instance we can test against? URL? Credentials?
5. **Prior art:** Any known CVEs, past bug bounty reports, or security assessments for this software?
6. **Scope:** Any specific areas to focus on or avoid?

Record answers in a temporary notes file at `~/audit/<target>/notes/00-intake.md`.

---

## Phase 1: Threat Model from CVEs (10% of effort)

**DO NOT start by asking "find all vulnerabilities."** This fails because there's no threat model — the LLM produces generic CWE noise.

### Step 1.1: Gather Historical Vulnerabilities

Spawn a **threat-model agent** to gather CVEs and historical bugs:

```
Agent: threat-model
Task: Gather all known CVEs and disclosed vulnerabilities for <target>.
Search: NVD API, GitHub Security Advisories, HackerOne hacktivity, Google Project Zero.
Output: CVE ID, description, root cause class, affected component, patch diff if available.
```

### Step 1.2: Generate Threat Model

Feed the CVE list to the LLM and ask:

> "Based on these historical vulnerabilities, craft a threat model of plausible bug classes. For each class, describe: the invariant being violated, the attacker model, the affected trust boundaries, and the pattern to look for in code."

### Step 1.3: Map the Environment

Extract from the codebase or ask the LLM to identify:
- **Entry points:** HTTP routes, RPC handlers, message consumers, CLI entrypoints, scheduled jobs
- **Trust boundaries:** Browser↔Server, Service↔Service, Plugin↔Host, Sandbox↔Privileged
- **High-risk operations:** Deserialization, templating, native bindings, authz checks, parsing untrusted input
- **Attacker models:** Remote unauthenticated, authenticated low-privileged, cross-tenant, local privilege escalation

Save to `~/audit/<target>/notes/01-threat-model.md`.

**Invariant format:** Every invariant must be falsifiable. Not "the system is secure" but "only authenticated users can access endpoints matching /api/admin/*".

---

## Phase 2: Thin Slicing + Slice Audit (60-80% of effort)

This is where findings actually happen. Each slice should be small enough that context doesn't rot.

### Step 2.1: Choose a Slice

Valid slices (pick ONE at a time):
- Auth & session management
- Route-level authorization
- Request parsing & input validation
- File upload & processing
- Deserialization (JSON, XML, binary, custom formats)
- SSO / OAuth integration
- API token management
- Password reset / account recovery
- Plugin/module boundary
- Sandbox / privilege separation
- Data export / report generation
- WebSocket / real-time messaging
- CLI tools & administrative scripts

### Step 2.2: For Each Slice — Run the Slice Audit

Spawn a **slice-audit agent** with this exact protocol:

**Step A — Explore the decision tree:**
> "Walk through exactly what happens when <scenario>. Don't look for bugs yet. Just trace the data flow and control flow. What are the edge cases? What happens in fallback paths? What defaults kick in when things aren't configured perfectly?"

**Step B — Identify invariants:**
> "List every invariant, assumption, or precondition that this code relies on for correctness. Be exhaustive. Include implicit assumptions."

**Step C — Violate each invariant:**
> "For each invariant, determine if an attacker can violate it. If yes, write a concrete proof-of-concept."

**Step D — Rotate through the Prompt Injection Playbook:**

For each invariant that holds under normal analysis, apply these techniques in order:

| # | Technique | Prompt Pattern |
|---|---|---|
| 1 | **Assert the vuln exists** | "This function has at least 2 security issues. Find them." |
| 2 | **Ask for the exploit** | "Write a PoC request that bypasses this validation." |
| 3 | **Prime as adversary** | "You are a red team operator paid to break this. Find the weakness." |
| 4 | **False anchoring** | "I already found one bug in this module. There are others. Find them." |
| 5 | **Invert the question** | "How would you break this?" (not "Is this secure?") |
| 6 | **Assume developer mistake** | "The developer made a mistake here. What is it?" |
| 7 | **Compare to known-good** | "How does this differ from the standard secure implementation?" |
| 8 | **Escalate iteratively** | "Set aside <already-found>. What OTHER bug classes exist here?" |
| 9 | **Constrain attacker model** | "You can only send unauthenticated HTTP requests. How do you break this?" |
| 10 | **Decompose + violate invariants** | List all assumptions → check each one → violate individually |

### Step 2.3: Document Findings

Save each finding to `~/audit/<target>/notes/02-findings.md` with:
- Severity (Critical/High/Medium/Low/Info)
- Endpoint/component
- Root cause
- Proof of concept (curl command or code snippet)
- Impact statement

---

## Phase 3: Verification (20-30% of effort)

**Never trust "the model says it's vulnerable."** Verify every finding.

### Verification Methods (in priority order):

1. **Live test against running instance** — curl, browser MCP, or Caido
2. **Grep-based invariant check** — search the codebase for patterns that would confirm/deny the finding
3. **Trace the code path** — verify the vulnerable code actually executes in the reported scenario
4. **Write a minimal reproduction** — small script that triggers the vulnerability

### Verification Protocol:

For each finding:
1. Spawn a **verify agent** with the finding details
2. It tests against the live instance (if available) OR traces the code to confirm
3. It reports: CONFIRMED / FALSE POSITIVE / NEEDS MORE INVESTIGATION
4. For CONFIRMED: write the full report. For FALSE POSITIVE: document why and move on.

---

## Token Budget

- <10% on scaffolding (threat model, invariants)
- 60-80% on slice audits in focused contexts
- 20-30% on verification loops

**If your scaffolding becomes the haystack, the vulnerability becomes the needle.**

---

## Quick Start Examples

### Example 1: Full Target Audit
```
User: "Audit the UISP codebase at ~/audit/ui/uisp/"
Claude: [Runs Phase 0 intake → Phase 1 threat model agent → User picks a slice → Phase 2 slice audit agent → Phase 3 verify]
```

### Example 2: Single Slice Deep Dive
```
User: "I want to audit the auth system. Source is at ~/audit/ui/uisp/unms-api/api.js"
Claude: [Skips Phase 0/1 if context exists, runs Phase 2 slice audit on auth slice, focuses on prompt injection playbook techniques 2, 3, 5, 6]
```

### Example 3: Single Technique
```
User: "Use technique #6 (assume developer mistake) on the token exchange handler in api.js"
Claude: [Loads the handler code, applies adversarial framing, outputs findings]
```

---

## Subagents

This skill comes with specialized subagents for each phase:

| Agent | File | When to Use |
|---|---|---|
| **threat-model** | `agents/threat-model.md` | Phase 1 — gather CVEs, build threat model |
| **slice-audit** | `agents/slice-audit.md` | Phase 2 — deep-dive into one thin slice |
| **prompt-injection** | `agents/prompt-injection.md` | Phase 2 alternate — run the 10-technique playbook on a specific function |
| **verify** | `agents/verify.md` | Phase 3 — confirm or disprove a finding |

### How to Use Subagents

```bash
# From within the skill, spawn agents like:
Agent(description="Threat model for <target>", subagent_type="general-purpose", prompt="<content from agents/threat-model.md with target filled in>")

Agent(description="Slice audit: auth", subagent_type="general-purpose", prompt="<content from agents/slice-audit.md with slice details>")

Agent(description="Verify finding: open redirect", subagent_type="general-purpose", prompt="<content from agents/verify.md with finding details>")
```

The agents are prompt templates — fill in the target/slice/finding details and spawn them.
