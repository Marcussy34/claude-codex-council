---
name: cross-model-deliberation
description: Runs an independent Claude and Codex decision review. Always use when the user asks for a second expert, cross-model opinion, or Claude-and-Codex deliberation. Otherwise, use only for a genuinely difficult and materially consequential architecture, security, data, concurrency, or ambiguous root-cause decision. Do not auto-invoke for routine, reversible, or clearly resolved work.
---

# Cross-Model Deliberation

Use Codex as an independent second expert before locking an exceptionally difficult or
consequential decision. Exchange concise decision memos with evidence and conclusions, not
private chain-of-thought.

## Trigger gate

Routing is driven by the description. Once this skill has loaded, re-check the decision and
abort back to the main session unless the user explicitly asked for a second expert, or the
decision is both genuinely difficult and one of: hard to reverse, such as a public interface,
data model or migration, vendor commitment, or core system boundary; carrying material security,
authentication, payments, data-integrity, concurrency, or distributed-system blast radius; or
resting on a root cause still ambiguous after inspecting the available logs, tests, and evidence.

Routine bugs, narrow reversible choices, and decisions where the evidence already points to one
answer stay with Claude. These exclusions override the risk-domain examples above.

## Workflow

1. **Form Claude's independent view.** Inspect the evidence and write a concise provisional memo
   covering options, assumptions, risks, evidence, and a recommendation. Keep the recommendation
   out of Codex's initial brief so it cannot anchor Codex.
2. **Tell the user before dispatching.** A max-effort consult is slow and costly, and this skill
   can load automatically. State in one line that Codex is being consulted and why.
3. **Confirm effective max effort.** Read `~/.codex/config.toml`, then any project-level
   `.codex/config.toml`, which overrides the user-level file in a trusted project. Both must
   leave `model_reasoning_effort = "max"` in force. If that cannot be established, disclose it
   and stop rather than calling the result a max-effort deliberation.
4. **Get Codex's independent view.** Invoke the host's Agent subagent surface with
   `subagent_type="codex:codex-rescue"` and a raw prompt beginning
   `--wait --fresh --model gpt-5.6-sol`. The `--wait` keeps the companion run synchronous inside
   the subagent. The host's Agent surface may still return an async handle and deliver the result
   as a completion notification; that is normal, not a failure. Wait for the result and do not
   proceed to synthesis without it. Leave effort unset, because the rescue
   interface rejects `max` and Codex inherits it from configuration instead. Build the brief from
   [references/brief-template.md](references/brief-template.md), whose mandatory first line is
   what makes the wrapper request the read-only sandbox.
5. **Compare; do not vote.** Identify agreements, material disagreements, novel risks, and
   missing evidence. Judge competing claims by evidence. Model agreement is not proof. When the
   evidence is genuinely balanced, do not break the tie privately: present both cases and let the
   user decide.
6. **Run one focused rebuttal only when needed.** If a disagreement could change the decision,
   launch one new read-only consult with `--wait --fresh --model gpt-5.6-sol`, using the rebuttal
   section of the brief template. Do not use `--resume`: it may select an unrelated concurrent
   repository task. Do not exceed the initial consult plus one rebuttal unless the user asks.
7. **Synthesize for the user.** Follow
   [references/synthesis-template.md](references/synthesis-template.md). The user decides every
   one-way or high-impact choice before implementation.
8. **Keep deliberation read-only.** Neither expert edits code during this workflow. Begin
   planning or implementation only after the decision is approved through the normal process.

## Rescue boundary

Use `codex:codex-rescue` only for this bounded, non-editing consultation. Never background, poll,
fetch, or convert a rescue consultation into an implementation task.

The read-only sandbox is requested, not automatic. The rescue wrapper adds `--write` by default
and omits it only when the request clearly reads as review, diagnosis, or research without
edits; the companion then sends `sandbox: "read-only"` to the Codex runtime. So the request
rests on the brief's opening line, not on the routing flags. Never drop it. This per-request
value has been verified taking effect over a user-level `sandbox_mode = "danger-full-access"` on
codex-cli 0.150.1 via the rollout log, but that precedence lives in the Codex binary: after a
CLI upgrade, re-verify by checking `sandbox_policy` in the newest
`~/.codex/sessions/YYYY/MM/DD/rollout-*.jsonl`. The same check can confirm any consult after the
fact. Read-only scopes Codex's own filesystem actions; the companion still writes its job state
and transcripts, and this skill is not an independent security boundary. Claude's own no-edit
rule is a behavioral constraint: this skill cannot revoke tools already granted to the host
session. Do not describe Claude itself as sandboxed unless the host permissions independently
enforce that.

Use the host's maximum foreground timeout; on Claude Code that is the 10-minute Bash ceiling, and
the Agent tool exposes no timeout of its own. Treat the second opinion as unavailable if the
Agent or companion call times out, errors, returns empty output, or supplies insufficient
evidence, or if the companion detaches the run as a background job despite `--wait`. The host's
Agent surface returning asynchronously is not that failure; only a companion response that
reports a queued or backgrounded job is. Disclose that state clearly and do not label the
decision cross-model validated. For a one-way or high-impact choice, ask the user whether to
proceed with Claude's analysis alone.
