---
name: cross-model-deliberation
description: Runs an independent Claude and Codex decision review. Always use when the user asks for a second expert, cross-model opinion, or Claude-and-Codex deliberation. Otherwise, use only for a genuinely difficult and materially consequential architecture, security, data, concurrency, or ambiguous root-cause decision. Do not auto-invoke for routine, reversible, or clearly resolved work.
---

# Cross-Model Deliberation

Use Codex as an independent second expert before locking an exceptionally difficult or
consequential decision. Exchange concise decision memos with evidence and conclusions, not
private chain-of-thought.

## Trigger gate

An explicit user request for a second expert always triggers this workflow. Otherwise, run it
only when the issue is genuinely difficult, the outcome is materially consequential, and at
least one condition applies:

- Two or more credible architectural approaches remain and the choice has substantial blast
  radius, operational cost, or long-term lock-in.
- The decision is difficult to reverse, such as a public interface, data model or migration,
  platform or vendor commitment, or core system boundary.
- A security, authentication, payments, data-integrity, concurrency, or distributed-system
  decision has material blast radius, uncertainty, or cross-system impact.
- A hard diagnosis remains materially ambiguous after inspecting the available logs, tests,
  and evidence, or Claude cannot establish the root cause with high confidence.

Routine-case exclusions override the risk-domain examples during automatic routing. Do not
auto-invoke for routine bugs, narrow reversible choices, or decisions where the evidence clearly
points to one answer.

## Workflow

1. **Form Claude's independent view.** Inspect the evidence and create a concise provisional
   memo covering options, assumptions, risks, evidence, and a recommendation. Do not reveal the
   recommendation in Codex's initial prompt, so it cannot anchor Codex.
2. **Get Codex's independent view.** Invoke the host's Agent subagent surface with
   `subagent_type="codex:codex-rescue"` and a raw prompt beginning
   `--wait --fresh --model gpt-5.6-sol`. Run it in foreground mode. The outer Agent call and the
   forwarded rescue request must both wait for the result. Make the prompt neutral,
   self-contained, and explicitly read-only. Leave effort unset because the rescue interface
   does not accept `max`; Codex must inherit `model_reasoning_effort = "max"` from its effective
   configuration. Before dispatch, check the user-level config and any trusted project-level
   override. If effective max effort cannot be established, disclose that and stop rather than
   calling the result a max-effort deliberation. Ask for options, a recommendation, assumptions,
   supporting evidence, failure modes, missing evidence, and confidence.
3. **Compare; do not vote.** Identify agreements, material disagreements, novel risks, and
   missing evidence. Judge competing claims by evidence. Model agreement is not proof.
4. **Run one focused rebuttal only when needed.** If a disagreement could change the decision,
   launch one new read-only rescue consult with `--wait --fresh --model gpt-5.6-sol`. Embed both
   complete concise memos and the relevant evidence, then ask Codex to challenge the disputed
   assumptions. Do not use `--resume`: it may select an unrelated concurrent repository task.
   Do not exceed the initial consult plus one rebuttal unless the user asks.
5. **Synthesize for the user.** Present the recommended decision, why it won, the strongest
   opposing case, unresolved uncertainty, and any experiment needed to resolve it. The user
   decides every one-way or high-impact choice before implementation.
6. **Keep deliberation read-only.** Neither expert edits code during this workflow. Begin
   planning or implementation only after the decision is approved through the normal process.

## Rescue boundary

Use `codex:codex-rescue` only for this bounded, non-editing consultation. Never background, poll,
fetch, or convert a rescue consultation into an implementation task.

Codex is sandboxed read-only because the rescue call omits `--write`. Claude's no-edit rule is a
behavioral constraint: this skill cannot revoke tools already granted to the host session. Do not
describe Claude itself as sandboxed unless the host permissions independently enforce that.

Use a 15-minute foreground tool timeout where the host exposes one. Treat the second opinion as
unavailable if the Agent or companion call times out, errors, returns empty output, starts a
background job despite `--wait`, or supplies insufficient evidence. Disclose that state clearly
and do not label the decision cross-model validated. For a one-way or high-impact choice, ask the
user whether to proceed with Claude's analysis alone.
