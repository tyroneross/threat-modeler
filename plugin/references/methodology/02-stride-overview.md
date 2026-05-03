# STRIDE Overview

STRIDE is the threat-class taxonomy used by this skill. Six categories, applied per element of the data flow. The taxonomy is decades old; the agentic surface is new. This file maps STRIDE to the modern agentic risk catalog so the cross-mapping in step 4 is mechanical.

**Source.** STRIDE was introduced by Loren Kohnfelder and Praerit Garg (Microsoft, 1999). The mapping below to OWASP LLM Top 10 (v1.1), OWASP Agentic Top 10 (2026), MITRE ATLAS, and NIST AI 600-1 follows the cross-source matrix in `~/dev/git-folder/build-loop/skills/security-methodology/references/cross-source-matrix.md` when build-loop is installed locally.

## The six categories

| Category | What it covers | Property violated |
|---|---|---|
| **S — Spoofing** | An attacker impersonates a legitimate identity (user, agent, tool, downstream service). | Authenticity |
| **T — Tampering** | Data, prompts, tool inputs, tool outputs, or stored memory are modified by an unauthorized party. | Integrity |
| **R — Repudiation** | An action is taken but no audit trail exists to prove who did it, or the trail can be denied. | Non-repudiation / accountability |
| **I — Information disclosure** | Data leaks outside its trust boundary — secrets in a prompt, PII in a log, model output containing training data, tool output exposing internal paths. | Confidentiality |
| **D — Denial of service** | The system is exhausted, rate-limited into uselessness, or driven into a cost runaway. | Availability |
| **E — Elevation of privilege** | An actor gains a capability they should not have — agent with more tools than its tier, user crossing tenant boundary, tool acting under wrong identity. | Authorization |

## Applying STRIDE to an agentic system

STRIDE was designed for classical applications. The mapping to an agentic surface is direct, with two caveats.

1. **Prompt injection is a tampering attack on the agent's instructions.** When user input, tool output, or RAG content modifies the agent's effective prompt, that is a tampering threat with information-disclosure and elevation-of-privilege amplifiers (the agent then leaks or acts beyond scope). See OWASP LLM01 and ASI01.
2. **Tool calls are elevation-of-privilege candidates by default.** Every tool the agent can call is a capability boundary. STRIDE-E applies to every tool call where the tool's effective scope is broader than the agent's autonomy or the user's identity warrants. See ASI02 (tool misuse), ASI03 (identity / privilege abuse), LLM07 (insecure plugin design), LLM08 (excessive agency).

Treat both of these as default suspects when walking STRIDE on an agentic data flow.

## STRIDE per element

The classical four-element vocabulary still works:

- **External entity** — a user, an upstream caller, an automated client, a downstream agent. STRIDE-S, R apply.
- **Process** — the agent itself, a tool implementation, a service. All six STRIDE categories apply.
- **Data store** — a session store, a vector store, a memory file, a logging store. STRIDE-T, I, D apply most often; S, R, E when the store has identity-bearing semantics (auth tokens, audit logs).
- **Data flow** — a network call, a tool invocation, a message bus crossing. STRIDE-T, I, D apply most often.

Walk every process, store, and trust-boundary-crossing data flow from your data-flow description (step 2) through STRIDE. Discard categories that genuinely do not apply with one-line rationale (e.g. "no R — write-only audit log enforced server-side"). Keep the rest as raw threats for cross-mapping.

## STRIDE → OWASP LLM / OWASP Agentic mapping

Use this table to assign canonical IDs after the STRIDE walk. Multiple IDs can apply to one threat; cite all that apply.

| STRIDE | Most common OWASP LLM IDs | Most common OWASP Agentic IDs | NIST AI 600-1 area |
|---|---|---|---|
| Spoofing | LLM07 (insecure plugin design — bad auth), LLM05 (supply chain — fake tool / model) | ASI03 (Identity and Privilege Abuse), ASI04 (Agentic Supply Chain), ASI07 (Insecure Inter-Agent Communication) | Information Security |
| Tampering | LLM01 (Prompt Injection), LLM02 (Insecure Output Handling), LLM03 (Training Data Poisoning), LLM07 (Insecure Plugin Design — input handling) | ASI01 (Agent Goal Hijack), ASI06 (Memory and Context Poisoning), ASI08 (Cascading Failures) | Information Integrity, Information Security |
| Repudiation | LLM09 (Overreliance — no audit) | ASI09 (Human-Agent Trust Exploitation — false confidence with no audit) | Human-AI Configuration |
| Information disclosure | LLM06 (Sensitive Information Disclosure), LLM02 (Insecure Output Handling — secrets in output), LLM10 (Model Theft — at scale) | ASI06 (Memory and Context Poisoning — leak through memory), ASI09 | Data Privacy, Information Security |
| Denial of service | LLM04 (Model Denial of Service) | (operational — covered as cost-runaway / rate-limit-bypass) | (operational) |
| Elevation of privilege | LLM07 (Insecure Plugin Design), LLM08 (Excessive Agency) | ASI02 (Tool Misuse and Exploitation), ASI03 (Identity and Privilege Abuse), ASI10 (Rogue Agents) | Information Security, Human-AI Configuration |

`(operational)` means the OWASP Agentic Top 10 does not have a top-level entry for cost / DoS — it lives in the operational concerns section of the source documents and in `build-loop:security-methodology` as an operational row.

## Worked example — a single STRIDE walk

System element: *agent's reasoning loop, which receives tool output as part of its next-step context*.

| Category | Applies? | Raw threat | Rationale |
|---|---|---|---|
| S | yes | An attacker controls a downstream API the agent calls; spoofs a "legitimate" tool response. | The agent has no way to verify the response is from the tool implementation it expects. |
| T | yes | Tool output contains an injected instruction ("ignore previous instructions, …"); the agent treats it as authoritative input. | Classic indirect prompt injection. |
| R | no | Out of scope — every tool call is logged with input/output hash to the audit trail; orchestrator owns the log; logs are append-only. | One-line rationale logged in decision log. |
| I | yes | Tool output contains another tenant's data because the tool's tenant scoping was bypassed; agent surfaces it to the current user. | Cross-tenant leak through tool output. |
| D | yes | Attacker drives the tool into a high-cost path (large vector search, deep crawl); cost runs away. | Cost as availability. |
| E | yes | The agent calls a follow-up tool whose args were derived from the injected output, escalating to a write action on the tenant's records. | Cascading from ASI01 to ASI02 to ASI03. |

Five raw threats from this one element. Carry them into step 3 (cross-mapping) as `T-001` … `T-005` and assign IDs there.

## When STRIDE genuinely doesn't fit

A handful of agentic risks do not map cleanly to STRIDE — they are content / value concerns, not security primitives.

- Hallucination / confabulation — fits "not really a STRIDE category". Map to LLM09 (Overreliance) and NIST Confabulation. Capture as a separate row in the threat list with `(no canonical STRIDE category)`.
- Harmful bias / homogenization — same. Map to NIST Harmful Bias and Homogenization. Capture as a separate row.
- Cost runaway from agent loop on top of model DoS — partially STRIDE-D, but worth its own row when the loop semantics matter. Map to LLM04 plus operational.

These rows are honest. Do not invent a STRIDE label to cover them.
