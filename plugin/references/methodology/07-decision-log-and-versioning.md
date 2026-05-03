# Decision Log And Versioning

Step 5b of the workshop. Every assumption, scoping decision, deferred item, accepted residual, and reviewer-flagged question lands in the decision log. The threat model artifact is a contract, not a conversation; the decision log is what makes it auditable across versions.

## Why a decision log

Threats and mitigations capture the system's state. The decision log captures **why** that state was chosen — which alternatives were considered, which were rejected, which were deferred, which assumptions are load-bearing. Without it, the next reviewer six months from now has to rebuild the reasoning from scratch and tends to silently change the answer.

A decision log entry is durable. It survives across artifact versions. It is also the canonical home for `accepted` mitigations' rationale and for "out of scope" markers.

## Entry shape

Every entry has the same fields:

```yaml
- decision_id: D-001
  date: 2026-05-02
  type: assumption | decision | deferral | acceptance | gap
  subject: |
    Short one-line statement of what is being decided / assumed / deferred.
  context: |
    A few sentences naming the alternatives, why this option was chosen, and
    what would change it.
  links:
    - threats: [T-007, T-009]
    - mitigations: [M-005]
    - external: ["https://atlas.mitre.org/techniques/AML.T0051"]
  owner: Tyrone Ross
  status: active | superseded | invalidated
  superseded_by: D-NNN          # only when status == superseded
```

`decision_id` is stable across versions. Entries are appended; superseded entries stay in the log with `status: superseded` and a pointer to the entry that replaced them.

## Type vocabulary

- **`assumption`** — something the artifact is built on top of. Example: "The model provider's tool-calling JSON schema is enforced server-side." If the assumption breaks, the entry's `links.threats` shows what to revisit.
- **`decision`** — a non-trivial choice between alternatives. Example: "Sandbox the `run-tests` tool with `firejail` rather than a full container; rationale: latency, single-user system." Includes alternatives considered.
- **`deferral`** — work known to be needed but not done in this artifact's scope. Example: "A2A identity propagation deferred until orchestrator + specialist split lands; covered by D-005." `links.threats` names the threat that survives.
- **`acceptance`** — explicit acceptance of a residual risk. Owner is the human who accepts it. Pairs with the `accepted` mitigation row. Acceptance entries are the most-audited entries in the log; do not take them lightly.
- **`gap`** — a known gap in framework coverage or in evidence. Example: "ASI07 (inter-agent comms) — no clean runtime control yet; surface as known unknown rather than paper over."

## What goes in the decision log vs the threat list

The threat list is the *what*. The decision log is the *why* and the *what-if*.

- "We considered using a regex prompt-injection scrubber and decided against it because [...]" → decision log.
- "User input is concatenated into the system prompt." → threat list.
- "We assume the orchestrator's signed identity claim is enforced server-side." → decision log (assumption).
- "Cross-tenant access is possible if the signed claim is replayed within the 5-minute key validity window." → mitigation residual-risk line, **and** decision log (acceptance) if the 5-minute window was a chosen trade-off.

When a threat row's `notes` column starts to grow into a paragraph of rationale, that rationale belongs in the decision log; the row should `link` to the entry instead.

## "What would change this artifact"

The artifact's tail section, populated from this file's guidance, names the events that would require regeneration. This is the artifact's review trigger surface — without it, threat models go stale invisibly.

Common entries:

- The agent gains a new tool above tier T2.
- The agent's autonomy level changes (e.g., A0 → A2).
- A new external API is added.
- A new class of user data starts being handled.
- A new model provider is added (supply chain change).
- A new persistent memory store is added or an existing one starts accepting agent-written content.
- A new A2A peer is added to the system.
- The orchestrator pattern changes (single-agent → multi-agent or vice versa).
- A regulatory regime starts applying.
- An OWASP Top 10 (LLM or Agentic) revision changes ID semantics.
- A MITRE ATLAS update introduces a technique that maps to an existing threat.

Each event is one line. The author commits to revisiting the artifact when any of them fires; the build-loop `security-reviewer` agent and the orchestrator's Phase 1 Assess flag use this list to decide when to re-run the workshop.

## Versioning

The artifact's YAML front-block carries a `version` field. Bump rules:

- **Patch (0.1.0 → 0.1.1)** — typo, clarification, link fix. No threat or mitigation rows added or removed.
- **Minor (0.1.0 → 0.2.0)** — new threat, new mitigation, new decision-log entry, status change on an existing row.
- **Major (0.1.0 → 1.0.0)** — system mission, scope, or architecture changed enough that the artifact is structurally different. Old artifact stays in place as a historical record; new artifact carries forward the still-applicable threat IDs to keep traceability.

Threat IDs (`T-NNN`) and mitigation IDs (`M-NNN`) are stable across versions. A retired threat keeps its ID and gets a `status: invalidated` decision-log entry pointing at the version that retired it. Reusing a retired ID for a new threat is forbidden — it confuses the audit trail.

## Reviewer questions

The decision log is the right home for unresolved reviewer questions. Format:

```yaml
- decision_id: D-008
  date: 2026-05-02
  type: gap
  subject: |
    Reviewer (jdoe) asked: does the cache TTL of 24h create a window where
    cross-PR data could leak through cached file contents?
  context: |
    Open. Needs investigation. Threat T-014 added pending answer; mitigation
    M-009 status is `open`.
  links:
    - threats: [T-014]
    - mitigations: [M-009]
  owner: Tyrone Ross
  status: active
```

Rather than leaving the question in a comment thread on a PR, lift it into the artifact. The artifact is the durable record; the comment thread evaporates.

## Cross-reference, do not duplicate

If the system has an agent-builder `assumption-log.md` (template 8 in `agentic-handoff/`), use that for assumptions and `link` to it from threat-modeler decision-log entries rather than duplicating. Same for `agent-adr.md` (template 13). Threat-modeler's decision log is **scoped to the threat model**; broader system decisions live in the system's own ADRs and assumption log.

## Output shape for step 5b

The artifact's `## Decision Log` section is a list of entries in the YAML shape above (or markdown sub-headings, equivalently). The artifact's `## What Would Change This Artifact` tail section is a bulleted list populated from the guidance above, scoped to the system at hand.
