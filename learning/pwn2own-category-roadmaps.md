# Pwn2Own Category Roadmaps

Last updated: 2026-05-17

Source rule page:

- https://www.zerodayinitiative.com/Pwn2OwnBerlin2026Rules.html

The Pwn2Own Berlin 2026 contest was scheduled for May 14-16, 2026. The next event rules may change, so always re-check the latest ZDI rules before choosing a final target. This roadmap uses the Berlin 2026 categories as a training structure.

The ten categories:

1. Virtualization
2. Web Browser
3. Enterprise Applications
4. Server
5. Local Escalation of Privilege
6. Container
7. AI Database
8. Coding Agent
9. Local Inference
10. NVIDIA

Important Pwn2Own reality:

- A normal web bug is usually not enough.
- Most categories require arbitrary code execution.
- Sandboxes, guest/host boundaries, tenant boundaries, and permission boundaries matter.
- You need a reliable local reproduction, a clear vulnerability root cause, a chain, and a strong whitepaper.
- Work only in local labs, owned infrastructure, open-source projects, vendor programs, or contest-approved environments.

## Best Starting Order For You

Given your background in IDOR, business logic, and privilege escalation, use this order:

1. Coding Agent
2. Local Inference
3. AI Database
4. Server
5. Enterprise Applications
6. Container
7. NVIDIA
8. Web Browser
9. Local Escalation of Privilege
10. Virtualization

Why:

- Coding Agent, Local Inference, AI Database, and Server have more overlap with web/API, auth, filesystem, workflow, and integration security.
- Browser, kernel, hypervisor, and virtualization require deeper native exploitation and exploit reliability.

## Universal Pwn2Own Research Workflow

Use this for every category:

1. Read the rules and exact target requirements.
2. Build a local lab matching target versions as closely as possible.
3. Learn normal user workflows.
4. Map attack surfaces.
5. Identify trust boundaries.
6. Read source code if available.
7. Search patch history and advisories.
8. Fuzz parsers and protocol inputs.
9. Triage crashes or abnormal behavior.
10. Prove exploitability safely in the lab.
11. Build a reliable demonstration.
12. Write a technical whitepaper.
13. Prepare a backup target or fallback chain.

Minimum submission package:

- Affected product and version.
- Environment and configuration.
- Vulnerability root cause.
- Attack vector.
- Reproduction steps.
- Reliability notes.
- Crash logs or execution proof.
- Exploit chain diagram.
- Security boundary crossed.
- Suggested mitigation.

## Category 1: Coding Agent

Berlin 2026 targets:

- Anthropic Claude Code
- OpenAI Codex
- Cursor

Contest expectation from rules:

- A successful entry must interact with a contestant-controlled resource such as a web page, repository, or media file.
- The vector must be a common coding-agent use case.
- The attempt must obtain arbitrary code execution and perform actions outside the intended sandbox and permissions boundaries.
- Unsafe permissionless modes are out of scope.
- Simple jailbreaks or prompt outputs that do not cross a security boundary are out of scope.

Why this is a strong category for you:

- It mixes web, repo parsing, file handling, shell/tool execution, permission prompts, and business-logic-style boundary mistakes.
- Your privilege escalation mindset applies directly: user intent vs agent action vs sandbox permission.

Core skills:

- Node.js and TypeScript basics.
- Python basics.
- Shell command behavior.
- Git internals and repository structure.
- Filesystem permissions.
- Sandbox design.
- Prompt injection and indirect prompt injection.
- Tool permission models.
- MCP/tool server basics.
- Package manager behavior: npm, pip, uv, pnpm, yarn.
- IDE/editor extension models.

Attack surfaces to study safely:

- Repository ingestion.
- Markdown and documentation parsing.
- Project configuration files.
- Build scripts.
- Test commands.
- Package manager lifecycle scripts.
- Agent tool calls.
- Permission prompt logic.
- Workspace trust.
- MCP configuration.
- Symlinks and path traversal.
- Git hooks and submodules.
- Generated files.
- Terminal command construction.
- Browser preview features.
- Image/media/document parsing if the agent accepts them.

Practice labs:

- Build a toy coding agent with read-only file tools, write tools, and shell tools.
- Add a permission prompt system to the toy agent.
- Try to design tests that ensure the agent cannot write outside workspace.
- Try to design tests that ensure the agent cannot execute blocked commands.
- Review open-source MCP servers in local labs.
- Review local-only VS Code extension security concepts.

Reading:

- OWASP Top 10 for LLM Applications: https://owasp.org/www-project-top-10-for-large-language-model-applications/
- OWASP GenAI Security Project: https://genai.owasp.org/
- PortSwigger Web LLM attacks: https://portswigger.net/web-security/learning-paths/llm-attacks
- Model Context Protocol documentation: https://modelcontextprotocol.io/
- Git documentation: https://git-scm.com/docs
- Node.js security best practices: https://nodejs.org/en/learn/getting-started/security-best-practices

Research tasks:

- Map every place user-controlled repo content influences agent behavior.
- Map every place model output influences tool execution.
- Map every place a permission decision is made.
- Map every filesystem path check.
- Map every subprocess invocation.
- Map workspace boundary logic.
- Create regression tests for sandbox boundary conditions.

Submission-ready bar:

- Not just prompt injection.
- Not just the model saying unsafe text.
- Must cross a real boundary: command execution, unauthorized file access, sandbox escape, or permission bypass in default configuration.

## Category 2: Local Inference

Berlin 2026 targets:

- Ollama
- LiteLLM
- LM Studio
- llama.cpp

Why this is a good category for web hunters:

- Local inference products expose HTTP APIs, model loading, provider integrations, file handling, plugin-like features, and local network behavior.
- LiteLLM especially overlaps with API gateway security.

Core skills:

- Python web APIs.
- Go basics for Ollama.
- C/C++ basics for llama.cpp.
- HTTP API testing.
- Auth and local network exposure.
- File upload and model file parsing.
- SSRF.
- Deserialization and config parsing.
- Path traversal.
- Local service threat modeling.

Attack surfaces:

- HTTP API routes.
- Model pull/import/load.
- Model metadata parsing.
- Prompt templates.
- Provider proxy routes.
- Admin/config endpoints.
- API keys.
- Web UI routes.
- File uploads.
- Local file references.
- Plugin/tool integrations.
- Default bind addresses and CORS.
- Logs and debug endpoints.
- Docker deployments.

