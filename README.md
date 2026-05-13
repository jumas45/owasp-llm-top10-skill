# owasp-llm-top10

A Claude skill that runs a structured **OWASP Top 10 for LLM Applications (2025)** security audit against any AI-powered codebase or architecture.

---

## What It Does

Drop this skill into your Claude setup and it turns Claude into a senior AI security engineer. Point it at a codebase — a RAG pipeline, agentic workflow, MCP server, chatbot, or any app that calls an LLM — and it will:

- Walk through all 10 OWASP LLM risk categories systematically
- Flag findings as **CRITICAL / HIGH / MEDIUM / LOW / INFO**
- Cite specific files, functions, and line references
- Produce a structured audit report with an executive summary, findings table, and prioritized remediation roadmap
- Score your overall security posture

---

## Triggers

The skill activates on both explicit and intent-based requests:

| What you say | What happens |
|---|---|
| "Run an OWASP audit on my app" | Full structured audit |
| "Is my LLM app secure?" | Scoped audit + report |
| "Check my MCP server for security issues" | Agentic/MCP-specific audit |
| "Review my RAG pipeline" | Audit with LLM08 emphasis |
| "Security review my chatbot" | Full audit |

---

## Risk Categories Covered

| ID | Risk |
|---|---|
| LLM01 | Prompt Injection |
| LLM02 | Sensitive Information Disclosure |
| LLM03 | Supply Chain |
| LLM04 | Data & Model Poisoning |
| LLM05 | Improper Output Handling |
| LLM06 | Excessive Agency |
| LLM07 | System Prompt Leakage |
| LLM08 | Vector & Embedding Weaknesses |
| LLM09 | Misinformation |
| LLM10 | Unbounded Consumption |

Each category has a dedicated reference file in `/references/` with attack scenarios, red-flag code patterns, mitigations, and an audit checklist.

---

## Repo Structure

```
owasp-llm-top10/
├── SKILL.md               # Skill definition and audit workflow
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

---

## Agentic & MCP-Specific Coverage

Beyond the standard OWASP categories, the skill has dedicated checks for agentic architectures and MCP servers:

- Tool schema validation
- Minimum-permission scoping per tool
- Prompt injection via tool results
- Irreversibility gates on destructive actions (delete, send, publish)
- Cross-agent trust and authentication in multi-agent setups

---

## Sample Output

```
## OWASP LLM Top 10 Audit Report
Application: My API
Date: 2025-05-12
Auditor: Claude (OWASP LLM Top 10 — 2025 Edition)

### Executive Summary
The application presents two critical risks requiring immediate attention...

### Findings Summary Table
| ID     | Risk                             | Severity | Status |
|--------|----------------------------------|----------|--------|
| LLM01  | Prompt Injection                 | CRITICAL | FOUND  |
| LLM06  | Excessive Agency                 | HIGH     | FOUND  |
| LLM02  | Sensitive Information Disclosure | MEDIUM   | FOUND  |
| LLM10  | Unbounded Consumption            | LOW      | FOUND  |
| LLM03  | Supply Chain                     | —        | CLEAN  |
...

### Security Posture Score
6 / 10 risks passing — Fair
```

---

## Standard

Based on the [OWASP Top 10 for Large Language Model Applications — 2025 Edition](https://owasp.org/www-project-top-10-for-large-language-model-applications/).
