# Zero-Day Hunting Learning Roadmap

Last updated: 2026-05-17

This note is focused on moving from web bug bounty skills into zero-day research, especially targets similar to Pwn2Own Berlin 2026: web browsers, enterprise apps, server targets, AI databases, coding agents, local inference tools, containers, and local privilege escalation.

Use this only for legal research: local labs, your own systems, CTFs, open-source projects with responsible disclosure, or bug bounty programs where the scope explicitly allows testing.

## What Was Missing From The First Plan

- A clear lab list for hands-on practice.
- A research reading list grouped by topic.
- A target-selection method for beginners.
- A workflow for using Claude Code and OpenAI Codex safely.
- A realistic path from web bugs to zero-day research.
- A reporting template for serious findings.
- Fuzzing and code-review practice, not just web scanning.
- AI-agent and local-inference security practice, since these appeared in Pwn2Own Berlin 2026.

## Pwn2Own-Style Target Reality

Pwn2Own-style bugs usually require more than normal web findings. The official Pwn2Own Berlin 2026 rules required entries to compromise the target and demonstrate arbitrary code execution. Some categories required sandbox escape, kernel escalation, guest-to-host escape, or crossing an AI/coding-agent permission boundary.

Official rules:

- https://www.zerodayinitiative.com/Pwn2OwnBerlin2026Rules.html

Relevant target families from Pwn2Own Berlin 2026:

- Web browsers: Chrome, Edge, Safari, Firefox.
- Enterprise apps: Adobe Reader, Microsoft 365 Apps.
- Server apps: Microsoft Exchange, SharePoint, RDP/RDS.
- AI databases: Chroma.
- Coding agents: Claude Code, OpenAI Codex, Cursor.
- Local inference: Ollama, LiteLLM, LM Studio, llama.cpp.
- Containers: Docker Engine, containerd, Firecracker.
- Local privilege escalation: Windows, macOS, Red Hat Enterprise Linux.
- Virtualization: KVM, VMware ESXi, Hyper-V.
- NVIDIA tools: Megatron Bridge, NVIDIA Container Toolkit, Dynamo.

For your current level, start with:

1. Web/API and server-side bugs.
2. Open-source AI/API tools such as LiteLLM, Chroma, Ollama, llama.cpp.
3. Coding-agent sandbox and permission-boundary research in local labs.
4. Fuzzing parsers and file handlers.
5. Browser/native exploitation later.

## Core Learning Path

### Stage 1: Web Security Foundation

Goal: become strong enough that normal web bugs are automatic.

Labs:

- PortSwigger Web Security Academy: https://portswigger.net/web-security
- PortSwigger all labs index: https://portswigger.net/web-security/all-labs
- OWASP WebGoat: https://owasp.org/www-project-webgoat/
- OWASP Juice Shop: https://owasp.org/www-project-juice-shop/
- Damn Vulnerable Web Application: https://github.com/digininja/DVWA
- crAPI, vulnerable API lab: https://github.com/OWASP/crAPI
- VAmPI, vulnerable API lab: https://github.com/erev0s/VAmPI
- Damn Vulnerable GraphQL Application: https://github.com/dolevf/Damn-Vulnerable-GraphQL-Application
- SSRF training: https://github.com/swisskyrepo/SSRFmap

Topics to finish:

- Authentication bypass.
- Access control and IDOR.
- SSRF.
- File upload.
- Path traversal.
- SQL injection and NoSQL injection.
- Server-side template injection.
- Deserialization.
- XXE.
- Request smuggling.
- Web cache poisoning and deception.
- OAuth, SAML, JWT.
- Race conditions.
- WebSockets.
- API testing.
- GraphQL.
- Web LLM attacks.

Reading:

- OWASP WSTG: https://owasp.org/www-project-web-security-testing-guide/
- OWASP API Security Top 10: https://owasp.org/API-Security/
- OWASP Top 10: https://owasp.org/www-project-top-ten/
- HackTricks: https://book.hacktricks.xyz/
- PayloadsAllTheThings: https://github.com/swisskyrepo/PayloadsAllTheThings
- PortSwigger Research: https://portswigger.net/research

