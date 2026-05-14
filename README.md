# owasp-llm-top10

> A Claude skill that performs structured, evidence-based **OWASP Top 10 for LLM Applications (2025)** security audits against any AI-powered codebase or architecture.

---

## Overview

This skill turns Claude into a senior AI security engineer. It systematically walks through all 10 OWASP LLM risk categories, identifies vulnerabilities with specific code evidence, scores severity, and produces a structured remediation report — complete with an executive summary, findings table, and prioritized action roadmap.

It works on any LLM-powered application: chatbots, RAG pipelines, agentic systems, MCP servers, API wrappers, code assistants, or multi-model orchestrators.

---

## Repo Structure

```
owasp-llm-top10/
├── SKILL.md                              # Skill definition, audit workflow, cheat sheet
└── references/
    ├── llm01-prompt-injection.md
    ├── llm02-sensitive-info.md
    ├── llm03-supply-chain.md
    ├── llm04-data-model-poisoning.md
    ├── llm05-improper-output.md
    ├── llm06-excessive-agency.md
    ├── llm07-system-prompt-leakage.md
    ├── llm08-vector-embedding.md
    ├── llm09-misinformation.md
    └── llm10-unbounded-consumption.md
```

`SKILL.md` is the entry point. The `references/` files are loaded per category during an audit — each one contains attack scenarios, red-flag code patterns, mitigations, and a per-category audit checklist.

---

## How to Trigger It

The skill activates on both explicit and intent-based requests. You don't need to say "OWASP."

| What you say | What happens |
|---|---|
| `"Run an OWASP audit on my app"` | Full structured audit |
| `"Is my LLM app secure?"` | Full audit + report |
| `"Security review my chatbot"` | Full audit |
| `"Check my MCP server for vulnerabilities"` | Agentic/MCP-focused audit |
| `"Review my RAG pipeline"` | Audit with LLM08 emphasis |
| `"Check my AI app for security issues"` | Full audit |
| `"Run OWASP against my code"` | Full audit |
| Sharing a codebase or repo structure | Audit inferred from code — no preamble needed |

---

## Audit Workflow

### Phase 1 — Scope Discovery

If the application context isn't clear from shared code, the skill asks seven scoping questions:

1. Application type (chatbot / RAG / agentic / API wrapper / MCP server / other)
2. LLM provider(s) used
3. Whether tools, function calling, plugins, or MCP servers are present
4. Whether a vector database or RAG pipeline is used
5. Whether the app is public-facing and takes untrusted user input
6. Whether the app has autonomous action capability (send emails, write files, call APIs, execute code)
7. Languages and frameworks in scope

When code is provided, Claude infers answers directly — no questions asked.

### Phase 2 — Systematic Scan

All 10 risk categories are reviewed in order. For each:

- The relevant `references/` file is loaded
- Specific files and code patterns in the target codebase are examined
- Findings are flagged as `[CRITICAL]`, `[HIGH]`, `[MEDIUM]`, `[LOW]`, or `[INFO]`
- Exact file/line references are cited where visible
- Concrete, actionable fixes are provided (not vague advice)
- Categories that genuinely don't apply are explicitly marked `N/A` with a reason

### Phase 3 — Structured Report

```
## OWASP LLM Top 10 Audit Report
Application: [name]
Date: [date]
Auditor: Claude (OWASP LLM Top 10 — 2025 Edition)

### Executive Summary
[2–3 sentences on overall posture]

### Findings Summary Table
| ID     | Risk                             | Severity | Status |
|--------|----------------------------------|----------|--------|
| LLM01  | Prompt Injection                 | CRITICAL | FOUND  |
| LLM02  | Sensitive Information Disclosure | HIGH     | FOUND  |
| LLM03  | Supply Chain                     | —        | CLEAN  |
| ...

### Detailed Findings
[Per-risk sections: evidence → impact → remediation]

### Prioritized Remediation Roadmap
1. [Highest severity — specific action]
2. ...

### Security Posture Score
[X / 10 risks passing — Needs Immediate Attention / Fair / Good / Strong]
```

---

## Risk Categories

### LLM01 — Prompt Injection
Attacker-controlled text causes the model to follow attacker instructions instead of the developer's. Includes both direct injection (via user input) and indirect injection (via retrieved documents, tool results, emails, or RAG content). No complete technical solution exists — defense must be architectural.

**Key red flags:** User input interpolated directly into prompts (`f"...{user_input}..."`); tool results re-injected without validation; no separation between instruction and data channels.

---

### LLM02 — Sensitive Information Disclosure
LLMs can leak PII, credentials, system architecture details, proprietary business logic, or training data — either through misconfiguration or active extraction attacks.

**Key red flags:** API keys or secrets in system prompts; PII passed into context without anonymization; multi-tenant RAG with no namespace isolation; raw LLM output returned without filtering; full prompts logged.

---

### LLM03 — Supply Chain
Vulnerabilities introduced via foundation model providers, fine-tuning datasets, third-party plugins, RAG data sources, inference APIs, SDKs, or agent integrations. Compromise at any layer can be nearly impossible to detect post-deployment.

**Key red flags:** Unpinned model versions (e.g., `"gpt-4"` instead of `"gpt-4-0125-preview"`); unpinned or hash-unverified dependencies; third-party MCP servers added without code review; no AI Bill of Materials (AI-BOM); unvetted fine-tuning datasets.

---

### LLM04 — Data and Model Poisoning
Manipulation of training data, fine-tuning datasets, or RAG knowledge bases to introduce backdoors, biases, or malicious behaviors that activate under attacker-controlled conditions. Happens pre-deployment — detection is significantly harder than runtime attacks.

**Key red flags:** RAG ingestion endpoints with no authentication or content validation; automated feedback-to-training loops without human review; no provenance tracking on ingested documents; open write access to knowledge base.

