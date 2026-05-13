# LLM05:2025 — Improper Output Handling

## What It Is
Failure to validate, sanitize, or safely handle LLM-generated output before
it is used downstream. LLM output should be treated as **untrusted input**
to any subsequent system, just as you would treat user input.

## Attack Scenarios

### Scenario A — XSS via LLM output
```python
# LLM generates: <script>document.cookie</script>
response = llm.complete(f"Write a message for: {user_input}")
return f"<div>{response}</div>"  # XSS if rendered in browser
```

### Scenario B — Code execution
```python
code = llm.complete("Write Python to parse this CSV")
exec(code)  # arbitrary code execution
```

### Scenario C — SQL injection via LLM
```python
query_fragment = llm.complete(f"Write SQL WHERE clause for: {user_request}")
db.execute(f"SELECT * FROM users WHERE {query_fragment}")
```

## What to Look For in Code
- Any use of `eval()`, `exec()`, `subprocess`, or `os.system()` with LLM output
- LLM output rendered directly as HTML without escaping
- LLM output used to construct SQL, shell commands, or file paths
- LLM output passed to other APIs without validation

## Mitigations
- **Never** pass LLM output directly to `eval()` or shell execution
- Parse structured output via schema (JSON Schema, Pydantic) — reject on failure
- Escape LLM output before rendering in HTML (use templating engine escaping)
- Use parameterized queries — never string-concatenate LLM output into SQL
- Run LLM-generated code in a sandboxed environment (Docker, WASM, subprocess
  with restricted permissions)
- Apply output length limits and content policy checks before use

## Audit Checklist
- [ ] No `eval()`/`exec()` on LLM output
- [ ] LLM output is HTML-escaped before rendering
- [ ] SQL queries use parameterization, not string interpolation
- [ ] LLM-generated code runs in a sandbox
- [ ] Structured output is schema-validated before use
- [ ] Output length and content limits are enforced