### Stage 2: Programming For Security Research

Goal: read code, write tests, automate analysis, and understand bugs without depending fully on AI.

Python:

- Automate the Boring Stuff: https://automatetheboringstuff.com/
- Python official tutorial: https://docs.python.org/3/tutorial/
- pytest docs: https://docs.pytest.org/
- FastAPI docs: https://fastapi.tiangolo.com/
- Flask docs: https://flask.palletsprojects.com/

JavaScript and Node.js:

- MDN JavaScript Guide: https://developer.mozilla.org/en-US/docs/Web/JavaScript/Guide
- Node.js docs: https://nodejs.org/en/learn
- Express docs: https://expressjs.com/

Git and Linux:

- Pro Git: https://git-scm.com/book/en/v2
- Linux Journey: https://linuxjourney.com/
- Missing Semester: https://missing.csail.mit.edu/

Minimum skill targets:

- Read Python and JavaScript web apps.
- Add logging.
- Run tests.
- Write small scripts.
- Understand stack traces.
- Use git branches and diffs.
- Use Docker for local labs.

### Stage 3: Code Review

Goal: find bugs by reading source code, not only by scanning.

Learn to search for:

- `subprocess`, `os.system`, `exec`, `eval`, `pickle`, `yaml.load`.
- Path joins with user input.
- Archive extraction.
- File upload and conversion.
- Deserialization.
- Template rendering.
- Auth decorators and middleware.
- Admin-only routes.
- Webhooks.
- Plugin systems.
- Import/export features.
- Model loading and file parsing.
- Shell tool execution in coding agents.
- Permission prompts and sandbox boundaries.

Resources:

- OWASP Code Review Guide: https://owasp.org/www-project-code-review-guide/
- Semgrep rules: https://semgrep.dev/explore
- CodeQL docs: https://codeql.github.com/docs/
- GitHub Security Lab research: https://github.blog/security/vulnerability-research/
- SonarSource vulnerability research: https://www.sonarsource.com/blog/
- Assetnote research: https://www.assetnote.io/resources/research

Practice method:

1. Pick one open-source project.
2. Install it locally.
3. List all routes, CLI commands, config files, parsers, and background jobs.
4. Mark every trust boundary.
5. Trace user input to risky sinks.
6. Write tests for suspicious behavior.
7. Prove impact locally.
8. Write a report.

### Stage 4: Fuzzing

Goal: find crashes, hangs, parser confusion, and logic bugs.

Start with:

- The Fuzzing Book: https://www.fuzzingbook.org/
- AFL++ docs: https://aflplus.plus/docs/
- libFuzzer docs: https://llvm.org/docs/LibFuzzer.html
- Google fuzzing tutorials: https://github.com/google/fuzzing
- OSS-Fuzz: https://google.github.io/oss-fuzz/

Practice targets:

- JSON parser.
- Markdown parser.
- YAML parser.
- ZIP/TAR extraction code.
- Image metadata parser.
- PDF parsing tools.
- URL parser.
- Template parser.
- Model/config file loader.
- API request parser.

Beginner fuzzing exercises:

- Write a Python script that mutates JSON input.
- Fuzz a local FastAPI endpoint with invalid payloads.
- Fuzz a file parser with AFL++ or libFuzzer.
- Use AddressSanitizer on a small C/C++ target.
- Minimize crashing inputs.
- Create a reproducible crash report.

### Stage 5: AI, Agent, And Local Inference Security

This is a good bridge from web security to modern Pwn2Own-style research.

Practice targets:

- LiteLLM: https://github.com/BerriAI/litellm
- Chroma: https://github.com/chroma-core/chroma
- Ollama: https://github.com/ollama/ollama
- llama.cpp: https://github.com/ggerganov/llama.cpp
- LangChain: https://github.com/langchain-ai/langchain
- LlamaIndex: https://github.com/run-llama/llama_index

Learning:

- OWASP Top 10 for LLM Applications: https://owasp.org/www-project-top-10-for-large-language-model-applications/
- OWASP AI Exchange: https://owaspai.org/
- Prompt injection guide: https://www.promptingguide.ai/risks

