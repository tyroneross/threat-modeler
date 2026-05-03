# threat-modeler

Lightweight workshop assistant that produces a threat-model artifact for an agentic system or any risk-surface change. Walks the author from risk signal (new tool / MCP / LLM call / persistent memory / auth boundary / external API / user-data handling) to a finished markdown artifact with assets, actors, data-flow description, STRIDE decomposition, OWASP LLM/Agentic + MITRE ATLAS + NIST AI 600-1 cross-mapping, mitigations, residual risk, and a decision log.

The output is built to satisfy build-loop's `plan-verify` rule 10 (`risk-surface-change-without-threat-model`) by default — the artifact filename, headers, and YAML front-block all match the rule's regex.

This is **build-time descriptive** output for human review and downstream tooling. It is not a runtime defense layer (DefenseClaw, NeMo Guardrails, Llama Guard remain that). It is also not a heavyweight enterprise threat-modeling framework — STRIDE-style decomposition with OWASP/ATLAS/NIST cross-mapping is intentionally the ceiling.

## Distribution model

threat-modeler ships as a single slim plugin. There is no companion workbench app, no Next.js dashboard, no generator scripts. Everything lives under `plugin/`.

- **RossLabs AI Toolkit marketplace plugin (when added):** the slim skill/instruction package. Installs via the marketplace and shows up inside Claude/Codex as a workshop assistant.
- **This GitHub repo:** the canonical source. Clone or fork to read the methodology files, templates, and worked examples without going through the marketplace.

## When it activates

Automatic triggers:

- The user asks for a threat model, security review of a design, STRIDE analysis, OWASP / ASI mapping, or "what could go wrong with this".
- A `build-loop:plan-verify` run flagged `risk-surface-change-without-threat-model` and the author needs to produce the cited artifact.
- A plan or design doc introduces a risk-surface signal: new tool / MCP / LLM call / prompt template / persistent memory / vector store / auth or permission boundary / external API / new user-data class.
- The user opens any file under `**/threat-model*.md`, `**/security-review*.md`, or `plan/threat-models/`.

See the `description` and `metadata.promptSignals` block in `plugin/SKILL.md` for the full trigger surface.

## Workshop in five steps

1. **Frame the system** — assets, actors, attacker personas, trust boundaries.
2. **Describe the data flow** — sources, sinks, processes, stores, boundary crossings.
3. **Apply STRIDE per element** — Spoofing / Tampering / Repudiation / Information disclosure / Denial of service / Elevation of privilege.
4. **Cross-map** — assign OWASP LLM (LLM01–LLM10), OWASP Agentic (ASI01–ASI10), MITRE ATLAS, NIST AI 600-1 IDs.
5. **Mitigation, residual risk, decision log** — predictable schema, predictable IDs, honest residuals.

The output is a single markdown file produced from `plugin/references/templates/threat-model.md`. When the change genuinely does not touch the risk surface, use `plugin/references/templates/threat-model-not-applicable.md` instead.

## Repo structure

```
threat-modeler/
├── README.md
├── LICENSE
└── plugin/                                      # slim skill — marketplace package
    ├── SKILL.md                                 # entry, trigger, router (cross-LLM)
    ├── .claude-plugin/plugin.json               # Claude Code marketplace manifest
    ├── .codex-plugin/plugin.json                # Codex manifest
    ├── commands/
    │   └── threat-modeler.md                    # /threat-modeler [scope] slash command
    ├── examples/
    │   ├── example-mcp-tool-agent-threat-model.md
    │   └── example-product-dev-orchestrator-threat-model.md
    └── references/
        ├── methodology/                         # 7 files — how to decide
        │   ├── 01-when-to-use.md
        │   ├── 02-stride-overview.md
        │   ├── 03-asset-and-actor-enumeration.md
        │   ├── 04-data-flow-diagrams.md
        │   ├── 05-mapping-to-owasp-asi.md
        │   ├── 06-mitigation-and-residual-risk.md
        │   └── 07-decision-log-and-versioning.md
        └── templates/
            ├── threat-model.md                  # the artifact template
            ├── threat-model-not-applicable.md   # explicit escape hatch
            └── attacker-personas.md             # generic personas
```

