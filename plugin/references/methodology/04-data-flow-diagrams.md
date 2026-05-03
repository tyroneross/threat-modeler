# Data Flow Description

Step 2 of the workshop. Output: a labeled markdown description of how data and control move through the system, naming every place where they cross a trust boundary identified in step 1.

The classical name is "data flow diagram" (DFD). This skill prefers structured prose over a visual diagram by default, because text is greppable, diff-friendly, and version-controllable. Mermaid is welcome when the flow is genuinely easier to read graphically; ASCII or numbered lists are equally acceptable.

## Vocabulary

A data flow description uses four element types — the same four STRIDE walks against in step 3.

- **External entity** — anything outside the trust boundary you are modeling. Users, upstream callers, model providers, third-party APIs.
- **Process** — anything inside the trust boundary that transforms or routes data. The agent's reasoning loop, a tool implementation, an orchestrator, a router.
- **Data store** — anywhere data persists. Session store, vector store, memory file, audit log, conversation history.
- **Data flow** — the named arc between two of the above. Includes the medium (HTTP, in-process, message queue), the schema, and the trust boundary it crosses (or does not).

Each element gets a stable ID. Conventional prefixes: `EE-` external entity, `P-` process, `DS-` data store, `DF-` data flow. The IDs make the STRIDE walk in step 3 readable.

## What "level" of detail to use

Shallow enough that the whole flow fits on one screen. Deep enough that every trust-boundary crossing has a `DF-NNN` ID.

If the system has a single agent calling three tools, one diagram covers it. If the system is an orchestrator with seven specialists, draw the orchestrator's flow at level 0, then drill into each specialist that surfaces a risk-relevant boundary at level 1. Do not flatten everything into one mega-flow.

## Required content

### 1. Element list

A short table or list of every element with its ID, type, and one-line description.

```markdown
| ID | Type | Name | Description |
|---|---|---|---|
| EE-1 | External entity | End user | Authenticated human; one user per session. |
| EE-2 | External entity | Model provider | OpenAI / Anthropic API. |
| EE-3 | External entity | GitHub API | Source of PR diffs and target of review comments. |
| P-1 | Process | Agent reasoning loop | Single LLM call per turn; consumes user input and tool output. |
| P-2 | Process | `run-lint` tool | Executes `./scripts/lint.sh` in a sandbox. |
| P-3 | Process | `run-tests` tool | Executes `./scripts/test.sh` in a sandbox. |
| P-4 | Process | `github-comment` tool | Posts one review comment to the PR. |
| DS-1 | Data store | Session log | Append-only audit of inputs, outputs, tool calls. |
| DS-2 | Data store | Cache | Per-PR cache of fetched files (24h TTL). |
```

### 2. Flow list

Every named arc, with the trust boundary it crosses (if any).

```markdown
| ID | Source | Sink | Crosses boundary? | Carries |
|---|---|---|---|---|
| DF-1 | EE-1 | P-1 | yes — user→agent | user prompt, repo target |
| DF-2 | P-1 | EE-2 | yes — agent→model provider | system prompt + user prompt + tool outputs |
| DF-3 | EE-2 | P-1 | yes — model provider→agent | model output (incl. proposed tool calls) |
| DF-4 | P-1 | P-2 | no — in-process | lint command args |
| DF-5 | P-2 | P-1 | no — in-process | lint stdout/stderr |
| DF-6 | P-1 | EE-3 | yes — agent→external API | review comment payload |
| DF-7 | P-1 | DS-1 | no — internal append | audit row |
```

The "Crosses boundary?" column is what step 3 walks against. STRIDE-T, STRIDE-I, and STRIDE-D apply to every `yes` row. STRIDE-S and STRIDE-E apply to every flow whose sink takes action on behalf of the source's identity.

### 3. Annotations

Each interesting flow gets at most a sentence or two of context. "Interesting" means: crosses a trust boundary, carries authentication material, carries user-controlled data into a place that interprets it (prompts, query languages, shell, file paths), or has a meaningful failure mode (rate limits, partial writes, retries).

## Optional — mermaid

When prose is genuinely harder to read than a graph, include a mermaid block. Keep it small.

```markdown
```mermaid
flowchart LR
    EE1[End user] -->|DF-1| P1((Agent loop))
    P1 -->|DF-2| EE2[Model provider]
    EE2 -->|DF-3| P1
    P1 -->|DF-4| P2[run-lint]
    P2 -->|DF-5| P1
    P1 -->|DF-6| EE3[GitHub API]
    P1 -->|DF-7| DS1[(Session log)]
    classDef boundary stroke:#f33,stroke-dasharray:4 4
    class EE1,EE2,EE3 boundary
``` ```

The dashed border on `EE1`, `EE2`, `EE3` marks them as external entities (across a trust boundary). This is a convention, not a hard rule.

## Trust boundaries to make sure are named

When scanning the flow list, confirm each of these boundaries is explicitly listed if it exists in the system. Missing one usually means a missing STRIDE row in step 3.

- **User → agent**. Where user input reaches the agent's prompt-construction step.
- **Agent → tool**. Every tool call is a boundary; the tool may run with different credentials than the agent.
- **Tool → agent**. The agent re-reads the tool output. Every tool that returns natural language is a potential indirect injection vector.
- **Agent → external API**. Egress. Carries credentials and possibly user data.
- **Agent ↔ memory store**. Both directions. ASI06 needs both arcs.
- **Agent → downstream agent**. A2A handoff if multi-agent. Carries identity (or fails to).
- **Operator → system config**. Out-of-band, but listed because the operator boundary affects what S, R, E look like in production.

## What this section is not

- It is not a UML diagram. UML actor / use-case diagrams are about behavior; STRIDE is about flow.
- It is not a sequence diagram. Sequence is fine for a runbook; for STRIDE you need flows and boundaries, not chronological steps.
- It is not the architecture document. Architecture covers components, dependencies, deployment. Threat-modeler's data flow is **scoped to risk** — only the elements that participate in a trust boundary or in a flow carrying sensitive data need to appear.

If the system already has an architecture doc, cross-reference it by absolute path. If the system has an agent-builder `flow-topology.md` artifact, cite that and reuse its element IDs to keep the cross-walk free.

## Output shape for step 2

The artifact's `## Data Flow` section uses the schema in `references/templates/threat-model.md`. The element list, the flow list, and (optionally) the mermaid block from this step land directly into that section.