Bug classes to study:

- Prompt injection that crosses a real security boundary.
- Tool execution without proper permission.
- Sandbox escape.
- Unauthorized file read/write.
- Path traversal through workspace features.
- Unsafe project import.
- Unsafe MCP/tool/plugin behavior.
- Secret leakage from local config.
- SSRF through model/provider integrations.
- Auth bypass in local APIs.
- Deserialization or unsafe model loading.
- Command injection through developer tools.

Important distinction:

- A model saying bad text is usually not a zero-day.
- A model or agent causing unauthorized file access, command execution, data exfiltration, or permission-boundary bypass may be security relevant.

### Stage 6: Native, Browser, And Exploit Development

This is advanced. Do it after web/API/code-review foundations.

Resources:

- pwn.college: https://pwn.college/
- OpenSecurityTraining2: https://ost2.fyi/
- RPISEC Modern Binary Exploitation: https://github.com/RPISEC/MBE
- Ghidra: https://ghidra-sre.org/
- pwndbg: https://github.com/pwndbg/pwndbg
- LiveOverflow binary exploitation videos: https://www.youtube.com/@LiveOverflow

Topics:

- C memory model.
- Stack and heap.
- Use-after-free.
- Out-of-bounds read/write.
- Type confusion.
- Integer overflow.
- ASLR, DEP/NX, stack canaries.
- Sanitizers.
- Debuggers.
- Crash triage.
- Exploit reliability.

Browser-specific:

- JavaScript engines.
- DOM bindings.
- IPC.
- Sandboxes.
- Renderer compromise.
- Browser sandbox escape.

Useful reading:

- Google Project Zero: https://projectzero.google/
- Project Zero in-the-wild series: https://projectzero.google/2021/01/introducing-in-wild-series.html
- Project Zero AI-assisted vulnerability research: https://projectzero.google/2024/10/from-naptime-to-big-sleep.html
- ZDI blog: https://www.zerodayinitiative.com/blog
- ZDI published advisories: https://www.zerodayinitiative.com/advisories/published/

## Hands-On Lab Checklist

### Web Labs

- [ ] PortSwigger authentication labs.
- [ ] PortSwigger access control labs.
- [ ] PortSwigger SSRF labs.
- [ ] PortSwigger deserialization labs.
- [ ] PortSwigger request smuggling labs.
- [ ] PortSwigger race condition labs.
- [ ] PortSwigger OAuth labs.
- [ ] PortSwigger API testing labs.
- [ ] OWASP Juice Shop full run.
- [ ] crAPI full run.
- [ ] DVGA GraphQL full run.

### Code Review Labs

- [ ] Review a small Flask app.
- [ ] Review a small FastAPI app.
- [ ] Review a Node/Express app.
- [ ] Review an open-source API proxy.
- [ ] Review a project import/export feature.
- [ ] Review a file upload pipeline.
- [ ] Review an archive extraction feature.
- [ ] Review an auth middleware implementation.

### AI/Agent Labs

- [ ] Install LiteLLM locally and map all routes.
- [ ] Install Chroma locally and map persistence and API behavior.
- [ ] Install Ollama locally and inspect local API exposure.
- [ ] Review an MCP server locally.
- [ ] Build a toy coding agent with file read/write and shell tools, then try to design permission checks.
- [ ] Test prompt injection only in your own local lab.
- [ ] Test if model output can reach filesystem, shell, network, or secrets.

### Fuzzing Labs

- [ ] Fuzz JSON payloads against a local endpoint.
- [ ] Fuzz a URL parser.
- [ ] Fuzz ZIP/TAR extraction code.
- [ ] Fuzz a Markdown parser.
- [ ] Fuzz a YAML config loader.
- [ ] Fuzz a small C parser with AFL++.
- [ ] Compile a target with AddressSanitizer.
- [ ] Minimize one crashing input and write a report.

### Native Labs

- [ ] Complete pwn.college intro modules.
- [ ] Learn GDB basics.
- [ ] Learn pwndbg basics.
- [ ] Complete stack overflow basics.
- [ ] Complete heap basics.
- [ ] Read one Project Zero exploit-chain writeup slowly and summarize it.