## Cross-LLM activation

`plugin/SKILL.md` is a single file that triggers in any host:

- **Claude Code, Claude Desktop, Claude API** — match against the natural-language `description` field.
- **Codex and compatible hosts** — match against the `metadata` frontmatter block (`pathPatterns`, `importPatterns`, `bashPatterns`, `promptSignals` with `minScore: 6`). Hosts that do not read that block ignore it without harm.

No variant files; one canonical SKILL.md serves every host.

## Install

**Slim plugin via the RossLabs marketplace** (when added):

```bash
/plugin marketplace add tyroneross/RossLabs-AI-Toolkit
/plugin install threat-modeler@RossLabs-AI-Toolkit
```

**As a standalone user skill** (any plugin host or bare Claude Code):

```bash
mkdir -p ~/.claude/skills/threat-modeler
rsync -a plugin/SKILL.md plugin/references plugin/examples plugin/commands \
  ~/.claude/skills/threat-modeler/
```

**Inside another plugin:** drop the contents of `plugin/` into that plugin's `skills/threat-modeler/` directory.

## Relationship to the build-loop security stack

| Tool | Relationship |
|---|---|
| `build-loop:plan-verify` rule 10 | The rule that creates demand for this skill. Threat-modeler output is the artifact that satisfies the rule. |
| `build-loop:security-methodology` | The canon (OWASP LLM, OWASP Agentic, MITRE ATLAS starter, NIST 600-1 mapping, cross-source matrix). Threat-modeler cites IDs *from* that canon when the canon is installed; falls back to the upstream OWASP / ATLAS / NIST URLs otherwise. |
| `build-loop` `security-reviewer` agent | Phase 4 Review grader. Reads the threat-model artifact this skill produces and grades it against the canon. |
| `agent-builder` templates (`system-boundary.md`, `tool-contract.md`, `guardrail.md`, `agent-manifest.md`) | Sibling artifacts. Threat-modeler cross-references these by absolute path when present, rather than re-authoring boundary or tool contracts. |

## Design posture

- Match weight to surface. A T1 read-only tool gets a 1-page artifact; a T4/T5 tool on a controlled-production agent gets the full template.
- Cross-reference rather than re-author the OWASP / ATLAS / NIST taxonomies. The IDs are the load-bearing artifact, not a private restatement.
- Greppable beats pretty. Predictable headers, predictable threat IDs (`T-NNN`), predictable mitigation IDs (`M-NNN`), predictable status enum values.
- Honest residual risk. Every threat ends with a residual-risk line. "Mitigated" without evidence is a claim, not a control.

## Source repos & inspiration

threat-modeler is a downstream artifact of three open-source projects. What was leveraged from each:

### [tyroneross/agent-builder](https://github.com/tyroneross/agent-builder) — structural template

threat-modeler's plugin shape mirrors agent-builder. Specifically:

- **Slim dual-LLM plugin layout** — `plugin/SKILL.md` with Claude `description` + Codex `metadata` frontmatter, sibling `.claude-plugin/plugin.json` and `.codex-plugin/plugin.json`, `references/methodology/` + `references/templates/` + `examples/` + `commands/` folders. Marketplace-portable, no symlinks crossing plugin boundaries.
- **Distribution model** — slim plugin only; no companion workbench. agent-builder's clone-only workbench was intentionally not replicated.
- **Methodology tone** — direct, specific, no filler. Numbered files (`01-when-to-use.md` … `07-decision-log-and-versioning.md`) match agent-builder's `01-…11-…` pattern.
- **Cross-reference pattern** — absolute paths to canon (rather than relative paths or symlinks) so the plugin remains marketplace-portable while still pointing at sibling installations.
- **Upstream artifact contracts** — agent-builder's templates (`system-boundary.md`, `tool-contract.md`, `agent-manifest.md`, `guardrail.md`, `flow-topology.md`, `role-card.md`) are the sibling artifacts threat-modeler **cross-references** instead of re-authoring. A threat-model artifact reads cleanly *because* the underlying boundary, tool, and manifest contracts already exist upstream.
- **Autonomy ladder (A0–A4)** from `methodology/13-agentic-product-dev-synthesis.md` — used to scope artifact weight (a T1 read-only tool gets a 1-page artifact; a T4/T5 tool at A2+ gets the full template).
- **MIT license** — matched.

