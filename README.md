# nategpt

An opt-in agent workflow for substantial builds with independent work packages: **Astra plans**, a **persistent Luna coordinator** executes, and **fresh Astra workers** implement bounded packages. Coordination history stays on Luna; workers never inherit the full transcript.

Inspired by [Nate B Jones' tweet](https://x.com/natebjones/status/2100097467429904797).

## Contents

| Path | Role |
| --- | --- |
| `SKILL.md` | The skill: planning, handoff, execution, completion rules |
| `agents/nategpt-luna.md` | Persistent coordinator (Luna, medium) |
| `agents/nategpt-astra.md` | Fresh implementation worker (Astra, high) |
| `agents/nategpt-bulldozer.md` | Bounded unblock / milestone reviewer (Astra, high) |

## Install (omp)

```sh
git clone https://github.com/danieljohnmorris/nategpt-skill
cd nategpt-skill
mkdir -p ~/.omp/agent/skills/nategpt ~/.omp/agent/agents
cp SKILL.md ~/.omp/agent/skills/nategpt/
cp agents/*.md ~/.omp/agent/agents/
```

Claude Code equivalent: put `SKILL.md` under `~/.claude/skills/nategpt/` and the agent files under `~/.claude/agents/`. Model strings in the agent frontmatter are omp-specific — adjust to your provider's names.

## Use

In a fresh Astra session:

```
/skill:nategpt <build request>
```

Or: `Use nategpt mode for this build.`

Don't use it for small tasks where delegation costs more than the work.

## Notes

- Agent frontmatter lists model fallbacks in preference order; never silently substitute an unavailable model.
- Requires two-level agent nesting (Main → Luna → Astra). Don't raise global depth or concurrency limits for it.
- Handoff and status ledger live in a timestamped run directory so a compaction or restart can recover.
