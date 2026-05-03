# Asset And Actor Enumeration

Step 1 of the workshop. Output: a short prose paragraph (≤ 400 words) that names the system, its mission, the actors that interact with it, the assets it protects, and the trust boundaries between actors and assets. This sets the scope for everything downstream — STRIDE walks against unnamed actors waste time.

## What you are listing

### Assets

What attackers want and what defenders protect. Pull these from the system itself, not from a generic checklist.

- **Data assets** — user data (PII, financial, health, credentials), tenant data, training data, prompt templates, system prompts, conversation history, memory contents, vector embeddings.
- **Capability assets** — tool credentials (API keys, OAuth tokens, service-account keys), model API keys, secrets, signing keys.
- **Trust assets** — the audit trail, the review queue, user-visible confidence scores, the agent's reputation with the user, the brand.
- **Availability assets** — request budget, model token budget, rate-limit headroom, downstream API quota.
- **Compliance assets** — regulated-data classifications, data-residency commitments, retention SLAs.

For agentic systems specifically, name **the agent's effective prompt** as a first-class asset. It is what attackers most commonly target through prompt injection, and it is what defenders most commonly forget to protect.

### Actors

Every entity that touches the system. For an agentic system this almost always includes more than the user.

- **End users** — the human the system serves. Note multi-tenancy: are users isolated from each other, or do they share resources?
- **Operators / maintainers** — humans who deploy, configure, debug, or review the system. Insider risk lives here.
- **Automated callers** — CI, cron jobs, webhooks, upstream services that invoke the agent.
- **Tools the agent calls** — every tool implementation is an actor for trust purposes. A tool that returns natural language is an actor that can speak into the agent's prompt.
- **Downstream agents** — A2A peers, sub-agents in a workflow, specialist agents in an orchestrator-worker pattern.
- **Model providers** — every external LLM provider (OpenAI, Anthropic, Google, etc.) plus any local model runtime. They are trusted differently from the user and from each other.
- **Attackers** — see `references/templates/attacker-personas.md` for the canonical generic personas.

### Trust boundaries

A trust boundary is any place where data or control crosses between actors with different trust assumptions. List them explicitly.

Common agentic trust boundaries:

- User → agent (the input boundary; LLM01 / ASI01 lives here when user input becomes prompt context).
- Agent → tool (the tool-call boundary; LLM07, LLM08, ASI02, ASI03 live here).
- Tool → agent (the tool-output boundary; LLM01 / ASI01 / ASI06 live here when tool output becomes prompt context — indirect prompt injection).
- Agent → external API (the egress boundary; LLM05, LLM06, ASI04 live here).
- Agent → memory store (the persistence boundary; ASI06 lives here).
- Memory store → agent (the recall boundary; ASI06 again, plus any tampering risk on the store).
- Agent → downstream agent (the A2A boundary; ASI07 lives here, currently the lowest-coverage cell of the cross-source matrix).
- User → operator (the support / escalation boundary).
- Operator → system config (the deploy / config boundary).

Drawing a literal box-and-arrow diagram is optional. Naming the boundaries in prose is required — the STRIDE walk in step 3 keys off of them.

## What "system mission" means

One sentence. Not marketing. The point of the system from a security perspective is to do *what specifically*, for *whom*, with *which side effects allowed*. Example:

> **System mission.** Read a pull request diff for the maintainer's own GitHub repos and post exactly one consolidated review comment per PR event. The system may read the diff, the touched files, and run two allowlisted scripts (`./scripts/lint.sh`, `./scripts/test.sh`); it may not merge, approve, push commits, or run other commands.

The "may not" half of the sentence is load-bearing — it tells the threat model what excessive-agency drift looks like.

## Cross-reference, do not duplicate

If the system has an agent-builder `system-boundary.md` artifact already, cite it by absolute path and summarize its `actions_that_change_the_world`, `external_tools`, `data_sources`, and `human_roles` here in one paragraph. Do not duplicate the YAML. Same rule for `agent-manifest.md`.

If `system-boundary.md` does not yet exist, naming the actors and boundaries in this section is the de facto boundary — and the threat model surfaces "no formal system_boundary artifact" as a finding (`T-NNN`, mapped to ASI03 and to the agent-builder template family).

## Common omissions to check for

Authors miss the same actors and assets repeatedly. Walk this short checklist before considering step 1 complete:

- Did you name **every tool** the agent can call as an actor?
- Did you name **the operator** as an actor (insider risk)?
- Did you name **the system prompt** as an asset?
- Did you name **the memory / vector store** as both an asset and a boundary?
- Did you name **cross-tenant boundaries** if the system is multi-tenant?
- Did you name **the audit trail** as a trust asset (its absence is repudiation)?
- Did you name **token / cost budgets** as availability assets?
- Did you name **the model provider** as an actor whose trust assumptions are not the same as your own?

If any of these is missing, it likely shows up as a missing STRIDE row in step 3. Catch it here.

## Output shape for step 1

The artifact's `## Assets` and `## Actors` sections take this enumeration and reformat it into the schema in `references/templates/threat-model.md`. The narrative paragraph this step produces is the prose that introduces those sections in the artifact — not a separate section.

Keep it under 400 words. If the system is large enough that 400 words is not enough, it likely deserves to be decomposed into multiple threat models scoped per subsystem.