### [tyroneross/build-loop](https://github.com/tyroneross/build-loop) — integration target

threat-modeler exists to satisfy a contract build-loop creates. What was leveraged:

- **`plan-verify` rule 10 (`risk-surface-change-without-threat-model`)** — the rule that *demands* a threat-model artifact in the first place. Threat-modeler's filename convention (`<feature>-threat-model.md`), header schema (`## STRIDE`, `## Mitigations`, etc.), and YAML front-block (`threat_model:` key) were authored to satisfy rule 10's regex three independent ways.
- **`security-methodology` skill** — the canonical OWASP × OWASP-Agentic × MITRE-ATLAS × NIST-600-1 cross-source matrix. Threat-modeler cites IDs *from* build-loop's canon when installed; falls back to upstream OWASP/ATLAS/NIST URLs otherwise.
- **`security-reviewer` agent contract** — the Phase 4-A grader. Threat-modeler's section names, threat ID schema (`T-NNN`), mitigation ID schema (`M-NNN`), decision-log ID schema (`D-NNN`), and status enum (`mitigated | accepted | open | transferred`) were chosen to be machine-parseable by the reviewer's grading prompt.
- **Severity vocabulary (`CRITICAL / HIGH / MEDIUM / LOW`) and permission-tier vocabulary (`T0–T5`)** — adopted verbatim from build-loop, not redefined.

### [cisco-ai-defense/defenseclaw](https://github.com/cisco-ai-defense/defenseclaw) — runtime context

threat-modeler is **build-time descriptive** and complementary to DefenseClaw's **runtime enforcement**. What it leverages:

- **Govern / Inspect / Prove three-pillar mental model** — DefenseClaw's framing of agentic security as admission control + runtime traffic inspection + audit evidence shaped how threat-modeler positions itself: a design-time artifact whose mitigations get *enforced* by Inspect-layer tools (DefenseClaw, NeMo Guardrails, Llama Guard, Galileo Agent Control) at runtime. The artifact does not replace the runtime layer.
- **"Mitigation without evidence is a claim" posture** — DefenseClaw's architecture clarifies that a threat-model is a description, not a control. Threat-modeler's mitigation rows therefore require an `evidence` column (PR / file / config / test) and a `status` (`mitigated | accepted | open | transferred`) — never just an assertion.
- **Detection-pattern influence** — DefenseClaw's regex rule packs (prompt-injection signatures, secrets patterns, exfiltration commands) implicitly informed the detection-pattern rows cited in `references/methodology/05-mapping-to-owasp-asi.md`.

### Methodology lineage (not repos, but worth naming)

- **STRIDE** — Microsoft's foundational threat-modeling framework. `references/methodology/02-stride-overview.md` adapts STRIDE for agentic systems; the threat decomposition table uses STRIDE categories as columns.
- **Cisco/Zenity Labs collaboration on MITRE ATLAS** (Oct 2025) — added 14 agent-specific techniques (autonomy/delegation/runtime attack paths). Influenced the "starter" technique set in `05-mapping-to-owasp-asi.md`.

## Sources

- **OWASP Top 10 for LLM Applications (v1.1)** — `https://genai.owasp.org/resource/llm-ai-security-and-governance-checklist/`.
- **OWASP Top 10 for Agentic Applications (2026)** — `https://genai.owasp.org/resource/owasp-top-10-for-agentic-applications-for-2026/`.
- **MITRE ATLAS** — `https://atlas.mitre.org/`.
- **NIST AI 600-1** — `https://nvlpubs.nist.gov/nistpubs/ai/NIST.AI.600-1.pdf`.
- **build-loop security-methodology cross-source matrix** — `~/dev/git-folder/build-loop/skills/security-methodology/references/cross-source-matrix.md`.
- **Canonical security research file** — `~/dev/research/topics/product-dev/product-dev.agentic-systems-security-references.md`.
- **agent-builder methodology** — `~/dev/git-folder/agent-builder/plugin/references/methodology/13-agentic-product-dev-synthesis.md`.

## License

MIT.