---

### LLM05 — Improper Output Handling
Failure to treat LLM output as untrusted before passing it to downstream systems. LLM output must be validated just like user input — it can contain XSS payloads, SQL injection, shell commands, or arbitrary code.

**Key red flags:** LLM output passed to `eval()` or `exec()`; output rendered as HTML without escaping; output used to construct SQL queries or shell commands; structured output not schema-validated before use.

---

### LLM06 — Excessive Agency
LLM-based systems granted more permissions, capabilities, or autonomy than necessary. Especially critical in agentic systems and MCP-based architectures where the LLM can autonomously invoke tools. Three dimensions: excessive permissions, excessive functionality, excessive autonomy.

**Key red flags:** Tools with write/delete/send capabilities when read-only would suffice; no human-in-the-loop for irreversible actions; overly broad OAuth scopes; no audit logging of tool invocations; agent can POST data to arbitrary external URLs.

---

### LLM07 — System Prompt Leakage
System prompts often contain proprietary business logic, security controls, API references, and sometimes credentials. Leaking them gives attackers a blueprint for crafting targeted injections or bypassing controls entirely.

**Key red flags:** No confidentiality instruction in the system prompt; system prompt stored client-side or returned in debug/error responses; credentials or internal URLs embedded in the prompt; prompt extraction attempts not monitored or rate-limited.

---

### LLM08 — Vector and Embedding Weaknesses
New to the 2025 list, reflecting widespread RAG adoption. Covers unauthorized data access via missing tenant isolation, embedding inversion attacks that can reconstruct source documents, semantic similarity attacks, and adversarial embedding injection.

**Key red flags:** Vector DB queries without tenant/user ID filters; unauthenticated or publicly accessible vector DB endpoints; source text stored alongside embeddings; retrieved documents injected into LLM context without validation; no similarity score threshold.

---

### LLM09 — Misinformation
LLMs generate plausible but false, outdated, or fabricated information — including hallucinated citations and false claims of expertise. Especially dangerous in legal, medical, financial, compliance, or safety-critical applications.

**Key red flags:** No disclaimers on high-stakes LLM output; LLM used as authoritative source for factual queries without RAG grounding; citations not verified against real sources; system prompt claims professional licensure the model doesn't have.

---

### LLM10 — Unbounded Consumption
Uncontrolled LLM resource usage enabling Denial of Service (DoS), Denial of Wallet (DoW via API cost inflation), model extraction through mass querying, or infrastructure resource exhaustion.

**Key red flags:** No rate limiting on LLM-facing endpoints; no input token length validation; `max_tokens` not set on LLM calls; agentic loops with no max iteration count or timeout; unauthenticated endpoints; no billing alerts or cost anomaly monitoring.

---

## Severity Definitions

| Level | Meaning |
|---|---|
| **CRITICAL** | Active exploitability — likely RCE, data exfiltration, or full control bypass |
| **HIGH** | Significant risk, exploitable under realistic conditions |
| **MEDIUM** | Exploitable under specific conditions or with attack chaining |
| **LOW** | Defense-in-depth gap, not directly exploitable |
| **INFO** | Best-practice gap with no direct exploit path |

---

## Agentic and MCP-Specific Coverage

When auditing agentic workflows or MCP servers, the skill applies additional checks beyond the standard OWASP categories:

- **Tool schema validation** — are tool inputs validated before execution, independent of the LLM?
- **Scope creep** — does each tool have the minimum permissions needed for its stated purpose?
- **Prompt injection via tool results** — can a malicious tool response inject instructions back into the LLM context?
- **Irreversibility gates** — are destructive actions (delete, send, publish, pay) gated behind human confirmation?
- **Cross-agent trust** — in multi-agent setups, is inter-agent communication authenticated?

---

## Auditor Principles

The skill follows strict standards for evidence quality:

- **No generic findings.** Every finding cites specific code evidence. "The application may be vulnerable to prompt injection" is not a finding.
- **Unverifiable findings are flagged.** When implementation details are hidden or missing, findings are marked `[UNVERIFIABLE — needs manual review]` with instructions on what to look for.
- **No dropped findings under pushback.** If a developer disputes a finding, the skill explains the risk and offers a lighter-weight mitigation — it does not simply remove the finding.
- **Prioritize actionability.** Every CRITICAL and HIGH finding should be immediately ticket-ready after reading the report.

---

## Quick Red-Flag Cheat Sheet

| Risk | Patterns to look for |
|---|---|
| LLM01 | `f"...{user_input}..."` in prompts; unsanitized tool results re-injected into context |
| LLM02 | Secrets in system prompts; PII in context; no output filtering; multi-tenant RAG without namespace isolation |
| LLM03 | Unpinned model versions; unpinned dependencies; third-party plugins without review; no AI-BOM |
| LLM04 | Open RAG write endpoints; automated feedback→training loops; no ingestion content validation |
| LLM05 | `eval()`/`exec()` on LLM output; HTML rendered without escaping; LLM output in SQL strings |
| LLM06 | Broad tool permissions; no human-in-the-loop for destructive actions; wide OAuth scopes |
| LLM07 | No confidentiality instruction; system prompt in client code; credentials in system prompt |
| LLM08 | Vector DB queries without tenant filter; unauthenticated DB; no similarity threshold |
| LLM09 | No disclaimers on high-stakes output; unverified citations; LLM used as authoritative source |
| LLM10 | No rate limiting; no `max_tokens`; no agent iteration limit; unauthenticated LLM endpoints |

---

## Standard

Based on the [OWASP Top 10 for Large Language Model Applications — 2025 Edition](https://owasp.org/www-project-top-10-for-large-language-model-applications/).