Practice targets:

- Ollama: https://github.com/ollama/ollama
- LiteLLM: https://github.com/BerriAI/litellm
- llama.cpp: https://github.com/ggerganov/llama.cpp
- LocalAI: https://github.com/mudler/LocalAI

Labs:

- Run Ollama locally and map all endpoints.
- Run LiteLLM locally and test auth/config boundaries.
- Run llama.cpp server locally and fuzz API inputs.
- Build a local proxy that imitates an inference gateway.
- Test local-only SSRF and path traversal patterns in a toy app.

Reading:

- OWASP API Security Top 10: https://owasp.org/API-Security/
- OWASP LLM Top 10: https://owasp.org/www-project-top-10-for-large-language-model-applications/
- FastAPI docs: https://fastapi.tiangolo.com/
- Go web security basics: https://go.dev/doc/security/
- The Fuzzing Book: https://www.fuzzingbook.org/

Research tasks:

- Code-review API handlers.
- Find trust boundaries between UI, API, model runtime, and filesystem.
- Fuzz model metadata and config parsing.
- Test API auth and admin assumptions.
- Test unsafe local network exposure.
- Review Docker deployment defaults.

Submission-ready bar:

- Arbitrary code execution or serious local boundary compromise.
- A reliable chain from supported interaction to code execution or equivalent contest-required impact.

## Category 3: AI Database

Berlin 2026 targets:

- Chroma
- Postgres pgvector
- Oracle Autonomous AI Database

Why this is relevant to you:

- AI databases combine API security, tenant boundaries, auth, document ingestion, metadata filters, embeddings, and query behavior.
- IDOR and multi-tenant isolation skills apply directly.

Core skills:

- Database fundamentals.
- SQL and PostgreSQL.
- Python APIs.
- Vector database concepts.
- Metadata filtering.
- Multi-tenant SaaS design.
- Auth and API key design.
- RAG architectures.
- Data ingestion pipelines.

Attack surfaces:

- Collection creation and deletion.
- Document insert/update/delete.
- Metadata filters.
- Query APIs.
- Tenant/project/workspace IDs.
- Embedding functions.
- Persistence paths.
- Import/export.
- Admin endpoints.
- Backup/restore.
- Connectors.
- SQL extensions.
- API auth.

Practice targets:

- Chroma: https://github.com/chroma-core/chroma
- pgvector: https://github.com/pgvector/pgvector
- Qdrant: https://github.com/qdrant/qdrant
- Weaviate: https://github.com/weaviate/weaviate
- Milvus: https://github.com/milvus-io/milvus

Labs:

- Build a toy multi-tenant RAG app.
- Add two tenants and intentionally weak metadata filters.
- Test cross-tenant retrieval.
- Test collection IDOR.
- Test API key scope enforcement.
- Test import/export isolation.
- Run Chroma locally and map API behavior.
- Run Postgres with pgvector and study extension behavior.

Reading:

- Chroma docs: https://docs.trychroma.com/
- pgvector README: https://github.com/pgvector/pgvector
- PostgreSQL security docs: https://www.postgresql.org/docs/current/security.html
- OWASP API Security Top 10: https://owasp.org/API-Security/
- OWASP LLM Top 10: https://owasp.org/www-project-top-10-for-large-language-model-applications/

Research tasks:

- Map tenant isolation design.
- Review metadata filter enforcement.
- Review auth middleware.
- Review persistence and file paths.
- Review embedding and model loading paths.
- Fuzz API payloads and query filters.
- Search for injection into SQL, filters, templates, or file paths.

Submission-ready bar:

- The contest requires arbitrary code execution, so pure cross-tenant data leaks may not qualify unless chained.
- Focus on parser bugs, unsafe extension behavior, deserialization, admin/API RCE paths, or ingestion-to-execution chains.

## Category 4: Server

Berlin 2026 targets:

- Microsoft Windows RDP/RDS
- Microsoft Exchange
- Microsoft SharePoint

Why this is high value:

- Server category has large prizes and high impact.
- It is difficult because targets are complex and often closed-source.

Best fit for your background:

- SharePoint first.
- Exchange second.
- RDP/RDS later.

Core skills:

- Windows Server administration.
- Active Directory basics.
- IIS.
- .NET and ASP.NET.
- PowerShell.
- Authentication protocols.
- NTLM, Kerberos, OAuth, SAML.
- Deserialization.
- SSRF.
- XML and SOAP.
- File upload and parsing.
- Patch diffing.

Attack surfaces:

- Web endpoints.
- Auth flows.
- Deserialization points.
- File upload and document parsing.
- Search/indexing.
- Admin APIs.
- Web services.
- Exchange Web Services.
- SharePoint REST APIs.
- Workflow engines.
- Template/page rendering.
- Connectors and integrations.

Labs:

- Build a local Windows Server lab.
- Install SharePoint trial/dev environment if available.
- Study Exchange lab deployments in an isolated environment.
- Practice with intentionally vulnerable ASP.NET apps.
- Build a toy .NET app with auth, roles, file upload, and XML parsing.

Reading:

- Microsoft Security Response Center: https://msrc.microsoft.com/
- Microsoft Exchange Server docs: https://learn.microsoft.com/en-us/exchange/
- Microsoft SharePoint docs: https://learn.microsoft.com/en-us/sharepoint/
- Microsoft RDS docs: https://learn.microsoft.com/en-us/windows-server/remote/remote-desktop-services/
- ZDI advisories: https://www.zerodayinitiative.com/advisories/published/
- Project Zero: https://projectzero.google/

Research tasks:

- Study historical Exchange and SharePoint CVEs.
- Patch-diff Microsoft advisories when patches are available.
- Build a timeline of bug classes by product.
- Practice .NET deserialization and auth flow review in labs.
- Map where user input reaches server-side parsers.

Submission-ready bar:

- Remote arbitrary code execution from contest network.
- Reliable chain against the exact configured target.
- Closed-source targets require strong reverse engineering and patch-diffing discipline.

## Category 5: Enterprise Applications

Berlin 2026 targets:

- Adobe Reader
- Microsoft 365 Apps: Word, Excel, PowerPoint, Outlook

Special Berlin 2026 add-ons:

- Microsoft 365 Copilot data exfiltration.
- Microsoft 365 Copilot action execution.

Why this is relevant:

