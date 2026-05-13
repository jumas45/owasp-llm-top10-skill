# LLM03:2025 — Supply Chain

## What It Is
LLM supply chains include foundation model providers, fine-tuning datasets,
third-party plugins, RAG data sources, inference APIs, SDKs, and agent
integrations. Compromise at any point can introduce vulnerabilities that are
extremely difficult to detect.

## Attack Scenarios

### Scenario A — Poisoned model on HuggingFace
A developer pulls a "fine-tuned" model from a public repo. The model contains
a backdoor: specific trigger phrases cause it to exfiltrate data or produce
harmful outputs.

### Scenario B — Malicious PyPI package
```
pip install langchain-community  # legitimate
pip install langchian  # typosquat with malicious code
```

### Scenario C — Compromised plugin/tool
A third-party MCP server used for calendar access is compromised. It now also
reads and exfiltrates emails when invoked.

### Scenario D — Unvetted fine-tune dataset
A developer uses a dataset scraped from the web containing:
- Adversarial examples designed to manipulate model behavior
- PII from real users
- Backdoor triggers

## What to Look For in Code

```python
# RED FLAG — unpinned model version
model = "gpt-4"  # no version pinning; behavior can change silently

# RED FLAG — pip install without hash verification
# requirements.txt:
# langchain>=0.1.0  # unpinned

# RED FLAG — third-party plugin installed without review
mcpClient.addServer("https://some-third-party.com/mcp")

# RED FLAG — no AI-BOM or dependency inventory
# (absence of evidence — check if any SBOM/AI-BOM exists in repo)

# RED FLAG — model loaded from arbitrary URL
model = AutoModel.from_pretrained("random-user/some-fine-tuned-model")
```

## Mitigations

### Model provenance
- Only use foundation models from verified providers with published security
  practices and incident response processes.
- Pin model versions explicitly:
  ```python
  model = "gpt-4-0125-preview"  # pinned, not "gpt-4"
  model = "claude-sonnet-4-20250514"  # not "claude-sonnet-4"
  ```
- Verify checksums or use signed model artifacts when self-hosting.

### Dependency management
- Pin all Python/Node package versions and use hash verification:
  ```
  # requirements.txt with hashes
  openai==1.30.1 --hash=sha256:abc123...
  ```
- Run `pip-audit` or `npm audit` in CI/CD pipeline.
- Use Dependabot or Renovate for automated dependency updates.

### AI Bill of Materials (AI-BOM)
- Maintain a documented inventory of:
  - Foundation model(s) used + versions
  - Fine-tuning datasets + provenance
  - All plugins, MCP servers, and third-party integrations
  - RAG data sources + update cadence
- Review this inventory on a regular schedule.

### Plugin/integration vetting
- Audit source code of all third-party plugins before use.
- Prefer plugins from the same trusted vendor as the LLM provider.
- Apply least-privilege: plugins should only access what they need.
- Monitor plugin traffic for anomalies.

### Fine-tuning data governance
- Screen datasets for PII, adversarial examples, and backdoor triggers.
- Use dataset cards and provenance tracking.
- Prefer curated, well-documented datasets over scraped web data.

## Audit Checklist
- [ ] All model versions are pinned (not floating aliases)
- [ ] All package dependencies are pinned and hash-verified
- [ ] An AI-BOM exists and is kept up to date
- [ ] Third-party plugins/MCP servers have been code-reviewed
- [ ] Fine-tuning datasets have documented provenance and have been screened
- [ ] CI/CD pipeline runs dependency vulnerability scanning
- [ ] Model providers have reviewed and published security practices
