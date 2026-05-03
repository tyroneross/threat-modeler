---
name: threat-modeler
description: Produce a threat-model artifact for an agentic system or any risk-surface change. Walks the author from risk signal (new tool / MCP / LLM call / persistent memory / auth boundary / external API / user-data handling) to a finished markdown threat model with assets, actors, data-flow description, STRIDE decomposition, OWASP LLM/Agentic cross-map, mitigations, residual risk, and a decision log. Output satisfies build-loop's plan-verify rule 10 (`risk-surface-change-without-threat-model`). Activates when the user asks for a threat model, a security review of a design, OWASP/ASI mapping for a feature, or a "is this risk-surface change covered" check; also activates implicitly when a plan introduces any risk-surface signal without an existing threat-model artifact.
author: Tyrone Ross
version: 0.1.0
tags: [threat-model, security, stride, owasp, asi, llm, agentic, risk-surface, build-loop, plan-verify, security-review]
category: developer-tools
difficulty: intermediate
metadata:
  priority: 6
  pathPatterns:
    - '**/threat-model*.md'
    - '**/threat_model*.md'
    - '**/security-review*.md'
    - '**/security_review*.md'
    - '**/.build-loop/plan*.md'
    - '**/plan/threat-models/**'
    - '**/.threat-modeler/**'
  importPatterns:
    - '@modelcontextprotocol/*'
    - 'langgraph'
    - '@langchain/*'
    - 'langchain'
    - 'deepagents'
    - 'claude_agent_sdk'
    - 'claude-agent-sdk'
    - 'pydantic_ai'
    - 'pydantic-ai'
    - 'crewai'
    - 'autogen'
  bashPatterns:
    - '\bplan_verify\.py\b'
    - '\bbuild-loop:plan-verify\b'
    - '\brisk-surface-change-without-threat-model\b'
  promptSignals:
    phrases:
      - "threat model"
      - "threat-model"
      - "threat modelling"
      - "threat modeling"
      - "security review of this design"
      - "security review for this feature"
      - "stride analysis"
      - "stride decomposition"
      - "owasp mapping"
      - "asi mapping"
      - "owasp llm top 10"
      - "owasp agentic top 10"
      - "is this safe to build"
      - "what could go wrong with this"
      - "abuse cases"
      - "attacker personas"
      - "data flow diagram"
      - "data-flow diagram"
      - "trust boundary"
      - "trust boundaries"
      - "risk surface"
      - "risk-surface change"
      - "plan-verify rule 10"
      - "risk-surface-change-without-threat-model"
      - "security artifact"
      - "security artefact"
      - "residual risk"
      - "mitigation matrix"
      - "threat-model: not-applicable"
    allOf:
      - [threat, model]
      - [security, review]
      - [stride, analysis]
      - [owasp, mapping]
      - [risk, surface]
      - [trust, boundary]
      - [attacker, persona]
      - [residual, risk]
      - [data, flow, diagram]
    anyOf:
      - "stride"
      - "owasp"
      - "asi01"
      - "asi02"
      - "asi03"
      - "asi04"
      - "asi05"
      - "asi06"
      - "asi07"
      - "asi08"
      - "asi09"
      - "asi10"
      - "llm01"
      - "llm02"
      - "llm05"
      - "llm06"
      - "llm07"
      - "llm08"
      - "abuse case"
      - "misuse case"
      - "kill chain"
    noneOf: []
    minScore: 6
---

# Threat Modeler

Cross-LLM skill (Claude Code, Codex). Frontmatter `metadata` block above is consumed by Codex for auto-triggering on file paths, imports, shell commands, and prompt signals; runtimes that don't read it ignore it without harm.

## Problem

A plan introduces a new tool, MCP server, LLM call, persistent memory, auth boundary, external API, or new class of user data. Build-loop's `plan-verify` rule 10 (`risk-surface-change-without-threat-model`) blocks the plan until a threat-model artifact is cited. Most authors at that point face a blank page: STRIDE was last touched in a textbook, OWASP LLM and Agentic Top 10 IDs are remembered partially, and the plan is overdue. They write a stub or skip the rule.

This skill removes the blank page. It walks the author through a five-step workshop that ends in a finished, greppable, plan-verify-compatible threat-model artifact — STRIDE-style decomposition cross-mapped to OWASP LLM, OWASP Agentic, MITRE ATLAS, and NIST AI 600-1, with mitigations and residual risk. Lightweight by design. Not a heavyweight enterprise framework.

## What this skill does and does not do

**Does:**
- Produce a single markdown threat-model artifact per risk-surface change.
- Enumerate assets, actors, attacker personas, trust boundaries, and data flows in plain markdown (mermaid optional).
- Apply STRIDE per element of the data flow.
- Cross-map every threat to OWASP LLM Top 10 (LLM01–LLM10), OWASP Agentic Top 10 (ASI01–ASI10), MITRE ATLAS technique IDs, and NIST AI 600-1 risk areas.
- Produce a mitigation table with owner, status, and residual risk per row.
- Log decisions, assumptions, and out-of-scope items.
- Satisfy build-loop `plan-verify` rule 10 by default — the artifact filename, headers, and an explicit `threat_model:` YAML key all match the rule's regex.
- Provide a `not-applicable` template for the explicit escape hatch when the plan genuinely does not touch the risk surface.