## 12-Week Plan

### Weeks 1-2: Web Foundation

- Finish PortSwigger labs for auth, access control, SSRF, file upload, path traversal.
- Write short notes for each bug class:
  - root cause
  - impact
  - detection method
  - fix

### Weeks 3-4: API And Auth

- Finish API, OAuth, JWT, GraphQL, request smuggling, race condition labs.
- Complete crAPI.
- Complete one GraphQL lab.

### Weeks 5-6: Programming And Code Review

- Learn enough Python to read FastAPI/Flask.
- Learn enough JavaScript to read Express.
- Pick one small open-source project.
- Map routes and auth checks.
- Write at least 10 tests for weird input.

### Weeks 7-8: AI/API Target Practice

- Install LiteLLM or Chroma locally.
- Map all input surfaces.
- Review auth, filesystem, provider integrations, config loading, and plugin/tool behavior.
- Use Codex/Claude to explain code, but verify manually.

### Weeks 9-10: Fuzzing

- Read the beginner chapters of The Fuzzing Book.
- Fuzz one local parser.
- Fuzz one local HTTP endpoint.
- Learn crash minimization.
- Write one mock vulnerability report.

### Weeks 11-12: Research Simulation

- Pick one target and stay with it for two weeks.
- Create an attack-surface map.
- Create a suspicious-code list.
- Write tests.
- Fuzz one component.
- Read related advisories.
- Produce a final research notebook, even if you do not find a real bug.

## Daily Research Workflow

Use this every day:

1. Choose one target and one component.
2. Read the docs as a user.
3. Run it locally.
4. Map inputs and trust boundaries.
5. Search risky sinks.
6. Trace attacker-controlled input.
7. Write tests.
8. Try malformed input.
9. Check logs and stack traces.
10. Decide if the behavior crosses a security boundary.
11. Document what you learned.

Do not jump targets too often. Depth finds zero-days.

## Target Selection Matrix

Good beginner zero-day research targets:

- Open-source.
- Easy to run locally.
- Has tests.
- Has HTTP/API surface.
- Has file parsing or import/export.
- Has auth or permissions.
- Has recent new features.
- Has plugins, tools, or integrations.
- Has code in Python, JavaScript, Go, or Rust.

Avoid at first:

- Closed-source enterprise software.
- Browsers.
- Kernels.
- Hypervisors.
- Complex C++ codebases.
- Targets you cannot legally test.

## How To Use Claude Code And OpenAI Codex

Use AI for acceleration, not blind trust.

Good tasks for AI:

- Explain code.
- Map routes.
- Find risky sinks.
- Generate tests.
- Build fuzz harnesses.
- Summarize advisories.
- Turn notes into reports.
- Compare patch diffs.

Bad tasks for AI:

- Blindly generating exploit code.
- Deciding if something is definitely a zero-day without verification.
- Testing real targets without authorization.
- Replacing your own understanding.

Useful prompts:

```text
I am doing authorized local security research on this repository.
Map the attack surface: HTTP routes, CLI commands, file parsers, config loaders, auth checks, subprocess calls, filesystem access, deserialization, template rendering, plugin systems, and external service calls.
Give me files and functions to review first.
```

```text
Explain this function for a beginner security researcher.
Tell me what inputs it trusts, what assumptions it makes, what security boundary exists, and what could go wrong.
```

```text
Review this code for trust-boundary mistakes.
Focus on user-controlled input reaching filesystem, shell, network, deserialization, templates, auth checks, or tool execution.
Do not write exploit code. Give me test ideas and file references.
```

```text
Generate local-only pytest tests for this handler.
Include path traversal, null bytes, Unicode, oversized input, malformed JSON, symlinks, missing auth, and unexpected content types.
```

```text
Help me build a local fuzzing harness for this parser.
Keep it safe and local. Show how to run it, detect crashes, and minimize inputs.
```

```text
Turn these notes into a responsible disclosure report:
summary, affected version, impact, root cause, reproduction in local lab, CWE, suggested fix, and timeline.
```

## Patch Diffing Workflow

