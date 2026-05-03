# Mapping STRIDE Threats To OWASP / ATLAS / NIST

Step 4 of the workshop. Every raw STRIDE threat from step 3 gets a stable threat ID (`T-NNN`) and is tagged with its canonical IDs from the four frameworks the threat-model artifact cross-cites:

- **OWASP Top 10 for LLM Applications (v1.1)** — `LLM01`–`LLM10`. Content / model layer risks.
- **OWASP Top 10 for Agentic Applications (2026)** — `ASI01`–`ASI10`. Orchestration / execution layer risks. Stacks on top of LLM Top 10.
- **MITRE ATLAS** — adversary technique IDs (`AML.T0019` etc.). Adversary perspective; cite when it sharpens a finding.
- **NIST AI 600-1** — risk-area names (`Information Integrity`, `Data Privacy`, etc.). Regulator-facing; cite at the end, not at the start.

The IDs are the load-bearing artifact. `build-loop:plan-verify` rule 10 keys off them, the build-loop `security-reviewer` agent grades against them, and any downstream tooling reads them.

## Source of truth for IDs

| Framework | Canonical source | Local copy when present |
|---|---|---|
| OWASP LLM Top 10 (v1.1) | `https://genai.owasp.org/resource/llm-ai-security-and-governance-checklist/` | `~/dev/git-folder/build-loop/skills/security-methodology/references/owasp-llm-top-10.md` |
| OWASP Agentic Top 10 (2026) | `https://genai.owasp.org/resource/owasp-top-10-for-agentic-applications-for-2026/` | `~/dev/git-folder/build-loop/skills/security-methodology/references/owasp-agentic-top-10.md` |
| MITRE ATLAS | `https://atlas.mitre.org/` | `~/dev/git-folder/build-loop/skills/security-methodology/references/mitre-atlas-starter.md` |
| NIST AI 600-1 | `https://nvlpubs.nist.gov/nistpubs/ai/NIST.AI.600-1.pdf` | `~/dev/git-folder/build-loop/skills/security-methodology/references/nist-600-1-mapping.md` |
| Cross-source matrix | (build-loop's own synthesis) | `~/dev/git-folder/build-loop/skills/security-methodology/references/cross-source-matrix.md` |

When the build-loop plugin is not installed locally, the canonical research file at `~/dev/research/topics/product-dev/product-dev.agentic-systems-security-references.md` carries the same matrix.

## Verified labels (use exactly)

Always cite IDs with the framework's exact short form. The label after the ID is for human readers; the ID is what tooling matches on.

### OWASP LLM Top 10 (v1.1)

```text
LLM01 — Prompt Injection
LLM02 — Insecure Output Handling
LLM03 — Training Data Poisoning
LLM04 — Model Denial of Service
LLM05 — Supply Chain Vulnerabilities
LLM06 — Sensitive Information Disclosure
LLM07 — Insecure Plugin Design
LLM08 — Excessive Agency
LLM09 — Overreliance
LLM10 — Model Theft
```

### OWASP Agentic Top 10 (2026)

```text
ASI01 — Agent Goal Hijack
ASI02 — Tool Misuse and Exploitation
ASI03 — Identity and Privilege Abuse
ASI04 — Agentic Supply Chain Vulnerabilities
ASI05 — Unexpected Code Execution
ASI06 — Memory and Context Poisoning
ASI07 — Insecure Inter-Agent Communication
ASI08 — Cascading Failures
ASI09 — Human-Agent Trust Exploitation
ASI10 — Rogue Agents
```

The Agentic Top 10 was released 2025-12-09. Treat it as "best framing available", not a decade-tested standard. Field practice with this taxonomy is still early.

## How to assign IDs

For each raw threat from the STRIDE walk:

1. **Start at the cross-source matrix.** Open `~/dev/git-folder/build-loop/skills/security-methodology/references/cross-source-matrix.md` (or the upstream research file). Find the row whose risk class matches the raw threat's intent.
2. **Read across.** That row gives you the OWASP LLM ID, OWASP Agentic ID, NIST 600-1 area, and the DefenseClaw control that fits the row.
3. **Cite both LLM and Agentic IDs when both apply.** Agentic risks stack on LLM risks. An agent that calls a tool whose output gets re-injected is two risks in one threat: ASI01 (the goal hijack) plus LLM01 (the prompt injection that did it). Cite both.
4. **MITRE ATLAS IDs are optional.** Add them when the technique sharpens the finding. Do not pad the artifact with ATLAS IDs that do not name something more specific than the OWASP ID already does.
5. **NIST areas go last.** The NIST 600-1 risk-area name is for the audit-grade report. Add it when the artifact will be read by a regulator or compliance reviewer.
6. **`(no canonical mapping)` is honest.** Some risks (hallucination, harmful bias, cost-runaway-as-loop) do not have a clean OWASP / ASI fit. Mark them so. Do not invent an ID.

## Common patterns

The same threat shapes show up over and over. The patterns below cover most of what you will see in step 3.

### Pattern: user input flows into a system prompt

- Raw threat (step 3): `Tampering on the agent prompt — user input concatenated into the system prompt allows the user to override agent goals.`
- IDs: **LLM01** (Prompt Injection — the technique) + **ASI01** (Agent Goal Hijack — the agentic consequence).
- NIST area: Information Integrity.
- Severity calibration: HIGH if the agent has any tool above T1; MEDIUM in a draft-only (A0) agent.

### Pattern: tool output is treated as instructions

- Raw threat: `Tampering / Elevation — tool output containing natural language is concatenated into the agent's next-step prompt without delimiter or sanitization.`
- IDs: **LLM01** + **LLM07** (Insecure Plugin Design — the tool that lets this happen) + **ASI01** + **ASI06** (Memory and Context Poisoning if the output is also stored).
- NIST area: Information Integrity + Information Security.
- Severity: HIGH or CRITICAL when the tool is at T2+ or its output drives a follow-up tool call.

### Pattern: agent uses ambient credentials when action is user-scoped

- Raw threat: `Elevation — tool call uses the agent's service-account token; the action is conceptually scoped to the requesting user; cross-user access is not enforced inside the tool.`
- IDs: **LLM07** + **ASI03** (Identity and Privilege Abuse). For multi-tenant systems also cite **OWASP Web A01** (Broken Access Control).
- NIST area: Information Security.
- Severity: CRITICAL by default for cross-tenant; HIGH for same-tenant privilege drift.

### Pattern: tool added to the agent that the agent did not need

- Raw threat: `Elevation — agent has more capability than its job requires; a future hijack can use the surplus capability.`
- IDs: **LLM08** (Excessive Agency) + **ASI02** (Tool Misuse and Exploitation).
- NIST area: Human-AI Configuration.
- Severity: scales with the surplus tool's tier; T4 / T5 surplus is HIGH.

### Pattern: persistent memory accepts agent-written content unchecked

- Raw threat: `Tampering — agent writes natural-language memory content; later runs read that memory back as authoritative context; an attacker can poison once, persist forever.`
- IDs: **ASI06** (Memory and Context Poisoning) + (implicit LLM01 on the recall side).
- NIST area: Information Integrity.
- Severity: HIGH; ASI06 is one of the highest-impact agentic risks because it survives session boundaries.

### Pattern: A2A handoff drops user identity

- Raw threat: `Spoofing / Elevation — orchestrator hands off to a specialist agent; the user identity is replaced with the orchestrator's service identity; specialist enforces its own scope and cannot enforce the user's.`
- IDs: **ASI03** + **ASI07** (Insecure Inter-Agent Communication).
- NIST area: Information Security.
- Severity: HIGH or CRITICAL depending on the specialist's tool tier. ASI07 is the lowest-coverage cell in the cross-source matrix — surface this honestly as a known gap when no clean runtime control exists.

### Pattern: tool-output → agent → user reaches a browser as HTML

- Raw threat: `Information disclosure / Tampering — model output is rendered as HTML; a tool that returned attacker-controlled markup leads to XSS in the user's browser.`
- IDs: **LLM02** (Insecure Output Handling) + **ASI05** (Unexpected Code Execution if the rendering surface evaluates).
- NIST area: Information Security.
- Severity: HIGH for any user-facing surface.

### Pattern: cost / token runaway

- Raw threat: `Denial of service — agent loops on a partial-failure path; downstream cost grows unbounded.`
- IDs: **LLM04** (Model Denial of Service). ASI side: `(operational)` — the Agentic Top 10 does not have a top-level entry; cite as an operational concern, not as `(no canonical mapping)`.
- NIST area: `(operational)`.
- Severity: scales with budget exposure.

### Pattern: hallucinated fact reaches user as authoritative

- Raw threat: `Repudiation / trust — agent presents a confabulated fact as authoritative; user acts on it.`
- IDs: **LLM09** (Overreliance) + **ASI09** (Human-Agent Trust Exploitation). STRIDE: `(no canonical STRIDE category)` — this is content / value, not a security primitive.
- NIST area: Confabulation.
- Severity: MEDIUM in an A0 agent that explicitly says "draft"; HIGH where a downstream system trusts the agent's assertions without review.

## How to write the row in the artifact

Each threat row in the artifact's `## STRIDE` section follows the schema in `references/templates/threat-model.md`. The cross-mapping fields look like this:

```yaml
- threat_id: T-007
  stride: [T, E]
  element: P-1 (agent reasoning loop) consuming DF-3 (model output)
  raw_threat: |
    Tool output is concatenated into the agent's next-step prompt without an
    untrusted-content delimiter. An attacker controlling any tool's output can
    rewrite the agent's effective goal.
  owasp_llm: [LLM01, LLM07]
  owasp_agentic: [ASI01, ASI06]
  mitre_atlas: [AML.T0051]              # optional; cite by ID
  nist_600_1: [Information Integrity, Information Security]
  severity: HIGH
  notes: |
    Promote to CRITICAL if the resulting agent action is irreversible (T4 / T5).
```

The `severity` value uses the calibration in this file's "Common patterns" table, not a project-specific scale. CRITICAL / HIGH / MEDIUM / LOW.

## Honesty rules

- If a threat does not fit a framework cell, mark it `(no canonical mapping)` rather than picking the closest ID.
- If the framework's coverage is itself a gap (ASI07 / inter-agent comms — no clean runtime control), mark it so. Surface gaps; do not paper over them.
- If you are citing a MITRE ATLAS technique, link to its ATLAS page in the artifact. ATLAS evolves; locking the link is the only defense against drift.
- The OWASP Agentic Top 10 is recent. If a row in the cross-source matrix has a `(gap)` marker, copy that marker into the artifact rather than guessing.