- Office and Reader exploitation blends document parsing, sandboxing, macro restrictions, preview modes, file format complexity, and now AI/Copilot integrations.

Core skills:

- File format internals.
- Document parsing.
- Windows and macOS application behavior.
- Sandboxes.
- PDF structure.
- Office Open XML.
- Outlook email rendering.
- Protected View.
- COM/OLE basics.
- Fuzzing.
- Crash triage.
- Copilot/Microsoft Graph concepts.

Attack surfaces:

- PDF objects and JavaScript.
- Office documents.
- Embedded media.
- Fonts.
- Templates.
- OLE/ActiveX legacy paths.
- Outlook email preview/rendering.
- Protected View transitions.
- Copilot grounding and actions.
- Microsoft Graph integrations.

Labs:

- Learn PDF structure with benign local files.
- Learn Office Open XML by unpacking `.docx`, `.xlsx`, `.pptx`.
- Build document fuzzing harnesses for open-source parsers first.
- Practice with LibreOffice or open-source document parsers before closed-source targets.
- Build a local app that parses uploaded documents and passes extracted text to an AI assistant.

Reading:

- Adobe PDF specification: https://developer.adobe.com/document-services/docs/overview/pdf-extract-api/
- Microsoft Open Specifications: https://learn.microsoft.com/en-us/openspecs/
- Microsoft Graph docs: https://learn.microsoft.com/en-us/graph/
- Microsoft 365 Copilot extensibility docs: https://learn.microsoft.com/en-us/microsoft-365-copilot/extensibility/
- The Fuzzing Book: https://www.fuzzingbook.org/
- AFL++ docs: https://aflplus.plus/docs/

Research tasks:

- Study historic Reader and Office CVEs.
- Build corpus-based fuzzing habits.
- Learn crash minimization.
- Map document-to-AI data flow.
- Study Outlook-specific attack surfaces.

Submission-ready bar:

- Launch must follow rules, often by opening/viewing a document or email.
- Protected View/Protected Mode matters.
- For Copilot add-ons, the base compromise must lead to unauthorized Copilot-powered data access or actions.

## Category 6: Container

Berlin 2026 targets:

- containerd
- Docker Engine
- Firecracker

Contest expectation:

- Launch from guest container or microVM.
- Execute arbitrary code on the host operating system.

Core skills:

- Linux namespaces.
- cgroups.
- seccomp.
- capabilities.
- AppArmor and SELinux basics.
- OCI image format.
- containerd architecture.
- Docker daemon architecture.
- runc.
- Firecracker microVM model.
- Kernel attack surface from containers.

Attack surfaces:

- Image parsing.
- Container runtime APIs.
- Mounts and volumes.
- Overlay filesystems.
- Device access.
- Network namespaces.
- Seccomp profiles.
- Cgroup behavior.
- Runtime shims.
- Snapshotters.
- Container escape primitives.
- Firecracker jailer and device model.

Practice targets:

- Docker Engine: https://github.com/moby/moby
- containerd: https://github.com/containerd/containerd
- runc: https://github.com/opencontainers/runc
- Firecracker: https://github.com/firecracker-microvm/firecracker

Labs:

- Learn Docker internals.
- Run containers with different capabilities and seccomp profiles.
- Build a container escape lab using intentionally vulnerable examples.
- Study runc historical CVEs.
- Build and inspect OCI images.
- Run Firecracker locally in an isolated lab.

Reading:

- Docker security docs: https://docs.docker.com/engine/security/
- containerd docs: https://containerd.io/docs/
- OCI runtime spec: https://github.com/opencontainers/runtime-spec
- Firecracker docs: https://github.com/firecracker-microvm/firecracker/tree/main/docs
- Linux capabilities: https://man7.org/linux/man-pages/man7/capabilities.7.html
- Linux namespaces: https://man7.org/linux/man-pages/man7/namespaces.7.html

Research tasks:

- Review runtime boundary assumptions.
- Study host file exposure paths.
- Review image extraction and mount handling.
- Fuzz runtime APIs in local labs.
- Study container escape CVEs and patch diffs.

Submission-ready bar:

- Escape from container/microVM to host code execution.
- No reliance on intentionally privileged or unsafe configuration unless rules allow it.

## Category 7: NVIDIA

Berlin 2026 targets:

- Megatron Bridge
- NVIDIA Container Toolkit
- Dynamo

Rules note:

- For network-accessible targets, launch from contestant laptop.
- For NVIDIA Container Toolkit, launch from a crafted container image and execute arbitrary code on host.
- For Megatron Bridge, pickle deserialization and `trust_remote_code=true` issues were out of scope in Berlin 2026.

Core skills:

- Python ML ecosystem.
- PyTorch basics.
- Distributed training concepts.
- Container security.
- GPU container runtime.
- Kubernetes basics.
- Model loading and checkpoint formats.
- YAML/config parsing.
- RPC and service APIs.
- Supply chain security.

Attack surfaces:

- Container images.
- Model artifacts.
- Config files.
- Distributed training communication.
- Checkpoint loading.
- REST/gRPC APIs.
- GPU runtime hooks.
- Container runtime integration.
- Mounts and driver access.
- Python package loading.
- Dataset ingestion.

Practice targets:

- NVIDIA Container Toolkit: https://github.com/NVIDIA/nvidia-container-toolkit
- NVIDIA Megatron Bridge: https://github.com/NVIDIA-NeMo/Megatron-Bridge
- NVIDIA Dynamo: https://github.com/ai-dynamo/dynamo
- PyTorch: https://github.com/pytorch/pytorch

Labs:

- Build a local GPU-container lab if hardware is available.
- If no GPU, still study image/runtime code paths and docs.
- Build a toy ML service that loads configs, datasets, and model files.
- Test config parsing and file path boundaries.
- Study container toolkit architecture.

Reading:

- NVIDIA Container Toolkit docs: https://docs.nvidia.com/datacenter/cloud-native/container-toolkit/
- NVIDIA security bulletins: https://www.nvidia.com/en-us/security/
- PyTorch security policy: https://github.com/pytorch/pytorch/security
- Kubernetes security docs: https://kubernetes.io/docs/concepts/security/
- Docker security docs: https://docs.docker.com/engine/security/

Research tasks:

- Map container-to-host boundaries.
- Review runtime hooks and mounts.
- Review config parsing.
- Review network APIs.
- Study excluded bug classes so you do not waste time on out-of-scope findings.

