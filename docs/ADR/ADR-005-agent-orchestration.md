# ADR-005 — Agent Architecture: Deterministic Spine, LLM Decision Points

- **Status:** Accepted
- **Date:** 2026-06-01
- **Deciders:** Agents team, Eng leads
- **Supersedes:** —
- **Superseded by:** —

---

## Context

Vibe's value is in its agents. Each agent (Capture, Analysis, Generation, SEO, Quality, Deployment, Delivery) must be:

- **Reliable.** The same input should produce a similar quality of output.
- **Cost-bounded.** Per-agent cost envelopes are part of the contract.
- **Observable.** Every decision must be traceable.
- **Debuggable.** When something goes wrong, the team must be able to reproduce and fix it.
- **Safe.** An LLM should not have free rein to call any tool with any input.

Alternatives considered:

1. **Pure LLM-driven agents** (the agent decides everything).
2. **Pure deterministic pipelines** (no LLM in the loop).
3. **Hybrid: deterministic spine, LLM decision points** (chosen).
4. **Multiple specialized LLMs** vs. one generalist.
5. **On-device / small models** vs. hosted frontier models.

---

## Decision

Adopt a **deterministic-spine, LLM-decision-point** architecture for every agent.

- Each agent is a finite state machine with explicit states and transitions.
- The transitions are decided by a small, well-tested function.
- When the decision requires semantic judgment, the function calls an LLM with a constrained prompt and a constrained output schema.
- Every tool call is mediated: the agent cannot call a tool without the orchestrator's tool registry providing it, with explicit arguments validated by Pydantic.
- Every step is logged with the prompt, the response, the tool, the cost, and the duration.
- Cost envelopes are enforced in code, not in prompt.

---

## Rationale

### Why hybrid (not pure LLM)

- **Reliability.** Pure LLM agents are non-deterministic. They will, in production, do unexpected things — including calling tools they should not, looping forever, or producing malformed output. The deterministic spine contains the risk.
- **Cost.** Pure LLM agents are expensive. The platform's per-job cost target ($5) is not achievable with general-purpose LLM decision-making at every step.
- **Observability.** A state machine is observable by construction. The team can see exactly what the agent did and why.
- **Debuggability.** When a job fails, the deterministic state machine tells us which transition failed and why.

### Why hybrid (not pure deterministic)

- Modernization is a *semantic* task. The source site is messy, the content is varied, and the target is opinionated. A pure pipeline cannot handle the long tail of edge cases.
- LLMs excel at the *judgment* parts: "what is this page about?", "is this a good headline?", "does this look like a header?". Constraining them to those decision points is the right balance.

### Why a state machine

- **Explicit states** map to recognizable phases (e.g., `analysis.parse`, `analysis.llm.summarize`, `analysis.compose`).
- **Explicit transitions** make the agent's behavior auditable.
- **Standard error handling** is possible: each state has a documented failure mode and a documented retry policy.
- **Testing is straightforward.** Tests pin inputs and expected transitions.

### Why mediated tools

- **Safety.** The LLM cannot, by construction, call `rm -rf` or `child_process.exec` on a worker.
- **Auditability.** Every tool invocation is logged with the LLM's intent (the prompt and the parsed tool call).
- **Reusability.** The same tool registry is used across agents and across human operators.
- **Versioning.** Tools are versioned; agents pin to a version.

### Why constrained outputs (Pydantic / Zod)

- An LLM that returns free-form text is a liability. Constrained outputs (JSON Schema) make responses machine-checkable.
- Pydantic and Zod give us validation with helpful errors.
- The platform uses Pydantic v2 server-side and Zod client-side; both enforce the same schema.

### Why cost envelopes in code

- An LLM cannot be trusted to stop spending on its own. The orchestrator enforces a hard cap, and a soft target, per phase.
- The cap is part of the agent's contract. A change to the cap is a code change, not a prompt change.

### Why a multi-model strategy

- The default is a *router* that picks the cheapest model that meets the quality bar for a given phase.
- For MVP, we use two providers and three model tiers (small/cheap, medium, large). A/B tests in CI tell us when to upgrade a phase to a more capable model.
- The router includes circuit breakers; an outage on one provider triggers failover to another.

### Why not on-device / small models

- The state of small models in 2026 is improving, but the long tail of edge cases in source sites still benefits from larger models. We will revisit at V3 if small models close the gap.

---

## The Spine Pattern (Detailed)

For each agent:

