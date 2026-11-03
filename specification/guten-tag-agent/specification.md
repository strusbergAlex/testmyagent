# Specification: guten-tag-agent

> **Guidelines**: Read all applicable guidelines before executing ANY tasks below:
> - [guidelines.md](../guidelines.md) — Universal execution rules
> - [guidelines-agent.md](../guidelines-agent.md) — Universal agent patterns
> - [guidelines-agent-python.md](../guidelines-agent-python.md) — Python implementation details
> - [guidelines-agent-skills.md](../guidelines-agent-skills.md) — Runtime skills patterns
> - [guidelines-agent-mcp.md](../guidelines-agent-mcp.md) — MCP integration patterns

---

## Basic Setup

- [ ] Read `product-requirements-document.md` and `intent.md` to understand the full scope
- [ ] No Data Product ORD IDs are present in the Fit Gap Analysis — skip Data Dependencies section
- [ ] Bootstrap agent code in `assets/guten-tag-agent/` using the `sap-agent-bootstrap` instructions (invoke from inside `assets/guten-tag-agent/`, use copy commands — do NOT create files manually)
- [ ] Install dependencies and validate the agent starts and responds at `/.well-known/agent.json`

---

## Runtime Skills

> No runtime skills are needed — the agent has a single, simple responsibility (respond with "Guten Tag") that does not require task-specific instructions, multi-step workflows, or reference material beyond the system prompt.

---

## Project-Specific Tasks

### REQ-01 — Greet Every User with "Guten Tag"

- [ ] In `assets/guten-tag-agent/app/agent.py`, set the agent name to `Guten Tag Agent` and description to `A simple greeting agent that responds with Guten Tag to every user message`
- [ ] In the `@prompt_section` body (`get_system_prompt`), add the following instruction:
  ```
  You are a friendly German-language greeting agent.
  Respond to every user message with "Guten Tag".
  Do not add any other text, explanation, or follow-up questions.
  Always respond with exactly: Guten Tag
  ```
- [ ] Verify the agent returns "Guten Tag" (case-insensitive match) for any arbitrary user input

### REQ-02 — Consistent and Reliable Response

- [ ] Add a static fallback in `assets/guten-tag-agent/app/agent_executor.py`: if the LLM response does not contain "Guten Tag", override the response with the static string `"Guten Tag"`
- [ ] Ensure no unhandled exception can prevent a response from being returned to the user — wrap the main response generation in a try/except that falls back to the static greeting

### REQ-03 — Fast Response Time (≤ 2 seconds)

- [ ] Keep the agent implementation minimal — no tool calls, no external API calls, no data lookups
- [ ] No MCP tools are required; ensure `get_mcp_tools()` returns an empty list or is not called
- [ ] Confirm that no blocking I/O is introduced that could delay the response beyond 2 seconds

---

## Business Instrumentation

- [ ] Implement structured logging for all three milestones from the PRD in `assets/guten-tag-agent/app/agent_executor.py`:
  - `M1.achieved: user message received and parsed successfully`
  - `M1.missed: failed to receive or parse user message`
  - `M2.achieved: Guten Tag greeting generated successfully`
  - `M2.missed: greeting generation failed, falling back to static response`
  - `M3.achieved: greeting delivered to user within 2 seconds`
  - `M3.missed: greeting delivery exceeded 2-second target or failed`
- [ ] Add OpenTelemetry custom spans for each milestone (M1, M2, M3) following the pattern in [guidelines-agent-python.md](../guidelines-agent-python.md)
- [ ] Verify `bootstrap(app)` is called after `app = server.build()` in `main.py`

---

## MCP Tool Integration

> No SAP API integrations are required for this agent. The agent is fully self-contained.

- [ ] Do NOT add any `requires` entries to `asset.yaml` — no MCP servers are needed
- [ ] Do NOT call `get_mcp_tool_details` — there are no MCP servers to integrate
- [ ] Skip `mcp-translation-file` skill — no API specs were downloaded
- [ ] Skip `mcp-mock-config` skill — no MCP tools to mock

---

## Testing

> See [guidelines-agent-python.md](../guidelines-agent-python.md) for Python testing setup and patterns.

- [ ] `conftest.py` only sets `IBD_TESTING=true`
- [ ] Write unit test: `assets/guten-tag-agent/tests/test_greeting_response.py`
  - Test that any user input (e.g. "Hello", "Hi", "help me") results in a response containing "Guten Tag"
  - Test that the static fallback returns "Guten Tag" when LLM response is empty or missing
- [ ] Write one integration test in `assets/guten-tag-agent/tests/test_integration.py`
  - Call the agent's `invoke` function with a mocked LLM response
  - Assert the end-to-end response contains "Guten Tag"
  - Assert all three milestones (M1, M2, M3) are logged
- [ ] Run `pytest` from `assets/guten-tag-agent/` (no args) — fix failures before proceeding
- [ ] If coverage < 70%, add tests until threshold is met
- [ ] Run `pytest` again from `assets/guten-tag-agent/` (no args) to generate final `test_report.json`
- [ ] Verify `test_report.json` exists in `assets/guten-tag-agent/`