Submission-ready bar:

- For NV Container Toolkit: crafted container image to host code execution.
- For network targets: reliable compromise of exposed service target.
- Avoid relying on explicitly out-of-scope unsafe ML loading modes.

## Category 8: Web Browser

Berlin 2026 targets:

- Google Chrome
- Microsoft Edge
- Apple Safari
- Mozilla Firefox

Possible outcomes:

- Renderer-only compromise.
- Sandbox escape.
- Kernel escalation add-on depending on target.

This is advanced. Do not start here unless you are ready for C++, browser internals, fuzzing, and exploit development.

Core skills:

- C++.
- JavaScript engine internals.
- JIT concepts.
- DOM and browser architecture.
- IPC.
- Sandboxing.
- Memory corruption.
- Type confusion.
- Use-after-free.
- Out-of-bounds read/write.
- Exploit mitigations.
- Debuggers and sanitizers.

Attack surfaces:

- JavaScript engines.
- DOM APIs.
- WebAssembly.
- WebGPU/WebGL.
- CSS engines.
- Media parsers.
- Fonts.
- IPC interfaces.
- Browser extensions.
- PDF/browser viewers.
- Sandbox brokers.

Practice targets:

- Chromium: https://www.chromium.org/developers/
- WebKit: https://webkit.org/
- Firefox/Gecko: https://firefox-source-docs.mozilla.org/
- V8: https://v8.dev/
- SpiderMonkey docs: https://firefox-source-docs.mozilla.org/js/index.html

Labs:

- pwn.college: https://pwn.college/
- OpenSecurityTraining2: https://ost2.fyi/
- RPISEC Modern Binary Exploitation: https://github.com/RPISEC/MBE
- Learn browser fuzzing on debug builds.
- Reproduce old patched browser bugs in local historical builds.

Reading:

- Project Zero: https://projectzero.google/
- V8 blog: https://v8.dev/blog
- WebKit blog: https://webkit.org/blog/
- Mozilla security blog: https://blog.mozilla.org/security/
- Chrome security blog: https://security.googleblog.com/

Research tasks:

- Learn one engine deeply.
- Build debug versions.
- Learn crash triage.
- Study old CVEs and patch diffs.
- Learn sandbox architecture.
- Start with renderer bugs before sandbox escapes.

Submission-ready bar:

- Reliable exploit served locally as required by rules.
- Renderer-only may qualify for lower prize.
- Full chain requires sandbox escape or kernel escalation depending on selected target and option.

## Category 9: Local Escalation Of Privilege

Berlin 2026 targets:

- Red Hat Enterprise Linux for Workstations
- Microsoft Windows 11
- Apple macOS

Contest expectation:

- Launch from non-admin/non-root account.
- Leverage a kernel vulnerability to execute arbitrary code with elevated privileges.

Core skills:

- OS internals.
- Kernel architecture.
- System calls.
- Drivers.
- Filesystems.
- IPC.
- Memory management.
- Race conditions.
- Kernel debugging.
- Exploit mitigations.

Linux:

- Kernel modules.
- eBPF.
- io_uring.
- netfilter.
- filesystems.
- namespaces and cgroups.
- SELinux.

Windows:

- Windows kernel objects.
- Win32k.
- Drivers.
- ALPC.
- token privileges.
- sandboxing.
- ETW basics.

macOS:

- XNU.
- IOKit.
- sandbox profiles.
- launchd.
- TCC concepts.
- Mach messages.

Labs:

- pwn.college.
- OpenSecurityTraining2.
- Linux Kernel Labs: https://linux-kernel-labs.github.io/
- Build and debug a Linux kernel in a VM.
- Learn WinDbg.
- Learn LLDB for macOS.

Reading:

- Linux Kernel docs: https://docs.kernel.org/
- Microsoft Windows Internals book series.
- Microsoft security research: https://msrc.microsoft.com/blog/
- Apple Platform Security: https://support.apple.com/guide/security/welcome/web
- Project Zero: https://projectzero.google/

Research tasks:

- Study historical LPE CVEs.
- Learn kernel crash triage.
- Fuzz syscalls or kernel interfaces in local VMs.
- Patch-diff security updates.
- Learn exploit reliability under mitigations.

Submission-ready bar:

- Non-admin to elevated code execution through kernel vulnerability.
- Must work reliably on the contest target configuration.

## Category 10: Virtualization

Berlin 2026 targets:

- KVM
- VMware ESXi
- Microsoft Hyper-V Client

Contest expectation:

- Launch from guest OS.
- Execute arbitrary code on host OS or hypervisor.
- Cross-tenant guest-to-guest code execution may be an add-on for some targets.

This is one of the hardest categories.

Core skills:

- CPU virtualization.
- Hypervisor architecture.
- Device emulation.
- VirtIO.
- QEMU/KVM.
- VMware internals at a conceptual level.
- Hyper-V architecture.
- Guest/host memory isolation.
- VM exits.
- Emulated devices.
- Paravirtual drivers.
- Kernel debugging.
- Fuzzing device models.

Attack surfaces:

- Virtual devices.
- Network adapters.
- Storage controllers.
- Clipboard/shared folders if in scope.
- Guest additions/tools if in scope.
- VM configuration parsing.
- Hypercalls.
- VirtIO devices.
- Emulated USB.
- Graphics devices, if in scope.
- Snapshot and migration features.

Practice targets:

- QEMU: https://www.qemu.org/
- KVM docs: https://www.linux-kvm.org/page/Documents
- Firecracker for microVM concepts: https://github.com/firecracker-microvm/firecracker
- VirtualBox source for learning: https://www.virtualbox.org/

Labs:

- Learn QEMU device models.
- Write a tiny emulated device or study a simple one.
- Fuzz QEMU device inputs locally.
- Study VirtIO.
- Study guest-to-host escape writeups.
- Build a KVM lab with snapshots.

Reading:

- QEMU docs: https://www.qemu.org/docs/master/
- Linux KVM docs: https://www.kernel.org/doc/html/latest/virt/kvm/index.html
- Project Zero virtualization posts: https://projectzero.google/
- ZDI virtualization advisories: https://www.zerodayinitiative.com/advisories/published/

Research tasks:

- Study historical VM escape bugs.
- Learn device emulation code.
- Fuzz selected device models.
- Learn hypervisor debugging.
- Understand out-of-scope components from rules before investing time.