**Does not:**
- Replace a runtime defense layer (DefenseClaw, NeMo Guardrails, Llama Guard, your own gateway). This is build-time descriptive output, not runtime enforcement.
- Generate adversarial test corpora or red-team playbooks.
- Map to compliance regimes beyond OWASP / ATLAS / NIST surfacing (no SOC 2, no ISO 42001, no EU AI Act in v0.1).
- Auto-generate runtime config. The output is a markdown artifact for human review and downstream tooling.
- Replace `build-loop:security-methodology` (the canon) or the build-loop `security-reviewer` agent (the grader). This skill is the **author-side workshop** that produces an input for both of those.

## Trigger conditions

Activate when any of the following hold:

- The user asks for a threat model, security review of a design, STRIDE analysis, OWASP/ASI mapping, or "what could go wrong with this".
- A `build-loop:plan-verify` run flagged `risk-surface-change-without-threat-model` (BLOCKER) and the author needs to produce the cited artifact.
- A plan or design doc introduces a risk-surface signal: new tool / MCP / LLM call / prompt template / persistent memory / vector store / auth or permission boundary / external API / new class of user data.
- The user is editing a file under `plan/threat-models/` or any path matching `**/threat-model*.md` or `**/security-review*.md`.
- The user is reviewing an existing agent-builder-style artifact pack (`tool-contract.md`, `agent-manifest.md`, `system-boundary.md`, `guardrail.md`) and asks "is this safe to ship".

## Default posture

1. **Match weight to surface.** A T1 read-only tool added to a draft-only (A0) agent gets a one-page artifact. A T4/T5 tool on a controlled-production (A3) agent gets the full template. Both are valid threat models; the not-applicable template is for the genuine no-risk-surface case, not for laziness.
2. **Cross-reference, don't re-author.** OWASP LLM/Agentic, ATLAS, NIST 600-1 are canonical at their source. This skill cites IDs and points at the canon (`build-loop:security-methodology` if installed; the references in the same canon's research file otherwise). Re-authoring the taxonomy is out of scope.
3. **Greppable beats pretty.** Predictable headers, predictable threat IDs (`T-001`, `T-002` …), predictable mitigation IDs (`M-001` …), predictable status values (`mitigated | accepted | open | transferred`). The artifact is built to be machine-readable by `plan-verify`, `security-reviewer`, and downstream automation.
4. **Honest residual risk.** Every threat ends with a residual-risk line. "Mitigated" is a claim that needs evidence, not a checkbox. Open and accepted residual risk is logged, not hidden.
5. **Decision log over comment thread.** Every assumption, every "out of scope", every "we'll revisit" lands in the decision log section. The artifact is a contract, not a conversation.

## Step 0 — Confirm the trigger

Before opening the workshop, make sure a threat model is what the situation needs.

- If the plan does not touch the risk surface (no new tool / LLM call / memory / auth / external API / user-data class), use `references/templates/threat-model-not-applicable.md`. State the rationale, cite the affected files, declare `threat-model: not-applicable: <reason>` so plan-verify rule 10 sees it. Stop.
- If the change does touch the risk surface, proceed to Step 1.

## Step 1 — Frame the system

Use `references/methodology/03-asset-and-actor-enumeration.md`.

Output: a short prose paragraph naming the system, its mission, its actors (users, operators, automated callers, downstream agents), the assets it protects (data, tokens, model weights, audit trail, user trust), and the trust boundaries between them. Keep it under 400 words. If an agent-builder `system-boundary.md` already exists for the system, cite it; do not duplicate.

## Step 2 — Describe the data flow

Use `references/methodology/04-data-flow-diagrams.md`.

Output: a labeled markdown description of the data flow — sources, sinks, processes, stores, trust-boundary crossings. Mermaid is optional; ASCII or numbered lists are fine. The goal is naming every place where data crosses a boundary, not a diagram beauty contest.

## Step 3 — Apply STRIDE per element

Use `references/methodology/02-stride-overview.md`.

Output: for each process, store, and data-flow trust-boundary crossing identified in Step 2, walk through STRIDE (Spoofing, Tampering, Repudiation, Information disclosure, Denial of service, Elevation of privilege). Discard categories that don't apply with one-line rationale; keep the rest as raw threats.

## Step 4 — Cross-map to OWASP LLM, OWASP Agentic, ATLAS, NIST

Use `references/methodology/05-mapping-to-owasp-asi.md`.

