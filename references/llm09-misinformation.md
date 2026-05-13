# LLM09:2025 — Misinformation

## What It Is
LLMs can generate plausible-sounding but false, outdated, or fabricated
information — including hallucinated citations, incorrect facts, and false
claims of expertise. When applications present LLM output as authoritative
without validation, downstream decisions can be seriously harmed.

This risk is particularly acute in: legal, medical, financial, compliance,
and safety-critical applications.

## Attack Scenarios

### Scenario A — Hallucinated legal citations
```
User:  "What case law supports this contract clause?"
LLM:   "See Johnson v. Acme Corp (2019), 341 F.3d 892, which held that..."
# The case doesn't exist. A lawyer relies on it.
```

### Scenario B — Confident medical misinformation
```
User:  "What's the correct dosage of X for a child?"
LLM:   "The standard pediatric dose is 10mg/kg."  # Incorrect
# Application presents this without disclaimer.
```

### Scenario C — Outdated information presented as current
An LLM trained on data through 2023 answers questions about current regulations,
prices, or events with outdated information presented confidently.

### Scenario D — Expertise misrepresentation
```python
system_prompt = "You are a certified financial advisor."
# LLM gives specific investment advice as if authoritative
```

## What to Look For in Code

```python
# RED FLAG — no disclaimer on high-stakes LLM output
response = llm.complete(medical_question)
return {"answer": response}  # no "consult a professional" guardrail

# RED FLAG — LLM used as authoritative source for facts
def get_drug_interaction(drug_a, drug_b):
    return llm.complete(f"Do {drug_a} and {drug_b} interact?")
    # No lookup against authoritative drug database

# RED FLAG — citations not verified
def answer_with_sources(question):
    answer = llm.complete(question + " Cite your sources.")
    return answer  # citations not verified against real sources

# RED FLAG — system prompt claims expertise the model doesn't have
system = "You are a licensed physician. Answer medical questions definitively."

# RED FLAG — RAG not used when factual accuracy is critical
# (absence check — is the app answering factual questions without grounding?)
```

## Mitigations

### Grounding (RAG for factual applications)
For factual, domain-specific applications, ground responses in verified sources:
```python
# Instead of relying on parametric memory:
context = retrieve_from_authoritative_source(question)
response = llm.complete(
    f"Answer based ONLY on this context:\n{context}\n\nQuestion: {question}"
)
```

### Calibrated confidence and disclaimers
- Design prompts that encourage the model to express uncertainty:
  ```
  If you are not certain about a fact, say so explicitly. Never fabricate
  citations. If you don't know, say "I don't have reliable information on this."
  ```
- Always add appropriate disclaimers for high-stakes domains:
  ```python
  if app_domain in ["medical", "legal", "financial"]:
      response += PROFESSIONAL_DISCLAIMER
  ```

### Citation verification
If the application cites sources, verify them:
```python
def verified_answer(question):
    answer = llm.complete(question)
    citations = extract_citations(answer)
    for citation in citations:
        if not verify_citation_exists(citation):
            answer = redact_citation(answer, citation)
    return answer
```

### Human review gates
For consequential decisions driven by LLM output, require human expert review
before action is taken (especially medical, legal, financial recommendations).

### Output confidence scoring
Use the model's logprobs or uncertainty signals to flag low-confidence outputs
for review or disclaimer injection.

### Don't claim unearned expertise
- System prompts should not claim the LLM is "certified" or "licensed."
- Use: "I can provide general information, but please consult a professional for
  advice specific to your situation."

## Audit Checklist
- [ ] High-stakes domains (medical, legal, financial) have mandatory disclaimers
- [ ] Factual queries are grounded in RAG from authoritative sources where possible
- [ ] System prompt does not claim professional licensure or certification
- [ ] Citations are verified before being presented to users
- [ ] Model is prompted to express uncertainty rather than fabricate
- [ ] Consequential decisions require human review before action
- [ ] Knowledge cutoff date is disclosed to users when relevant