```python
class AgentState(StrEnum):
    INIT = "init"
    LOAD_INPUTS = "load_inputs"
    DECIDE_NEXT = "decide_next"  # LLM call
    EXECUTE_TOOL = "execute_tool"  # tool registry call
    VALIDATE_OUTPUT = "validate_output"
    EMIT_RESULT = "emit_result"
    HANDLE_ERROR = "handle_error"
    DONE = "done"
    FAILED = "failed"

class Agent:
    def __init__(self, ctx: AgentContext, llm: LLMClient, tools: ToolRegistry):
        self.state = AgentState.INIT
        self.ctx = ctx
        self.llm = llm
        self.tools = tools
        self.cost = 0.0
        self.attempts = 0

    async def run(self) -> AgentResult:
        while self.state not in (AgentState.DONE, AgentState.FAILED):
            await self._tick()
        return self._result()

    async def _tick(self) -> None:
        # The spine: explicit transition logic.
        match self.state:
            case AgentState.INIT:
                ...
            case AgentState.LOAD_INPUTS:
                ...
            case AgentState.DECIDE_NEXT:
                await self._decide_next()
            case AgentState.EXECUTE_TOOL:
                await self._execute_tool()
            ...
```

The state machine is a Temporal workflow (or a sub-flow). Each state is an activity. Failures are explicit. Retries are bounded.

### Tool Registry

Tools are Python functions decorated with metadata:

```python
@tool(
    name="fetch_page",
    description="Fetch a page via the browser pool.",
    input_schema=FetchPageInput,
    output_schema=FetchPageOutput,
    cost_units=1,
    requires=["browser:read"],
)
async def fetch_page(input: FetchPageInput) -> FetchPageOutput: ...
```

The agent's LLM is presented with a JSON Schema derived from the registered tools. The orchestrator validates the parsed tool call against the schema and permissions, then executes it.

### Cost Envelope

```python
class CostGuard:
    def __init__(self, cap_usd: float):
        self.cap = cap_usd
        self.spent = 0.0

    def charge(self, usd: float) -> None:
        if self.spent + usd > self.cap:
            raise CostExceeded(spent=self.spent, cap=self.cap)
        self.spent += usd
```

The guard wraps every LLM call. A breach is a hard error that propagates up the workflow.

### Prompt Versioning

- Prompts are stored as files in the agent's package, versioned with the code.
- A/B tests in CI score prompt changes against a golden fixture set.
- A regression in score fails the PR.
- Promotion to production is gated by the test.

---

## Consequences

### Positive

- **Predictable cost.** Envelopes are code; the platform never goes wildly over budget on a job.
- **Predictable behavior.** A state machine is testable; the team can ship confidently.
- **Auditable.** Every transition is logged with the prompt, response, tool, cost, and duration.
- **Cheap iteration.** Prompt changes are reviewed like code, with tests and gates.
- **Safe.** Tools are mediated; the LLM cannot do anything the registry does not allow.

### Negative

- **Engineering overhead.** Each agent is a state machine, not a prompt. This is more code, but the right amount of code.
- **Spine rigidity.** The state machine can become a constraint when a problem demands more flexible behavior. Mitigated by allowing a state to delegate to a "free-form" sub-agent for tightly scoped tasks.
- **Tool design burden.** Every tool needs a schema, a description, and tests. This is intentional; tools are the platform's leverage.

### Neutral

- We accept that the architecture is opinionated. The trade-off is reliability and cost over flexibility.

---

## Alternatives Considered

| Option | Verdict |
|--------|---------|
| Deterministic spine + LLM decision points | **Chosen** |
| Pure LLM-driven agents | Rejected — unreliable, expensive, unsafe |
| Pure deterministic pipelines | Rejected — cannot handle semantic tasks |
| ReAct / AutoGPT-style loop | Rejected — unconstrained loops, hard to bound |
| Code-writing LLM agent (e.g., SWE-Agent) | Rejected — overkill, ungrounded |
| Fine-tuned small model for every agent | Considered for V3 — premature at MVP |

---

## Notes

- The agent architecture is the most important decision in the platform. It is the most expensive to change after launch.
- A dedicated Agents team owns the spine, the tool registry, and the prompt library.
- The architecture is reviewed at every Tier 1 risk change and at every milestone gate.
- The "self-repair" loop in the Generation agent is a state machine with a small LLM call per repair attempt. It is capped at 3 attempts.
- An "agent eval" suite (offline traces) is built at M2 and runs nightly from M3 onward.
