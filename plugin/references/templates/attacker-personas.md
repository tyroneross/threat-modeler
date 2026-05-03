# Generic Attacker Personas

A short library of attacker personas that show up repeatedly in agentic-system threat models. Copy the relevant ones into §4 (Attacker Personas) of `references/templates/threat-model.md`. Add system-specific personas where these generic ones miss something material.

Personas matter because STRIDE in the abstract is too soft — "spoofing" with no named adversary lets the artifact dodge specifics. A persona forces the threat to be concrete: *who* is doing the spoofing, *what* they want, and *what they can do* shape severity and mitigation. The personas below are intentionally narrow and named.

## AT-G1 — Outsider (script-kiddie / opportunistic)

**Profile.** Untrusted internet user with no insider access. Uses public attack surfaces (login pages, public APIs, scraping endpoints). Often automated; low cost per attempt; high volume.

**Capability.** Send arbitrary input through any user-facing surface. Try known prompt-injection payloads and tool-name guessing. Scrape public outputs.

**Typical goals.** Free use of paid services. Extract someone else's data through prompt injection. Discover internal tool / model names. Cause cost runaway as a denial-of-service.

**Maps most often to.** LLM01 (Prompt Injection), LLM04 (Model DoS), LLM06 (Sensitive Information Disclosure), ASI01 (Agent Goal Hijack), ASI02 (Tool Misuse). MITRE ATLAS reconnaissance and resource-development tactics.

**Severity calibration.** MEDIUM by default. HIGH when the system has T2+ tools or multi-tenant data accessible through the agent.

## AT-G2 — Authenticated Outsider (active customer / abuser)

**Profile.** A legitimate signed-up user with some authorization in the system. May be a paying customer, a free-tier user, or a malicious account created specifically for abuse.

**Capability.** Everything AT-G1 has, plus authenticated session, user-scoped tool calls, and the ability to feed inputs into the agent that the agent treats as authoritative because they came from the "user" channel.

**Typical goals.** Extract another user's data through tool-permission boundary errors (cross-tenant). Escalate from their own scope to broader scope through agent-mediated tool calls. Get the agent to act as them in a downstream system that does not validate identity propagation.

**Maps most often to.** ASI03 (Identity and Privilege Abuse), ASI02 (Tool Misuse), LLM07 (Insecure Plugin Design), LLM08 (Excessive Agency), OWASP Web A01 (Broken Access Control).

**Severity calibration.** Cross-tenant access is **CRITICAL** by default. Same-tenant privilege widening is **HIGH**.

## AT-G3 — Insider (operator / maintainer)

**Profile.** A human with legitimate operator access — deploys, configures, debugs, reviews logs. Could be a current employee, a contractor, or a former employee whose access was not promptly revoked.

**Capability.** Modify deployed config, redeploy, read logs, read memory contents, modify prompt templates if not gated by review, modify tool implementations.

**Typical goals.** Exfiltrate data. Plant a back door (a tool that exposes data, a prompt-template change that leaks). Cover their tracks.

**Maps most often to.** ASI06 (Memory and Context Poisoning — through stored prompt or memory tampering), LLM03 (Training Data Poisoning if training is in scope), STRIDE-T (tampering on prompts and configs), STRIDE-R (repudiation if logs are mutable).

**Severity calibration.** HIGH or CRITICAL by default. The mitigations are mostly process and infrastructure: append-only signed audit logs, four-eyes review on prompt-template and tool-implementation changes, prompt review-of-changes hooks, secret rotation on operator offboarding.

**Note.** Insider risk is uncomfortable but unavoidable. Treating it as out-of-scope is a security choice; log the choice in the decision log if you do.

## AT-G4 — Compromised Tool / Supply Chain

**Profile.** A tool the agent calls is compromised — the tool implementation itself, the package the tool depends on, the MCP server the tool integrates with, or the upstream API the tool wraps.

**Capability.** Return arbitrary content as tool output. Return content that looks like instructions (indirect prompt injection). Return content that looks like another tool's expected schema (cross-tool confusion). Return content that, when re-rendered downstream, executes (XSS, SSRF, RCE).

