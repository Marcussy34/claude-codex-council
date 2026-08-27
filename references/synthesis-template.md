# Synthesis template

The final user-facing output of a deliberation (step 7). Present it directly in the response.
The user decides; this artifact exists to make that decision cheap and well-founded.

## Structure

```text
## Recommendation
<The decision, in one or two sentences. State it plainly.>

## Why it wins
<The evidence that decides it, cited by file and line or by test and log. Reasoning, not
a vote count. Never justify a conclusion by saying both models agreed.>

## Strongest opposing case
<The best argument against the recommendation, stated fairly enough that its proponent would
recognise it. If Codex dissented, this is where its position goes, undiluted.>

## Where the analyses diverged
<Material disagreements only. For each: what each side claimed, and what evidence settled it or
left it open. Omit this section entirely when there was no material disagreement, and say so.>

## Unresolved uncertainty
<What is still not known, and what would resolve it: a specific experiment, benchmark, query, or
spike. If a cheap test would settle a material question, recommend running it before committing.>

## Decision needed from you
<The explicit ask. For a one-way or high-impact choice, do not proceed without approval.>
```

## Rules

- Report degraded runs honestly. If the consult timed out, errored, returned empty output, went
  to background despite `--wait`, or came back thin, say so and do not label the decision
  cross-model validated. Offer to proceed on Claude's analysis alone.
- When the evidence is genuinely balanced, do not manufacture a winner. Present both cases under
  Recommendation and hand the tie to the user.
- Agreement between the two models is evidence to inspect, not proof. Shared blind spots are the
  expected failure mode of this workflow; say when both analyses rest on the same unverified
  assumption.
- No implementation begins in this response.