Output: each raw STRIDE threat from Step 3 is assigned a stable threat ID (`T-001`, `T-002`, …) and tagged with its OWASP LLM IDs (`LLM01`–`LLM10`), OWASP Agentic IDs (`ASI01`–`ASI10`), zero or more MITRE ATLAS technique IDs, and zero or more NIST AI 600-1 risk areas. Threats that have no clean cross-map keep `(no canonical mapping)` rather than be force-fitted.

## Step 5 — Mitigation, residual risk, decision log

Use `references/methodology/06-mitigation-and-residual-risk.md` and `references/methodology/07-decision-log-and-versioning.md`.

Output: a mitigation table with one row per threat, a residual-risk paragraph, and a decision log capturing assumptions, deferred items, and reviewer questions. The mitigation table uses the predictable schema (`M-NNN` IDs, status enum, owner, evidence link, residual risk) so downstream tooling can parse it.

## Workshop output

Apply the structure of `references/templates/threat-model.md` to produce the final artifact. The template is the source of truth for the artifact shape; this SKILL.md walks the author *to* the template, the template defines what the artifact *is*.

The artifact's filename should follow `<feature-or-component>-threat-model.md` (or `threat-model-<feature>.md` — both match plan-verify rule 10's regex `threat[_\s-]?model`). Drop it under `plan/threat-models/` in the consuming repo, or wherever the project's plan storage convention puts it.

## How a plan cites this artifact

Build-loop `plan-verify` rule 10 scans the entire plan markdown for any of: `threat-model`, `threat_model`, `threat model`, `security-review`, `security_review`, `owasp`, `asi01`–`asi10`, `llm01`–`llm10`, `security-methodology`, `security-reviewer`, or the explicit declaration `threat-model: not-applicable: <reason>`. **Any one match anywhere in the plan satisfies the rule for the whole document.**

Recommended cite formats inside the plan:

```markdown
Threat model: see [plan/threat-models/<feature>-threat-model.md](plan/threat-models/<feature>-threat-model.md).
```

Or, when the change does not surface risk:

```markdown
threat-model: not-applicable: pure visual change to an internal-only admin
panel; no new tools, no new LLM call, no new persistent memory, no auth or
permission change, no external API, no new user-data class. See
plan/threat-models/admin-panel-cleanup-not-applicable.md for the rationale.
```

The not-applicable form is the canonical escape hatch. Use it honestly. Do not use it to dodge the rule when the change actually does touch the risk surface.

## Operating rules

- Convert "this feels risky" into a stable threat ID, a cross-mapped category, and a named mitigation. No vague gut calls in the artifact.
- Push back on threat lists that are missing residual-risk lines. "Mitigated" without evidence is a claim, not a control.
- Cite OWASP, ATLAS, NIST IDs by their exact short forms (`LLM01`, `ASI03`, `AML.T0019`, `NIST:Information Integrity`). The IDs are load-bearing — `plan-verify` rule 10 and downstream tooling key off them.
- Keep the artifact small for small changes. A risk-surface change to a single internal tool does not need a 30-page threat model; a 1–2 page artifact covering the relevant STRIDE elements is enough. The not-applicable template is for the case where there is genuinely no risk-surface change.
- When the agent-builder skill is present in the repo, cross-reference its templates by absolute path (`/Users/<you>/dev/git-folder/agent-builder/plugin/references/templates/agentic-handoff/system-boundary.md` etc.) rather than re-authoring. If agent-builder is absent, the canonical research file at `~/dev/research/topics/product-dev/product-dev.agentic-systems-template-pack-addendum-v2.md` carries the same content.
- When `build-loop:security-methodology` is present, cite IDs against that canon's `cross-source-matrix.md`. Otherwise cite the upstream OWASP / ATLAS / NIST source URLs documented in the methodology files.

## Output contract

The finished artifact (whether produced via the workshop or the not-applicable template) must include, at minimum:

- A YAML front-block with `threat_model:` (key match for plan-verify rule 10), the system or feature name, version, author, date, and status.
- An assets and actors section.
- A data-flow description.
- A STRIDE-decomposed threat list with stable threat IDs and cross-source mappings.
- A mitigation table with stable mitigation IDs, owner, status, evidence, and residual risk.
- A decision log.
- A "what would change this artifact" section naming the future events that would invalidate it.

The template enforces all of these by section. Do not delete sections; mark them `n/a — <reason>` if a section truly does not apply.

## Final check before responding

- Did you produce a single markdown artifact, not a conversation transcript or a slide deck?
- Did every raw STRIDE threat get a stable `T-NNN` ID and at least one OWASP / ATLAS / NIST mapping (or an honest `(no canonical mapping)`)?
- Did every threat get a mitigation row with owner, status, evidence, and residual risk?
- Did you cite OWASP / ATLAS / NIST IDs by their exact short forms?
- Did the artifact's filename or YAML front-block match plan-verify rule 10's regex?
- If the case was genuinely no-risk-surface-change, did you use the `threat-model-not-applicable.md` template and declare `threat-model: not-applicable: <reason>` rather than producing a stub?