**Typical goals.** Hijack the agent's goal through indirect prompt injection. Get the agent to call other tools the tool itself could not call. Exfiltrate data via the next agent step. Inject persistent memory the agent will trust forever.

**Maps most often to.** LLM01 (Prompt Injection — indirect), LLM02 (Insecure Output Handling), LLM05 (Supply Chain), LLM07 (Insecure Plugin Design), ASI01 (Agent Goal Hijack), ASI04 (Agentic Supply Chain), ASI06 (Memory and Context Poisoning), ASI08 (Cascading Failures).

**Severity calibration.** HIGH by default. CRITICAL when (a) the tool is at T2+ or (b) the agent has memory write paths that persist tool output.

## AT-G5 — Prompt-Injection Attacker (at scale, multi-vector)

**Profile.** An adversary who specializes in prompt injection. May be the same person as AT-G1 / AT-G2 / AT-G4, but treated as a separate persona because the attack surface is unified across "user input", "tool output", and "RAG content".

**Capability.** Craft payloads that survive your delimiter scheme, your output validator, your downstream rendering layers. Has read up on the system through public outputs (AT-G1) and adapts.

**Typical goals.** Same as AT-G4, but originated from any channel that feeds the agent's prompt — including indirect channels you did not think about (a comment on a third-party forum the agent's tool fetches; a value in a database row the agent reads; a filename in a directory listing).

**Maps most often to.** LLM01, ASI01, ASI06, ASI08. ATLAS techniques in the "Initial Access" and "ML Attack Staging" tactics when the injection eventually reaches model behavior.

**Severity calibration.** HIGH by default. The mitigation surface is the union of every input channel and every memory write path; the artifact's mitigation rows for this persona are usually multi-row.

## AT-G6 — Compromised Downstream / A2A Peer

**Profile.** Specific to multi-agent systems. A peer agent (orchestrator, specialist, or external A2A peer) is compromised or behaves adversarially. Its outputs reach this agent through an A2A handoff.

**Capability.** Send messages with crafted content. Drop or replace identity claims at the handoff boundary. Replay old messages.

**Typical goals.** Trick this agent into acting on the peer's behalf when it should act on the user's. Get cross-agent privilege escalation by laundering identity through the orchestrator.

**Maps most often to.** ASI03 (Identity and Privilege Abuse), ASI07 (Insecure Inter-Agent Communication), ASI10 (Rogue Agents), ASI08 (Cascading Failures).

**Severity calibration.** HIGH or CRITICAL. ASI07's coverage gap (no clean general-purpose runtime control) is honest; surface it in the decision log rather than papering over.

## AT-G7 — Compromised Model Provider / Model Itself

**Profile.** The model provider returns adversarial content, or the model itself has been backdoored / poisoned at training time. Rare but worth listing for systems with regulated data or high-impact actions.

**Capability.** Subtly biased outputs that escape obvious validators. Targeted misclassification that benefits a specific party. In the limit: a backdoored model that behaves normally except on a trigger phrase.

**Typical goals.** Model-level influence operations. Inserted bias that benefits the attacker over time. Watermark / trigger-phrase activation.

**Maps most often to.** LLM03 (Training Data Poisoning), LLM05 (Supply Chain), ASI04 (Agentic Supply Chain), NIST AI 600-1 Information Integrity / Harmful Bias and Homogenization.

**Severity calibration.** Usually MEDIUM in agentic systems because mitigation is mostly out of scope for the agent author (you do not retrain the foundation model). Capture and accept; require periodic eval against a held-out adversarial set if the system handles regulated data.

## How to use this library in §4

For each system, copy 3–5 of the personas above into the artifact's §4. Most agentic systems should include AT-G1, AT-G2, AT-G4, and AT-G5 by default. Add AT-G3 when insider risk is in scope; add AT-G6 if the system is multi-agent; add AT-G7 only if you have a real reason to.

For each persona kept, the §4 row carries:

- The persona ID.
- A one-sentence summary of *what they want* in this specific system.
- A one-sentence summary of *what they can do* against this specific system.
- A pointer to the threat IDs (`T-NNN`) in §6 that this persona drives.

Personas that are not driving any threat ID in §6 should not be in §4. The personas exist to make threats concrete, not to lengthen the artifact.