Submission-ready bar:

- Guest-to-host or guest-to-hypervisor code execution.
- Reliability is difficult and critical.

## Cross-Category Skills To Build

### Programming

- Python: automation, fuzzing, API testing, PoCs.
- JavaScript/TypeScript: web, agents, frontend, Node apps.
- C/C++: browsers, parsers, native tools, kernels.
- Go: containers, local inference, cloud tooling.
- Rust basics: modern systems tools.

### Debugging

- gdb.
- lldb.
- WinDbg.
- pwndbg.
- rr.
- sanitizers.
- logs and tracing.

### Fuzzing

- AFL++.
- libFuzzer.
- honggfuzz.
- boofuzz for protocols.
- grammar-based fuzzing.
- corpus minimization.
- crash deduplication.

### Code Review

- ripgrep.
- Semgrep.
- CodeQL.
- git diff.
- patch diffing.
- call graph reading.

### Reporting

- Root cause clarity.
- Exact affected version.
- Trigger path.
- Security boundary crossed.
- Reliability notes.
- Crash artifacts.
- Mitigation suggestions.

## 12-Month Preparation Plan

### Months 1-2: Choose Track And Build Foundation

Pick one primary track:

- AI/Web track: Coding Agent, Local Inference, AI Database, Server.
- Systems track: Container, NVIDIA, LPE, Virtualization.
- Browser track: Web Browser, Enterprise Apps.

For your background, choose AI/Web first.

Tasks:

- Finish API, OAuth, GraphQL, race, and logic labs.
- Learn Python and TypeScript code review.
- Build local labs for LiteLLM, Chroma, Ollama, and a toy agent.

### Months 3-4: Code Review And Fuzzing

Tasks:

- Pick one open-source target.
- Map attack surface.
- Write tests.
- Add fuzzing for one parser/API.
- Read historical advisories.
- Practice writing whitepapers.

### Months 5-6: Target Deep Dive

Tasks:

- Choose one likely Pwn2Own-style target.
- Track releases and commits.
- Build repeatable local environment.
- Create a bug-class matrix.
- Run focused fuzzing and manual review.

### Months 7-8: Chain Building

Tasks:

- Turn bugs into reliable impact.
- Identify needed primitives.
- Work on reliability.
- Document exact preconditions.
- Start a second backup target.

### Months 9-10: Contest Simulation

Tasks:

- Rebuild target from scratch.
- Run exploit/demo repeatedly.
- Write full whitepaper.
- Practice a timed demonstration.
- Remove fragile assumptions.

### Months 11-12: Registration Readiness

Tasks:

- Re-check current rules.
- Confirm target versions and configuration.
- Prepare entry documentation.
- Prepare fallback.
- Register with ZDI if eligible and ready.

## Recommended Track For You

Primary track:

1. Coding Agent
2. Local Inference
3. AI Database
4. Server

Secondary track:

1. Container
2. NVIDIA

Long-term track:

1. Enterprise Applications
2. Web Browser
3. Local Escalation of Privilege
4. Virtualization

This gives you the highest overlap with your current strengths while building toward real Pwn2Own-grade research.

## Weekly Schedule

Monday:

- Read code and map attack surface.

Tuesday:

- Build tests and reproduce suspicious behavior.

Wednesday:

- Fuzz one input surface.

Thursday:

- Read advisories and patch diffs.

Friday:

- Try to turn one abnormal behavior into real impact.

Saturday:

- Write notes, diagrams, and report sections.

Sunday:

- Review progress and choose next high-value hypothesis.

## Final Rule

Do not spread across all ten categories at once.

Pick one main category, one backup category, and one long-term category.

Best choice for you:

- Main: Coding Agent
- Backup: Local Inference
- Long-term: Server or Container

The fastest realistic path from web bug bounty to Pwn2Own is not browser or hypervisor first. It is AI-agent and AI-infrastructure targets where web, API, filesystem, auth, sandbox, and tool-permission bugs meet.

## Beginner Glossary: Bug Types Mentioned In The Rules

If you know web bug bounty but not Pwn2Own-style exploitation, this section explains the important words from the rules and how to start learning each one.

### Arbitrary Code Execution

Meaning:

- The attacker makes the target run attacker-controlled instructions.
- In web bug bounty language, this is similar to RCE, but Pwn2Own usually requires stronger proof and reliability.

Where it appears:

- Every category. The rules say all entries must compromise the target and demonstrate arbitrary code execution.

Examples of root causes:

- Memory corruption.
- Command injection.
- Unsafe deserialization.
- Unsafe plugin/tool execution.
- File parser bug.
- Template injection leading to code execution.
- Sandbox permission bypass leading to shell execution.

How to start learning:

- Web path: command injection, SSTI, deserialization, file upload to execution.
- Code path: Python/Node subprocess bugs, unsafe eval, unsafe YAML/pickle.
- Native path: buffer overflow, use-after-free, type confusion.

Labs:

- PortSwigger OS command injection: https://portswigger.net/web-security/os-command-injection
- PortSwigger server-side template injection: https://portswigger.net/web-security/server-side-template-injection
- PortSwigger deserialization: https://portswigger.net/web-security/deserialization
- pwn.college: https://pwn.college/
- OpenSecurityTraining2: https://ost2.fyi/

### Sandbox Escape

Meaning:

- A sandbox limits what compromised code can access.
- A sandbox escape means code breaks out of that restricted environment and reaches more powerful access.

Where it appears:

- Web Browser.
- Adobe Reader.
- Coding Agent.
- Sometimes Enterprise Apps and browsers require this for higher prizes.

Real examples:

- Browser renderer process is compromised, then exploit escapes to browser/OS level.
- PDF Reader Protected Mode is bypassed.
- Coding agent default sandbox is bypassed and performs actions outside allowed permissions.

How to start learning:

- First understand what a sandbox is.
- Then study browser process models, OS permissions, and permission prompts.
- For your background, start with coding-agent sandboxes before browser sandboxes.

Labs and reading:

- Chromium sandbox docs: https://chromium.googlesource.com/chromium/src/+/main/docs/design/sandbox.md
- Project Zero blog: https://projectzero.google/
- OWASP LLM Top 10: https://owasp.org/www-project-top-10-for-large-language-model-applications/
- PortSwigger Web LLM attacks: https://portswigger.net/web-security/learning-paths/llm-attacks