Patch diffing means studying security fixes to understand what was vulnerable.

Use this legally and responsibly. Do not weaponize fresh bugs against real systems.

Steps:

1. Watch releases and changelogs.
2. Compare old and new versions.
3. Identify security-relevant changed files.
4. Ask why the patch was needed.
5. Build a local reproduction if allowed.
6. Learn the bug class.
7. Search for similar patterns in other code.

Tools:

- `git diff`
- GitHub compare view.
- Semgrep.
- CodeQL.
- ripgrep.
- local tests.

Good prompt:

```text
Compare this patch and explain what security issue it appears to fix.
Identify the vulnerable assumption, affected input path, and similar patterns to search for.
Do not generate an exploit.
```

## Vulnerability Report Template

```markdown
# Title

## Summary

Short explanation of the vulnerability and affected component.

## Affected Version

Product:
Version:
Commit:
Configuration:

## Impact

Explain what an attacker can do.

## Preconditions

Authentication needed:
User role:
Feature enabled:
Network/local access:

## Root Cause

Explain the vulnerable code path and broken assumption.

## Reproduction Steps

1. Run the target locally.
2. Configure the test environment.
3. Send the test input.
4. Observe the result.

## Evidence

Logs, stack traces, screenshots, PCAP, or test output.

## CWE

Example: CWE-22 Path Traversal, CWE-918 SSRF, CWE-502 Deserialization, CWE-78 Command Injection.

## Suggested Fix

Describe validation, authorization, sandboxing, safer APIs, or parser hardening.

## Timeline

Date found:
Date reported:
Vendor response:
Patch date:
Disclosure date:
```

## Reading List By Topic

### General Vulnerability Research

- Project Zero blog: https://projectzero.google/
- Project Zero disclosure FAQ: https://projectzero.google/vulnerability-disclosure-faq.html
- ZDI blog: https://www.zerodayinitiative.com/blog
- ZDI advisories: https://www.zerodayinitiative.com/advisories/published/
- GitHub Security Lab: https://github.blog/security/vulnerability-research/
- SonarSource blog: https://www.sonarsource.com/blog/
- Assetnote research: https://www.assetnote.io/resources/research

### Web Research

- PortSwigger Research: https://portswigger.net/research
- Orange Tsai research: https://blog.orange.tw/
- James Kettle research: https://portswigger.net/research
- HackTricks: https://book.hacktricks.xyz/

### Fuzzing

- The Fuzzing Book: https://www.fuzzingbook.org/
- AFL++ docs: https://aflplus.plus/docs/
- libFuzzer docs: https://llvm.org/docs/LibFuzzer.html
- OSS-Fuzz docs: https://google.github.io/oss-fuzz/

### Native Exploitation

- pwn.college: https://pwn.college/
- OpenSecurityTraining2: https://ost2.fyi/
- RPISEC MBE: https://github.com/RPISEC/MBE
- Ghidra: https://ghidra-sre.org/
- pwndbg: https://github.com/pwndbg/pwndbg

### AI Security

- OWASP LLM Top 10: https://owasp.org/www-project-top-10-for-large-language-model-applications/
- OWASP AI Exchange: https://owaspai.org/
- Project Zero Big Sleep/Naptime post: https://projectzero.google/2024/10/from-naptime-to-big-sleep.html

## Safe Practice Rules

- Test only where you have permission.
- Prefer local labs and open-source projects.
- Do not scan random public IPs.
- Do not test destructive payloads.
- Do not access real user data.
- Do not publish unpatched vulnerability details without following disclosure rules.
- Keep clean notes from the start.
- If a bug might be real, stop expanding impact and prepare a responsible report.

## Your Best Immediate Path

Your strongest route is:

1. Finish advanced web labs.
2. Learn Python and JavaScript code reading.
3. Practice code review on open-source API apps.
4. Pick LiteLLM, Chroma, or Ollama as a research target.
5. Use Codex/Claude to map code and generate tests.
6. Learn fuzzing on small parsers.
7. Read one serious vulnerability writeup every week.
8. Write reports even for lab bugs.

Do this for 90 days before trying browser, kernel, or hypervisor research.
