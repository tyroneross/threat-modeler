---
description: Produce a threat-model artifact for a risk-surface change. Walks STRIDE → OWASP/ASI cross-mapping → mitigations → residual risk.
argument-hint: "[scope]  e.g. \"adding web_fetch tool\" or \"path/to/feature\""
---

# /threat-modeler

Run the threat-modeler workshop on the named scope. Produces a single markdown artifact that satisfies `build-loop:plan-verify` rule 10 (`risk-surface-change-without-threat-model`).

## What this command does

1. Reads `plugin/SKILL.md` and routes to the right artifact weight (full template vs. not-applicable) based on whether the scope touches a risk surface.
2. Walks the five-step workshop:
   - **Step 1 — Frame.** Assets, actors, attacker personas, trust boundaries (`references/methodology/03-asset-and-actor-enumeration.md`).
   - **Step 2 — Flow.** Data flow elements + flows + optional mermaid (`references/methodology/04-data-flow-diagrams.md`).
   - **Step 3 — STRIDE.** Per-element walk (`references/methodology/02-stride-overview.md`).
   - **Step 4 — Cross-map.** OWASP LLM, OWASP Agentic, MITRE ATLAS, NIST AI 600-1 IDs (`references/methodology/05-mapping-to-owasp-asi.md`).
   - **Step 5 — Mitigations + residual + decisions.** Predictable schema (`references/methodology/06-mitigation-and-residual-risk.md`, `07-decision-log-and-versioning.md`).
3. Writes the finished artifact using `references/templates/threat-model.md` (or `threat-model-not-applicable.md` for the genuine no-risk-surface case) to `plan/threat-models/<feature>-threat-model.md` (or wherever the consuming repo's plan storage lives).
4. Returns the absolute path of the artifact and a cite line ready to paste into the originating plan.

## Argument

The argument is a free-form description of the scope. Examples:

- `/threat-modeler "adding web_fetch tool to research assistant"`
- `/threat-modeler "new requirements_specialist agent + persistent memory + A2A handoff"`
- `/threat-modeler "visual-only refactor of admin settings page"` (will route to `threat-model-not-applicable.md`)
- `/threat-modeler plan/2026-05-02-add-web-fetch.md` (path argument; reads the plan and infers scope from its risk-surface signals)

If the argument is omitted, ask the user for the scope before doing anything else. Do not guess.

## Default behavior

- **Match weight to surface.** A T1 read-only tool on an A0 agent gets a 1-page artifact; a T4/T5 tool on an A3 agent gets the full template. See `references/methodology/01-when-to-use.md`.
- **Cross-reference rather than re-author.** When the consuming repo has agent-builder artifacts (`system-boundary.md`, `agent-manifest.md`, `tool-contract.md`, `flow-topology.md`), cite them by absolute path. Do not duplicate their contents.
- **Cite IDs by exact short form.** `LLM01`, `ASI03`, `AML.T0019`, `NIST:Information Integrity`. Tooling reads them literally.
- **Honest residual risk.** Every mitigation row ends with a residual-risk line. "Mitigated" without evidence is `open`, not `mitigated`.

## Output contract

The command's terminal output is brief:

```
[threat-modeler] artifact: <absolute path>
[threat-modeler] cite line for plan:
    Threat model: see [plan/threat-models/<feature>-threat-model.md](plan/threat-models/<feature>-threat-model.md).
[threat-modeler] residual risk summary: <one line — highest residual + acceptability>
```

For not-applicable runs:

```
[threat-modeler] not-applicable artifact: <absolute path>
[threat-modeler] cite line for plan:
    threat-model: not-applicable: <one-line rationale>
```

The artifact itself follows the schema in the relevant template; do not mutate the template structure when emitting.

## Plan-verify rule 10 contract

The artifact's filename (`threat-model` substring), YAML front-block (`threat_model:` key), and headers (`## STRIDE`, mapping to `owasp` and `asi\d+` / `llm0\d+`) all match `build-loop:plan-verify` rule 10's regex independently. Any one match anywhere in the plan satisfies the rule for the whole document. The cite line emitted above is sufficient to lift the BLOCKER.

## When to ask before running

If the scope is unclear — e.g., a path argument that points at a plan with no risk-surface signals at all, or a free-form description that is ambiguous about which feature is being added — ask one clarifying question before walking the workshop. Do not produce a stub artifact. The not-applicable template is the right output for "no risk surface change"; do not use the full template for a placeholder.

## Notes

- This command is the orchestrator surface for the workshop. The skill `plugin/SKILL.md` is the source of truth for routing, methodology files, and templates. Read SKILL.md before walking the workshop; do not re-derive the steps from this command file.
- The build-loop `security-reviewer` agent grades the resulting artifact in Phase 4 Review when build-loop is in use. Producing the artifact via this command makes the agent's job mechanical rather than interpretive.