Beginner practice:

- Build a toy app that lets users run limited commands.
- Add allow/deny rules.
- Try to find ways commands can escape those rules.
- Build a toy agent that can read files only inside one folder.
- Test symlinks, relative paths, hidden files, generated files, and tool output.

### Kernel Escalation Of Privilege

Meaning:

- You start as a normal user.
- You exploit a kernel bug to become root/admin or execute code in kernel/elevated context.

Where it appears:

- Local Escalation of Privilege category.
- Browser category as an add-on path.
- Enterprise apps may allow kernel escalation as an escape option.

Root causes:

- Use-after-free.
- Race condition.
- Integer overflow.
- Out-of-bounds access.
- Bad permission checks in kernel APIs or drivers.

How to start learning:

- Learn C basics.
- Learn OS internals.
- Learn memory safety.
- Learn debugging.
- Start with Linux before Windows/macOS if you are new.

Labs and reading:

- Linux Kernel Labs: https://linux-kernel-labs.github.io/
- Linux kernel docs: https://docs.kernel.org/
- pwn.college: https://pwn.college/
- OpenSecurityTraining2: https://ost2.fyi/
- Project Zero: https://projectzero.google/

Beginner practice:

- Learn Linux users, groups, permissions, and capabilities.
- Build a vulnerable local C program and exploit it.
- Learn gdb.
- Then move to kernel labs.

### Guest-To-Host Escape

Meaning:

- You start inside a virtual machine, container, or microVM.
- You exploit the isolation layer and execute code on the host system.

Where it appears:

- Virtualization.
- Container.
- Firecracker.
- NVIDIA Container Toolkit.

Root causes:

- Bug in virtual device emulation.
- Bug in container runtime.
- Unsafe mounts.
- Runtime API flaw.
- Kernel bug reachable from isolated environment.
- Bad filesystem boundary handling.

How to start learning:

- Learn Linux containers first.
- Then learn virtual machines.
- Then learn hypervisors.

Labs and reading:

- Docker security docs: https://docs.docker.com/engine/security/
- containerd docs: https://containerd.io/docs/
- OCI runtime spec: https://github.com/opencontainers/runtime-spec
- Firecracker docs: https://github.com/firecracker-microvm/firecracker/tree/main/docs
- QEMU docs: https://www.qemu.org/docs/master/
- Linux namespaces: https://man7.org/linux/man-pages/man7/namespaces.7.html
- Linux capabilities: https://man7.org/linux/man-pages/man7/capabilities.7.html

Beginner practice:

- Learn Docker from normal user perspective.
- Learn what `--privileged`, mounts, capabilities, and seccomp do.
- Run containers with reduced permissions and observe what breaks.
- Study old container escape CVEs as patch-diff exercises.

### Cross-Tenant Code Execution

Meaning:

- One tenant/user/VM affects another tenant/user/VM.
- In the virtualization add-on, code execution in one VM must also result in code execution in a separate VM managed by the same virtualization target.

Where it appears:

- Virtualization add-on.
- The concept also matters in SaaS, AI databases, and coding agents.

How this maps to your web background:

- This is like cross-tenant IDOR, but with code execution instead of data access.
- Your IDOR and multi-tenant thinking helps here.

How to start learning:

- Practice multi-tenant SaaS isolation first.
- Then study VM/container isolation.
- Then study shared services and shared resources.

Labs:

- Build a toy multi-tenant app.
- Build two isolated workspaces.
- Test whether one workspace can affect another.
- Build a toy RAG app with two tenants and test cross-tenant retrieval.

### Authentication Bypass

Meaning:

- The rules say if authentication exists, the exploit must happen before authentication or include an authentication bypass.
- In simple terms: if the target requires login, you either exploit it without logging in or prove you can bypass the login/security check.

Where it appears:

- Server category.
- Web/API services.
- AI database services.
- Local inference APIs if auth is present.

Root causes:

- Missing auth check.
- Token confusion.
- JWT validation bug.
- SSO/OAuth flaw.
- Default credentials.
- API route not protected.
- Pre-auth parser bug.

How to start learning:

- PortSwigger authentication labs: https://portswigger.net/web-security/authentication
- PortSwigger OAuth labs: https://portswigger.net/web-security/oauth
- PortSwigger JWT labs: https://portswigger.net/web-security/jwt
- OWASP API Security Top 10: https://owasp.org/API-Security/

Beginner practice:

- Test apps with two users.
- Compare authenticated and unauthenticated routes.
- Test old API versions.
- Test mobile APIs.
- Test password reset, invite, SSO, and API key flows.

### Data Exfiltration

Meaning:

- Stealing or extracting sensitive data from the target environment.
- In the Microsoft 365 Copilot add-on, examples include email, calendar, contacts, SharePoint/OneDrive docs, Teams messages, or meeting transcripts.

Where it appears:

- Enterprise Applications Copilot add-on.
- Coding agents.
- AI databases.
- Server apps.

Important:

- In Pwn2Own, this usually happens after a qualifying compromise.
- A plain prompt leak is usually not enough if it does not cross a real security boundary.

How to start learning:

- Learn Microsoft Graph basics.
- Learn OAuth scopes.
- Learn RAG and semantic index concepts.
- Learn tenant isolation.

Reading:

- Microsoft Graph docs: https://learn.microsoft.com/en-us/graph/
- Microsoft 365 Copilot extensibility: https://learn.microsoft.com/en-us/microsoft-365-copilot/extensibility/
- OWASP LLM Top 10: https://owasp.org/www-project-top-10-for-large-language-model-applications/

Beginner practice:

- Build a toy app with private documents and an AI search feature.
- Create two users and two document sets.
- Test whether one user can retrieve another user's documents.
- Add tool actions and test scope enforcement.

### Copilot Or Agent Action Execution

Meaning:

- The exploit causes an AI assistant or coding agent to perform actions the user did not authorize.
- For the Copilot add-on, the rules require at least two unauthorized actions on behalf of the victim.

Where it appears:

- Enterprise Applications Copilot add-on.
- Coding Agent category.

Examples:

- Sending email.
- Modifying documents.
- Scheduling meetings.
- Invoking connectors.
- Running commands outside sandbox.
- Editing files outside allowed workspace.

How to start learning:

