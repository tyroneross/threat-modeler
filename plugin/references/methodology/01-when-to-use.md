# When To Use This Skill

Read this file first. It tells you whether a threat model is the right output for the situation in front of you, and how heavy the artifact should be.

## Use this skill when

- The plan introduces a new tool, MCP server, plugin, or skill.
- The plan introduces a new LLM call or modifies a prompt template that ships in production.
- The plan introduces persistent memory or a vector store.
- The plan changes an authentication, authorization, identity, or permission boundary.
- The plan introduces a new external API call.
- The plan introduces handling of a new class of user data — PII, financial, health, credentials, regulated records.
- `build-loop:plan-verify` flagged `risk-surface-change-without-threat-model` (BLOCKER) and the author needs to produce the cited artifact.
- The user asks for STRIDE analysis, OWASP / ASI mapping, "what could go wrong", or "is this safe to ship".
- The user is reviewing an existing agent-builder-style artifact pack (`tool-contract.md`, `agent-manifest.md`, `system-boundary.md`, `guardrail.md`) and needs the security companion.

These are the same risk-surface signals that flip `triggers.riskSurfaceChange: true` in build-loop Phase 1 Assess. The two trigger lists are intentionally aligned.

## Do not use this skill when

- The change is a pure refactor with no new tool, no new LLM call, no new memory, no new auth path, no new external API, no new user-data class. Use `references/templates/threat-model-not-applicable.md` to produce the explicit escape-hatch declaration that `plan-verify` rule 10 looks for. Stop.
- The change is a visual-only edit to an internal admin panel, a copy fix, a dependency bump that does not introduce a new outbound call, a typo fix in a prompt, a documentation edit. Same — use the not-applicable template.
- The work is a runtime defense layer (DefenseClaw, NeMo Guardrails, Llama Guard config, OPA policy). Threat-modeler is build-time descriptive output for human review; runtime enforcement lives in those tools.
- The work is a red-team playbook, an adversarial test corpus, or a fuzzing harness. Out of scope; pair with project-specific red-team work.
- The work is a compliance audit against SOC 2, ISO 42001, the EU AI Act, or a sector regulator. Out of scope in v0.1; threat-modeler surfaces OWASP / ATLAS / NIST IDs, but compliance regimes layer additional requirements that this skill does not track.

## Match weight to surface

Not every artifact is the full template. The point of this skill is producing the *right-sized* artifact for the change.

| Surface | Recommended artifact weight |
|---|---|
| New T1 read-only tool on a draft-only (A0) agent. | 1-page artifact: STRIDE on the tool-input boundary only, single mitigation row, 2–3 line residual risk. |
| New T2 tool on a reversible-decisions (A1) agent. | 1–2 page artifact: STRIDE on the tool-input and the data-store-write boundaries, 2–4 mitigation rows. |
| New T3 tool on a bounded-execution (A2) agent. | Full template, every section. |
| New T4 / T5 tool on a controlled-production (A3) agent. | Full template plus an explicit human-checkpoint section in the decision log. |
| New persistent memory or vector store. | Full template. ASI06 (Memory and context poisoning) lives here; treat it like a T3 tool by default. |
| New auth or permission boundary. | Full template. ASI03 (Identity and Privilege Abuse) and OWASP Web A01 (Broken Access Control) both apply; cite both. |
| New external API call. | 1–2 page artifact at minimum, full template if the API receives or returns user data. ASI04 (Agentic Supply Chain) plus LLM05. |
| New class of user data being handled. | Full template. NIST Data Privacy applies; LLM06 (Sensitive Information Disclosure) applies. |
| Pure refactor, copy, visual, doc-only change. | `not-applicable` template only. |

The autonomy levels (`A0`–`A4`) and tool tiers (`T0`–`T5`) match `agent-builder/plugin/references/methodology/13-agentic-product-dev-synthesis.md` and `agent-builder/plugin/references/templates/agentic-handoff/tool-contract.md`. If the autonomy or tier is not yet decided, that itself is an out-of-scope finding for the threat model — note it in the decision log.

## What "the artifact" means

A threat model in this skill is one markdown file. Filename convention: `<feature>-threat-model.md` or `threat-model-<feature>.md`. Either form matches build-loop's `plan-verify` rule 10 regex (`threat[_\s-]?model`).

The artifact is built from `references/templates/threat-model.md`. The not-applicable rationale is built from `references/templates/threat-model-not-applicable.md`. The artifact is greppable by design — predictable headers (`## Assets`, `## Actors`, `## Data Flow`, `## STRIDE`, `## Mitigations`, `## Residual Risk`, `## Decision Log`), predictable threat IDs (`T-001`, `T-002`, ...), predictable mitigation IDs (`M-001`, `M-002`, ...), predictable status enum (`mitigated | accepted | open | transferred`).

## Plan-verify rule 10 contract

`build-loop:plan-verify` rule 10 (`risk-surface-change-without-threat-model`) is a doc-level BLOCKER: any one match anywhere in the plan satisfies the rule for the whole document. The rule's regex matches:

- `threat[_\s-]?model` (covers `threat-model`, `threat_model`, `threat model`)
- `security[_\s-]?review` (covers `security-review`, `security_review`, `security review`)
- `owasp`
- `asi\d+` (covers `asi01` through `asi99`)
- `llm0\d+` (covers `llm01` through `llm09`)
- `security-methodology`
- `security-reviewer`
- `threat-model:\s*not[\s-]?applicable`

The artifacts produced from the templates in this skill match `threat[_\s-]?model` (filename and YAML key), `owasp` (mapping section header), and `asi\d+` / `llm0\d+` (cross-mapping section). Any one of those is sufficient. Plans that cite the artifact by relative path automatically inherit a match.

## What this skill does not produce

- A diagram. Mermaid is welcome inside the artifact when the data flow benefits, but the artifact's primary form is structured markdown text. ASCII or numbered lists are equally acceptable.
- An exploit proof-of-concept.
- A vendor selection between DefenseClaw, NeMo Guardrails, and Llama Guard.
- A regulatory filing.
- An internal red-team report. Red-team work happens against this artifact, not inside it.
