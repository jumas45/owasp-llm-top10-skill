# LLM10:2025 — Unbounded Consumption

## What It Is
LLM inference is computationally expensive. Unbounded consumption occurs when
an application allows excessive or uncontrolled use of LLM resources, enabling:
- **Denial of Service (DoS)** — overwhelming the app for other users
- **Denial of Wallet (DoW)** — inflating API costs via excessive requests
- **Model extraction** — reconstructing model behavior through massive querying
- **Resource exhaustion** — crashing the hosting infrastructure

## Attack Scenarios

### Scenario A — Denial of Wallet via prompt flooding
```python
# Attacker sends 10,000 requests with max-token prompts
# Each request costs $0.10 → $1,000 bill in minutes
# No rate limiting in place
```

### Scenario B — Context window bomb
```
User: [pastes 500,000 token document] "Summarize this."
# At $0.01/1K tokens input, single request = $5
# No input length limit, no authentication
```

### Scenario C — Recursive or nested agent loops
```python
def agent_loop(prompt):
    while True:
        response = llm.complete(prompt)
        if "DONE" in response:
            break
        prompt = f"Continue: {response}"  # no iteration limit
# Prompt injection causes infinite loop → runaway costs
```

### Scenario D — Model extraction
Attacker makes thousands of targeted queries to map the model's decision
boundaries, effectively replicating fine-tuned behavior. No rate limiting
enables this at scale.

## What to Look For in Code

```python
# RED FLAG — no rate limiting on LLM endpoint
@app.route("/api/chat", methods=["POST"])
def chat():  # no rate limit decorator
    return llm.complete(request.json["message"])

# RED FLAG — no input token limits
response = openai.chat.completions.create(
    model="gpt-4",
    messages=messages,
    max_tokens=4096  # output limit set, but input not checked
)

# RED FLAG — no iteration limit in agent loop
for step in agent.run_until_complete(task):  # no max_steps

# RED FLAG — no timeout on LLM calls
response = llm.complete(prompt)  # no timeout parameter

# RED FLAG — no cost monitoring or alerting
# (absence check — no billing alerts in cloud console)

# RED FLAG — unauthenticated LLM endpoint
# Public endpoint with no auth = open invitation to abuse
```

## Mitigations

### Rate limiting
```python
from slowapi import Limiter
limiter = Limiter(key_func=get_remote_address)

@app.route("/api/chat")
@limiter.limit("20/minute")
@limiter.limit("200/hour")
def chat():
    ...
```

### Input validation and limits
```python
MAX_INPUT_TOKENS = 4000

def chat(user_message):
    token_count = count_tokens(user_message)
    if token_count > MAX_INPUT_TOKENS:
        return {"error": "Message too long. Maximum 4,000 tokens."}
    ...
```

### Agent safeguards
```python
MAX_ITERATIONS = 10
MAX_COST_PER_SESSION = 0.50  # USD

agent = Agent(
    tools=tools,
    max_iterations=MAX_ITERATIONS,
    timeout_seconds=30,
    budget_tokens=50000
)
```

### Timeouts
```python
response = openai.chat.completions.create(
    model="gpt-4",
    messages=messages,
    max_tokens=1000,
    timeout=30  # 30 second timeout
)
```

### Authentication
- All LLM-powered endpoints must require authentication.
- Per-user token/cost budgets enforced at the application layer.

### Cost monitoring and alerting
- Set billing alerts at 50%, 80%, 100% of expected monthly budget.
- Monitor per-user and per-session spend in real time.
- Auto-disable accounts that exceed spend thresholds pending review.

### Output limits
- Always set `max_tokens` on every LLM call — never leave it unbounded.
- Set streaming timeouts so long-running generations can be terminated.

## Audit Checklist
- [ ] Rate limiting is applied to all LLM-facing endpoints
- [ ] Input token length is validated before each LLM call
- [ ] `max_tokens` is set on every LLM API call
- [ ] Agentic loops have a maximum iteration count and timeout
- [ ] All LLM endpoints require authentication
- [ ] Per-user spend/token budgets are enforced
- [ ] Billing alerts are configured at meaningful thresholds
- [ ] Cost anomaly monitoring is in place
- [ ] Long-running LLM calls have timeout handling