- Learn LLM tool calling.
- Learn OAuth scopes and connector permissions.
- Learn agent permission prompts.
- Learn how apps separate "suggestion" from "action".

Labs:

- Build a toy assistant that can call tools.
- Add `send_email`, `write_file`, and `run_command` mock tools.
- Add permissions.
- Test whether prompt injection or repo content can trigger unauthorized tools.

Reading:

- OWASP LLM Top 10: https://owasp.org/www-project-top-10-for-large-language-model-applications/
- PortSwigger Web LLM attacks: https://portswigger.net/web-security/learning-paths/llm-attacks
- Model Context Protocol docs: https://modelcontextprotocol.io/

### Prompt Injection That Crosses A Security Boundary

Meaning:

- Prompt injection alone is often not enough.
- It becomes meaningful when it causes unauthorized data access, tool execution, sandbox escape, or permission bypass.

Where it appears:

- Coding Agent.
- Enterprise Copilot add-ons.
- AI Database/RAG systems.
- Local inference products with tools/plugins.

How to start learning:

- Learn indirect prompt injection.
- Learn tool calling.
- Learn RAG.
- Learn permission boundaries.

Practice:

- Put malicious instructions in a document in your own local RAG app.
- See if the assistant follows document instructions over user/system policy.
- Then add real security boundaries and test whether they hold.

### Native Tool Vulnerability

Meaning:

- Coding agents rely on local tools such as Git, package managers, compilers, shells, interpreters, linters, and test runners.
- The rules say native tooling relied on by the coding agent may be in scope depending on registration decisions.

Where it appears:

- Coding Agent category.

Examples:

- Parser bug in a tool the agent invokes.
- Package manager lifecycle behavior.
- Unsafe project config handling.
- Shell command construction issue.
- Git/submodule/path behavior.

How to start learning:

- Learn Git internals.
- Learn npm/pip package behavior.
- Learn how test runners and build systems execute code.
- Learn shell quoting and command construction.

Reading:

- Git docs: https://git-scm.com/docs
- npm scripts docs: https://docs.npmjs.com/cli/v10/using-npm/scripts
- Python packaging docs: https://packaging.python.org/
- Node.js security best practices: https://nodejs.org/en/learn/getting-started/security-best-practices

### Memory Corruption

Meaning:

- A program incorrectly handles memory, allowing crashes or attacker-controlled behavior.
- This is a major path for browsers, PDF readers, Office apps, kernels, hypervisors, and parsers.

Common types:

- Buffer overflow.
- Use-after-free.
- Double free.
- Out-of-bounds read/write.
- Type confusion.
- Integer overflow.
- Null pointer dereference.

Where it appears:

- Web Browser.
- Enterprise Applications.
- Local Escalation of Privilege.
- Virtualization.
- Container.
- Native local inference such as llama.cpp.

How to start learning:

- Learn C.
- Learn memory layout.
- Learn gdb/lldb.
- Learn exploit mitigations.
- Learn fuzzing.

Labs:

- pwn.college: https://pwn.college/
- OpenSecurityTraining2: https://ost2.fyi/
- RPISEC MBE: https://github.com/RPISEC/MBE
- The Fuzzing Book: https://www.fuzzingbook.org/
- AFL++: https://aflplus.plus/docs/

### File Parser Vulnerability

Meaning:

- A target parses a complex file and mishandles malformed content.
- This may cause memory corruption, path traversal, deserialization, command execution, or logic bugs.

Where it appears:

- Adobe Reader.
- Microsoft Office.
- Browsers.
- Local inference model files.
- AI databases import/export.
- Coding agents reading repositories and media files.

File types to study:

- PDF.
- DOCX/XLSX/PPTX.
- ZIP/TAR.
- JSON/YAML/TOML.
- Markdown.
- Images.
- Model files and configs.

How to start learning:

- Start with ZIP/TAR, JSON, YAML, and Markdown.
- Then move to PDF and Office formats.
- Then move to native binary parsers.

Labs:

- Write a small parser and fuzz it.
- Fuzz open-source parsers first.
- Learn corpus minimization.

### Deserialization

Meaning:

- The app loads structured data back into objects.
- If untrusted data controls that process, it may cause code execution or object injection.

Where it appears:

- Web apps.
- Server apps.
- AI/ML tools.
- Enterprise apps.
- Config and model loading.

Important NVIDIA rule detail:

- For Megatron Bridge, pickle deserialization and `trust_remote_code=true` issues were out of scope in Berlin 2026.
- This means you must always check what the rules exclude.

How to start learning:

- PortSwigger deserialization: https://portswigger.net/web-security/deserialization
- OWASP deserialization cheat sheet: https://cheatsheetseries.owasp.org/cheatsheets/Deserialization_Cheat_Sheet.html
- Python pickle docs: https://docs.python.org/3/library/pickle.html

Practice:

- Learn why pickle is unsafe with untrusted data.
- Learn safer formats like JSON.
- Review apps for `pickle`, `yaml.load`, Java serialization, .NET serializers, and custom object loaders.

### SSRF

Meaning:

- Server-Side Request Forgery makes the target server send network requests chosen by the attacker.

Where it appears:

- Server apps.
- AI tools that fetch URLs.
- Webhook systems.
- Local inference integrations.
- PDF/image/URL preview features.

How to start learning:

- PortSwigger SSRF labs: https://portswigger.net/web-security/ssrf
- OWASP SSRF prevention: https://cheatsheetseries.owasp.org/cheatsheets/Server_Side_Request_Forgery_Prevention_Cheat_Sheet.html

Practice:

- Test URL preview features.
- Test webhook targets.
- Test import-from-URL.
- Test AI tools that browse or fetch resources.
- Use only authorized labs and targets.

### Request Smuggling And HTTP Parser Confusion

Meaning:

- Different servers in a chain interpret the same HTTP request differently.
- This can let one request hide inside another or bypass controls.

Where it appears:

- Server category.
- Web apps behind proxies/CDNs.
- Enterprise products.

How to start learning:

- PortSwigger request smuggling: https://portswigger.net/web-security/request-smuggling
- PortSwigger HTTP/2 research: https://portswigger.net/research

Practice:

- Use labs first.
- Do not test aggressively on production without explicit permission because it can break traffic.

### Side-Channel Attack

Meaning:

- Leaking information through indirect signals like timing, cache behavior, power, or CPU behavior.
- The rules mention known hardware-level CPU side channels may be treated as previously known and scored lower.

