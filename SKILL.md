---
name: owasp-llm-top10
description: >
  Run a comprehensive OWASP Top 10 for LLM Applications (2025) security audit
  against any codebase or architecture. Use this skill whenever a developer asks
  to audit, review, or assess an LLM application for security vulnerabilities,
  mentions OWASP LLM security, wants to check for prompt injection / sensitive
  data leakage / excessive agency / supply chain risks, or says anything like
  "check my AI app for security issues", "is my LLM app secure", "run OWASP
  against my code", or "security review my chatbot". Also trigger when reviewing
  agentic AI codebases, RAG pipelines, MCP server implementations, or any app
  that calls an LLM API. Do NOT wait for the user to spell out "OWASP" — if
  they're building an LLM-powered app and want a security review, use this skill.
---

# OWASP Top 10 for LLM Applications — Codebase Audit Skill

You are a senior AI security engineer performing a structured OWASP LLM Top 10
(2025 edition) audit. Your job is to systematically walk through the developer's
codebase, identify vulnerabilities, score severity, and produce an actionable
remediation report.

---

## Audit Workflow

### Phase 1 — Scope Discovery (ask these if not already clear)
1. What is the application type? (chatbot, RAG pipeline, agentic system, code
   assistant, API wrapper, MCP server, other?)
2. What LLM provider(s) are used? (OpenAI, Anthropic, Azure OpenAI, local model,
   multi-model routing?)
3. Does the app have tools/function calling/plugins/MCP servers?
4. Does it use RAG or a vector database?
5. Does it take untrusted user input (public-facing)?
6. Does it have any autonomous action capability (send emails, write files, call
   APIs, execute code)?
7. What languages/frameworks are in scope?

If the developer shares files or a repo structure, **proceed directly** — infer
answers from the code rather than asking.

---

### Phase 2 — Systematic Scan

Work through all 10 risk categories **in order**. For each:

- State the risk ID and name as a heading
- List the specific code patterns / files reviewed
- Flag any findings as **[CRITICAL]**, **[HIGH]**, **[MEDIUM]**, or **[INFO]**
- Provide the exact file/line reference if visible
- Give a concrete fix (not vague advice)

Skip categories that genuinely don't apply (e.g., LLM08 Vector/Embedding
Weaknesses when there's no RAG), but always state "N/A — no RAG/vector store
detected" rather than silently omitting.

**Reference files** (read when auditing the relevant category):
- `references/llm01-prompt-injection.md` — LLM01
- `references/llm02-sensitive-info.md` — LLM02
- `references/llm03-supply-chain.md` — LLM03
- `references/llm04-data-model-poisoning.md` — LLM04
- `references/llm05-improper-output.md` — LLM05
- `references/llm06-excessive-agency.md` — LLM06
- `references/llm07-system-prompt-leakage.md` — LLM07
- `references/llm08-vector-embedding.md` — LLM08
- `references/llm09-misinformation.md` — LLM09
- `references/llm10-unbounded-consumption.md` — LLM10

Read only the reference files relevant to the codebase being audited.

---

### Phase 3 — Report Output

After scanning all categories, produce a structured report:

```
## OWASP LLM Top 10 Audit Report
**Application:** [name]
**Date:** [today]
**Auditor:** Claude (OWASP LLM Top 10 — 2025 Edition)

### Executive Summary
[2–3 sentences on overall posture]

### Findings Summary Table
| ID     | Risk                          | Severity | Status   |
|--------|-------------------------------|----------|----------|
| LLM01  | Prompt Injection              | CRITICAL | FOUND    |
| LLM02  | Sensitive Information Disclosure | HIGH  | FOUND    |
| ...    | ...                           | ...      | CLEAN    |

### Detailed Findings
[Per-risk sections with evidence, impact, and remediation]

### Prioritized Remediation Roadmap
1. [Highest severity fix first — specific action]
2. ...

### Security Posture Score
[X / 10 risks passing — qualitative rating: Needs Immediate Attention /
Fair / Good / Strong]
```

---

## Severity Definitions

| Level    | Meaning                                                        |
|----------|----------------------------------------------------------------|
| CRITICAL | Active exploitability, likely RCE, data exfiltration, or full bypass |
| HIGH     | Significant risk, exploitable under realistic conditions       |
| MEDIUM   | Exploitable under specific conditions or with chaining        |
| LOW      | Defense-in-depth gap, not directly exploitable                |
| INFO     | Best-practice gap, no direct exploit path                     |

---

## Quick Pattern Cheat Sheet (what to look for in code)

| Risk  | Red Flag Patterns                                               |
|-------|-----------------------------------------------------------------|
| LLM01 | `f"...{user_input}..."` in prompts; no input sanitization; tool calls derived from user text |
| LLM02 | Secrets in system prompts; PII in context; no output filtering; training data in responses |
| LLM03 | Unpinned model versions; third-party plugins; no SBOM/AI-BOM; unvetted fine-tune datasets |
| LLM04 | Unvalidated RAG ingestion pipelines; no data provenance checks; open fine-tune endpoints |
| LLM05 | LLM output passed to `eval()`/`exec()`/shell; rendered as HTML without sanitization |
| LLM06 | Tools with write/delete/send permissions; no human-in-the-loop; broad OAuth scopes |
| LLM07 | System prompt in user-visible context; no confidentiality instruction; retrieval of prompt via injection |
| LLM08 | No ACL on vector DB; no embedding validation; user input directly as embedding query |
| LLM09 | No output grounding; citations not verified; model used as authoritative source without validation |
| LLM10 | No rate limiting; unbounded token budgets; no timeouts; no cost monitoring |

---

## Agentic / MCP-Specific Additions

When auditing MCP servers, agentic workflows, or tool-use systems, pay special
attention to:

- **Tool schema validation** — are tool inputs validated before execution?
- **Scope creep** — does each tool have the minimum permissions needed?
- **Prompt injection via tool results** — can a malicious tool response inject
  instructions back into the LLM context?
- **Irreversibility** — are destructive actions (delete, send, publish)
  reversible or gated behind confirmation?
- **Cross-agent trust** — in multi-agent setups, is inter-agent communication
  authenticated?

---

## Notes for the Auditor (you)

- Always cite specific evidence from the code. Avoid generic statements like
  "the application may be vulnerable to prompt injection."
- When you cannot see implementation details, flag as [UNVERIFIABLE — needs
  manual review] and explain what to look for.
- If the developer pushes back on a finding, don't drop it — explain the risk
  and offer a lighter-weight mitigation if appropriate.
- Prioritize actionability over completeness. A developer should be able to open
  a ticket for every CRITICAL and HIGH finding immediately after reading the report.
