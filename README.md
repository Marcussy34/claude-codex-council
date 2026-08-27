# Claude × Codex Council

> Two frontier coding agents. Independent reasoning. One evidence-based decision.

Claude × Codex Council is a personal Claude Code skill for difficult architecture and root-cause
decisions. Claude forms its own position, asks OpenAI Codex for an independent read-only opinion,
compares the evidence, and presents one synthesis for the user to decide.

It is deliberately selective. An explicit request for a second expert always triggers it.
Otherwise, routine bugs and reversible choices stay with Claude; genuinely difficult and
materially consequential decisions with high blast radius, long-term lock-in, security or
data-integrity risk, or unresolved root causes get a second expert.

## How it works

1. Claude examines the evidence and writes a provisional decision memo.
2. Codex receives the same evidence without Claude's recommendation, reducing anchoring.
3. Claude compares agreements, disagreements, missing evidence, and failure modes.
4. One fresh rebuttal is allowed when a disagreement could change the decision.
5. Claude synthesizes the result; the user approves every high-impact or one-way choice.

The Codex consultation runs in the plugin's read-only sandbox. That is not automatic: the rescue
wrapper adds `--write` by default and omits it only when the request clearly reads as review or
diagnosis without edits, so every brief opens with a mandatory read-only line. The skill also
instructs Claude not to edit, but a skill cannot revoke tools already granted to the host session.
For strict enforcement, use Claude Code permission controls or a disposable read-only workspace.

## Prerequisites

- A local [Claude Code](https://code.claude.com/docs/en/overview) session
- Node.js 18.18 or later
- A ChatGPT subscription, including Free, or an OpenAI API key for Codex authentication
- [OpenAI Codex CLI](https://developers.openai.com/codex/cli/)
- The official [Codex plugin for Claude Code](https://github.com/openai/codex-plugin-cc)
- Access to `gpt-5.6-sol`

Install the Codex plugin from inside Claude Code:

```text
/plugin marketplace add openai/codex-plugin-cc
/plugin install codex@openai-codex
/reload-plugins
/codex:setup
```

`/codex:setup` verifies the Codex CLI and authentication and may offer to install the CLI when it
is missing. The completed setup must expose the `codex:codex-rescue` subagent.

This installation targets local Claude Code. Cowork and cloud sessions do not automatically read
personal skills from `~/.claude/skills` or use the machine-local Codex CLI and plugin. The skill
text can be distributed separately through Claude's other
[skill locations](https://code.claude.com/docs/en/skills#where-skills-live), but the complete
workflow still needs a compatible Codex rescue runtime in that environment.

## Install the skill

Clone this repository into Claude Code's personal skills directory:

```bash
mkdir -p ~/.claude/skills
git clone https://github.com/Marcussy34/claude-codex-council.git \
  ~/.claude/skills/cross-model-deliberation
```

The target directory name matters: it must be `cross-model-deliberation` to match the skill's
`name` field. Cloning without the explicit path leaves a `claude-codex-council` directory and a
name mismatch.

Start a fresh Claude Code session after installation. The skill is available as
`/cross-model-deliberation` and can also load automatically when its description matches the
task.

## Configure max-effort Codex

The skill pins the Codex model to `gpt-5.6-sol` and intentionally leaves the effort flag unset
because the rescue interface does not currently accept `max`. Set the inherited effort in the
effective Codex configuration. A user-level default belongs in `~/.codex/config.toml`:

```toml
model = "gpt-5.6-sol"
model_reasoning_effort = "max"
```

Trusted projects can override this with `.codex/config.toml`. Confirm that no project-level
override lowers `model_reasoning_effort` before relying on max-effort deliberation. Configuration
at either scope also applies to other Codex runs at that scope.

## Add the routing rule

For reliable automatic routing, add this compact rule to `~/.claude/CLAUDE.md`:

```md
### Cross-Model Deliberation

Whenever the user requests a second expert, invoke `/cross-model-deliberation`. Otherwise,
invoke it before deciding or implementing only when the issue is genuinely difficult, materially
consequential, and involves a hard-to-reverse choice or unresolved high-impact risk. Do not
auto-invoke it for routine, reversible, or clearly resolved work.
```

This keeps the always-loaded instruction small. Claude Code loads the full `SKILL.md` body only
when the skill is used.

## Usage

Ask naturally:

```text
We have two credible database architectures and this choice will be expensive to reverse.
Have Claude and Codex deliberate before recommending one.
```

Or invoke the skill directly:

```text
/cross-model-deliberation Review this migration strategy before we lock the schema.
```

The skill should not auto-activate for ordinary work such as typo fixes, small reversible
refactors, or bugs whose root cause is already established by evidence, unless the user
explicitly requests a second expert.

## Safety and limitations

- Codex is invoked with a fresh brief whose first line explicitly requests review and diagnosis
  without edits. The rescue wrapper honors that by omitting `--write`, which selects the
  companion's read-only sandbox. The guarantee rests on that line, not on the routing flags.
- Claude's no-edit rule is a behavioral instruction, not a host-level security boundary.
- Claude and Codex can still share blind spots. Agreement is evidence to inspect, not proof.
- No implementation begins until the user approves the synthesized decision.
- The workflow depends on the Codex plugin's `codex:codex-rescue` interface and may need updates
  if that plugin changes.
- Model availability and usage limits depend on the user's OpenAI account.

## Validation

The skill has been tested in fresh Claude Code sessions with two routing cases:

- A hard-to-reverse payment-ledger architecture decision loaded the skill and completed one
  foreground Codex consultation.
- A README typo stayed in the main Claude session and spawned no subagent.

## License

[MIT](LICENSE)

## Disclaimer

This is an independent community project. It is not affiliated with or endorsed by Anthropic or
OpenAI.