Where it appears:

- Browsers.
- Virtualization.
- Kernel.
- Hardware/CPU contexts.

How to start learning:

- Learn timing attacks first.
- Learn cache side-channel concepts later.

Reading:

- Project Zero side-channel posts: https://projectzero.google/
- Spectre paper: https://spectreattack.com/
- Meltdown paper: https://meltdownattack.com/

Beginner practice:

- Write a toy timing leak locally.
- Learn constant-time comparison concepts.

### Man-In-The-Middle, ARP Spoofing, And Downgrade Attacks

Meaning:

- MITM means placing yourself between two communicating systems.
- ARP spoofing is a local network MITM technique.
- Downgrade attacks force weaker versions/protocols.

Where it appears:

- Rules say these may be accepted at reduced prize at Sponsor discretion.

How to start learning:

- Learn TLS basics.
- Learn certificate validation.
- Learn local network fundamentals.
- Learn protocol downgrade concepts.

Reading:

- Mozilla TLS guide: https://wiki.mozilla.org/Security/Server_Side_TLS
- OWASP Transport Layer Security Cheat Sheet: https://cheatsheetseries.owasp.org/cheatsheets/Transport_Layer_Security_Cheat_Sheet.html

Beginner practice:

- Use local labs only.
- Learn Wireshark.
- Learn how HTTPS prevents MITM when configured correctly.

## From Zero Knowledge To First Pwn2Own Track

If you know nothing about these categories, do not try all ten at once.

Use this path:

### Phase 1: Understand The Rule Language

Learn these first:

- RCE / arbitrary code execution.
- Sandbox.
- Privilege escalation.
- Authentication bypass.
- Tenant boundary.
- Container escape.
- VM escape.
- Data exfiltration.
- Permission boundary.
- Whitepaper.

### Phase 2: Build Web-To-Pwn2Own Bridge Skills

Start with:

- Python.
- JavaScript/TypeScript.
- HTTP/API testing.
- Linux basics.
- Docker basics.
- Git basics.
- File paths and filesystem permissions.
- Basic C.

### Phase 3: Pick One Beginner-Friendly Category

Best choice:

- Coding Agent.

Backup:

- Local Inference.

Third:

- AI Database.

Do not start with:

- Virtualization.
- Browser.
- Kernel LPE.

Those are long-term tracks.

### Phase 4: Build Three Local Labs

Lab 1: Coding Agent Permission Lab

- Build or use a toy agent.
- Give it file read/write tools.
- Give it a shell tool.
- Add permission checks.
- Test if repo content can trick it into crossing the boundary.

Lab 2: Local Inference API Lab

- Run LiteLLM or Ollama locally.
- Map API routes.
- Test auth, config, path, model loading, and SSRF-like features.

Lab 3: AI Database/RAG Lab

- Build a small RAG app.
- Add two users and two tenants.
- Add private documents.
- Test if one tenant can retrieve another tenant's data.
- Then look for ways that ingestion or parsing could become code execution in a controlled lab.

### Phase 5: Learn Fuzzing

Start with simple formats:

- JSON.
- YAML.
- Markdown.
- ZIP/TAR.
- URLs.

Then move to:

- PDF.
- Office files.
- Model files.
- Native parsers.

Resources:

- The Fuzzing Book: https://www.fuzzingbook.org/
- AFL++: https://aflplus.plus/docs/
- libFuzzer: https://llvm.org/docs/LibFuzzer.html

### Phase 6: Learn To Write A Whitepaper

Pwn2Own requires a Markdown whitepaper with:

- Preconditions.
- Exploit steps.
- Explanation of exploit techniques.
- Version information.
- CWE for each bug.
- Code listings with line numbers.
- Your comments explaining the vulnerability mechanism.

Practice by writing whitepapers for lab bugs even when they are not real zero-days.

## Beginner 6-Month Plan

### Month 1: Basics

- Learn Linux command line.
- Learn Python basics.
- Learn JavaScript basics.
- Learn Git.
- Learn Docker basics.
- Finish PortSwigger command injection, deserialization, SSRF, file upload, and access control labs.

### Month 2: API And Agent Security

- Learn HTTP deeply.
- Complete PortSwigger API testing labs.
- Complete PortSwigger Web LLM attack labs.
- Build a toy coding agent with tools.
- Build a toy RAG app.

### Month 3: Local Inference And AI Database

- Install LiteLLM, Ollama, Chroma, and pgvector locally.
- Map endpoints and configs.
- Read source code with Codex/Claude help.
- Write tests for auth, path handling, and config loading.

### Month 4: Fuzzing And Parser Bugs

- Learn fuzzing basics.
- Fuzz JSON/YAML/Markdown inputs.
- Fuzz one local HTTP endpoint.
- Learn crash reproduction and minimization.

### Month 5: Code Review And Patch Diffing

- Learn Semgrep basics.
- Learn CodeQL basics.
- Read ZDI advisories.
- Read Project Zero writeups.
- Practice patch diffing open-source projects.

### Month 6: One Target Deep Dive

- Pick one target: LiteLLM, Chroma, Ollama, or a coding-agent tool.
- Spend the whole month on it.
- Build attack-surface map.
- Build tests.
- Fuzz one component.
- Write a mock Pwn2Own whitepaper.

## Safe AI Prompts For Learning These Bug Types

Use Codex/Claude like this:

```text
Explain this Pwn2Own rule in beginner language. Define every technical term and give a legal local-lab example.
```

```text
Map this open-source target's attack surface: API routes, file parsers, auth checks, subprocess calls, filesystem access, deserialization, model loading, tool execution, and sandbox boundaries.
```

```text
Teach me this bug class from zero: what it is, where it appears, what code patterns cause it, what labs to do, and how to recognize it safely.
```

```text
Generate local-only tests for this function. Focus on path traversal, malformed JSON, unsafe config, auth bypass, oversized input, symlinks, and permission boundary checks.
```

```text
Turn this lab finding into a Pwn2Own-style whitepaper with preconditions, affected version, root cause, CWE, code listing placeholders, and reproduction steps.
```

## Important Mindset

Bug bounty often asks:

```text
Can I access data or perform an unauthorized action?
```

Pwn2Own asks:

```text
Can I make the target execute code or cross a protected security boundary reliably, on the exact version, under contest rules, in under 10 minutes?
```

That is the gap you must train for.
