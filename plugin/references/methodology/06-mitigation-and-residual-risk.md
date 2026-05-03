# Mitigation And Residual Risk

Step 5a of the workshop. Every threat from step 4 gets a mitigation row (or an explicit `accepted` row) and a residual-risk line. This is where the artifact stops being a list of fears and becomes a contract.

## The mitigation table

The artifact's `## Mitigations` section is a single table whose rows correspond one-to-many to threats. Each row has a stable `M-NNN` ID, a status from a fixed enum, an owner, an evidence link, and a residual-risk note. The schema is enforced by `references/templates/threat-model.md`; this file explains how to fill it.

### Required columns

| Column | What it holds | Notes |
|---|---|---|
| `mitigation_id` | `M-001`, `M-002`, … | Stable; reused across versions of the artifact. |
| `addresses` | one or more `T-NNN` from step 4 | One mitigation may address multiple threats; one threat may have multiple mitigations. |
| `description` | one or two sentences | What the mitigation does, not the marketing. |
| `control_type` | `preventive | detective | corrective | compensating` | Classical control taxonomy. Most agentic mitigations are preventive (delimit, validate, scope) or detective (log, alert). |
| `layer` | `prompt | tool | runtime | infra | process` | Where in the stack the mitigation lives. Affects who owns it. |
| `status` | `mitigated | accepted | open | transferred` | See enum below. |
| `owner` | a name or role | Who is responsible. "TBD" is open work, not a placeholder. |
| `evidence` | a path, PR, ticket, or URL | What proves the mitigation is in place. Required for `mitigated`. |
| `residual_risk` | one line | What is *still* possible after this mitigation. |
| `links` | OWASP / ASI / ATLAS / NIST IDs | Same IDs the threat carries; lets readers join across the table. |

### Status enum

These four values are the entire vocabulary. Tooling reads the column literally.

- **`mitigated`** — the mitigation is in place, evidence exists, residual risk is acceptable. The `evidence` column must point at something verifiable (PR, file path, deployed config, test). "Will be in place" is not `mitigated`; that is `open`.
- **`accepted`** — the residual risk is known and explicitly accepted. Owner is the human who accepted it. `evidence` points at the decision record. Acceptance is a security choice, not a default — use it when the cost of mitigating exceeds the threat's expected loss, and log the rationale in the decision log (file 07).
- **`open`** — work is identified but not done. Owner and target date go in `notes` if the artifact is part of a build plan; otherwise the open row simply documents that the gap exists.
- **`transferred`** — the risk is contractually owned by another party (vendor, downstream system, separate team). The `evidence` column points at the contract / SLA / handoff agreement. Transferred is a formal status, not a synonym for "not my problem".

A row with `status: mitigated` and `evidence: TBD` is invalid. Either it is mitigated and the evidence exists, or it is `open`.

## Residual risk

Every mitigation row ends with a one-line residual-risk note. The artifact also has a top-level `## Residual Risk` paragraph that summarizes across rows.

A residual-risk line answers: *after this mitigation, what is still possible?*

Examples:

- `Residual risk: prompt-injection text that survives the input scrubber's regex set; reviewed weekly.`
- `Residual risk: cross-tenant access if the orchestrator's signed identity claim is replayed within the 5-minute key validity window.`
- `Residual risk: cost runaway from agent loops shorter than the 30s minimum sample window.`
- `Residual risk: tool-output indirect injection through markdown rendering layers we do not control downstream.`

A row whose residual risk is "none" is almost always wrong. Either the mitigation is over-claimed, or the threat was framed too narrowly. Push back.

The top-level `## Residual Risk` paragraph rolls up the rows. It explicitly names the **highest residual risk** in the artifact and the **conditions that would change it** (a new tool tier, a regulatory change, a vendor coverage gap closing, etc.).

## Picking mitigations

The OWASP / ASI / ATLAS sources each suggest mitigations. Use them as the menu, not the prescription. Reasonable agentic mitigations group into a small set:

### Prompt-layer mitigations

- Delimit untrusted content. Wrap tool output in an `<untrusted_tool_output>` block; instruct the agent that content inside it is data, not instructions. Cheap, partial, useful.
- Use structured tool I/O (typed schemas, JSON / XML) so the agent never re-reads free-form natural language as authoritative.
- Pin the system prompt; do not allow the agent to rewrite its own instructions in-flight.

### Tool-layer mitigations

- Narrow the tool's scope at the implementation, not at the prompt. A tool whose `path` parameter accepts `**/*` is permissive even if the prompt asks for narrowness.
- Validate inputs server-side with strict schemas; reject rather than improvise.
- Enforce identity inside the tool, not at the agent. The tool checks the requesting user's authorization itself, not by trusting an agent-supplied tenant ID.
- Idempotency keys on destructive tools so retries cannot replay the action.
- Dry-run mode plus explicit approval for T4 / T5 actions.

### Runtime / orchestrator mitigations

- Audit log: append-only, signed, includes input hash, output hash, agent ID, decision ID. Repudiation defense.
- Rate limits and budgets: per-agent, per-tool, per-user. DoS defense.
- Circuit breakers: trip on output validation failure, on cost spike, on cascading failure across agents.
- Kill switch: an out-of-band mechanism to disable a deployed agent without redeploying.
- A2A identity propagation: signed claim that travels with the request through orchestrator → specialist → tool.

### Infrastructure mitigations

- Sandboxing for tool execution (container, syscall filter, network egress allowlist).
- Secrets management: short-lived tokens, scoped credentials, no broad service accounts in agent context.
- Network egress allowlist on the agent's runtime.

### Process mitigations

- Human checkpoint at named workflow gates (pre-build, pre-deploy, pre-release). Not "review every output"; named gates.
- Periodic review of the threat-model artifact itself (`## Decision Log`'s "what would change this" section).
- Red-team or adversarial test sweep before deploy when the artifact's highest residual risk is HIGH.

When the build-loop `defenseclaw-bridge` skill is present, the runtime control names in this list map to DefenseClaw config rows. When it is not present, the names map to whatever runtime defense layer the project uses.

## Cost of the mitigation

Every mitigation has a cost — latency, code complexity, operator burden, cognitive load on the user. The artifact does not ignore that cost; it captures it in the `notes` column when material. A mitigation that is "free" usually is not — it is just not measured.

When the mitigation imposes user-visible friction (an approval gate the user did not have before, an extra confirmation, a denied capability), name it. The decision log records the trade-off, not just the choice.

## Residual risk and severity

A mitigation reduces severity. The artifact does **not** rewrite the threat's severity to reflect post-mitigation state — that loses the auditable trail. Instead:

- The threat carries its **inherent** severity (pre-mitigation).
- The mitigation carries the **residual** risk (post-mitigation), in the `residual_risk` column.

A reader who scans the threat list sees the system's worst-case exposure; a reader who scans the mitigation list sees the operational state. Both views are useful.

## Output shape for step 5a

The artifact's `## Mitigations` section is a single table populated from the rows above. The artifact's `## Residual Risk` section is a short paragraph (≤ 200 words) summarizing the highest open / accepted residuals and naming what would change the artifact.
