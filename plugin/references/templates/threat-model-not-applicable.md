# Threat Model — Not Applicable Template

> Use this template when the change genuinely does not touch the risk surface and a full threat model would be ceremony, not safety. The output of this template is a short markdown file whose body declares `threat-model: not-applicable: <reason>` — the explicit escape hatch that `build-loop:plan-verify` rule 10 looks for.
>
> **Honesty rule.** The not-applicable form is not a way to dodge rule 10 when the change actually does surface risk. If you are unsure, use the full `threat-model.md` template; the 1-page artifact-weight guidance in `references/methodology/01-when-to-use.md` keeps it cheap.

## When to use this template

A change qualifies for the not-applicable form **only** when **all** of the following are true:

- No new tool, MCP server, plugin, or skill.
- No new LLM call. No change to any prompt template that ships in production.
- No new persistent memory or vector store. No new agent-writable memory channel.
- No authentication, authorization, identity, or permission boundary change.
- No new external API call.
- No new class of user data being handled (no new PII, financial, health, credentials, regulated records).

If any one of those is false, use the full `threat-model.md` template instead. Match the artifact weight to the surface — most risk-surface changes only need a 1-page artifact, not the full template.

## Template

Copy the block below into `plan/threat-models/<feature>-not-applicable.md` (or wherever the project's plan storage lives). Filename should still contain `threat-model` so `plan-verify` rule 10's regex matches.

---

```yaml
---
threat_model:
  name: "<feature or change name>"
  version: "0.1.0"
  status: "approved"
  author: "<name>"
  reviewers: ["<name>"]
  date: "YYYY-MM-DD"
  scope_summary: |
    One sentence naming the change. Example: "Visual-only refactor of the
    admin panel's settings page; no behavior change."
  source_plan: "<path or URL to the plan / PR / design doc>"
  classification: "not-applicable"
---
```

# {{ threat_model.name }} — Threat Model: Not Applicable

**Declaration.** `threat-model: not-applicable: <one-line rationale>`

## What changed

One paragraph naming the change in concrete terms. Cite specific files, components, or PRs.

## Why no risk-surface change

A short bulleted list confirming each of the six rule-10 risk-surface signals is **not** triggered. Be specific; do not paraphrase the criteria back at the reader.

- **No new tool / MCP / plugin / skill.** Confirm by listing the components touched and noting that none are tool implementations.
- **No new LLM call. No prompt-template change in production.** Confirm by listing any prompt-related files touched (or stating "none touched").
- **No new persistent memory or vector store.** Confirm by listing the memory / store paths touched (or stating "none touched").
- **No auth / authz / identity / permission boundary change.** Confirm by listing the auth / permission files touched (or stating "none touched").
- **No new external API call.** Confirm by listing any networked code paths touched (or stating "none touched").
- **No new class of user data.** Confirm by listing any data-handling code touched and naming the data classes (or stating "none touched").

## Files touched

Bulleted list of file paths or a stable diff reference. Reviewers cross-check this against the plan / PR diff.

## Future events that would invalidate this declaration

Bulleted list. The declaration is durable, but only as long as the change stays in scope. List the specific events that would require a real threat model.

- A future change to this same component that adds a tool.
- A future change to this same component that adds a prompt or LLM call.
- A future change to this same component that adds an external API.
- A future change to this same component that starts handling a new user-data class.
- A future change that promotes the agent's autonomy level (A0 → A1+).

## Reviewer signoff

```yaml
signoff:
  - name: "<reviewer>"
    role: "<role>"
    date: "YYYY-MM-DD"
    decision: "approved"
```

---

**Plan-verify rule 10 cite-path note.** The literal phrase `threat-model: not-applicable` in the body of this file satisfies rule 10's escape-hatch regex. The plan citing this file inherits the satisfaction; recommended cite line in the plan markdown:

```markdown
threat-model: not-applicable: pure visual refactor of the admin settings
page, no risk-surface signals triggered. Rationale at
plan/threat-models/<feature>-not-applicable.md.
```
