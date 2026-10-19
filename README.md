# nategpt

An opt-in agent workflow for substantial builds with independent work packages: **Astra plans**, a **persistent Luna coordinator** executes, and **fresh Astra workers** implement bounded packages. Coordination history stays on Luna; workers never inherit the full transcript.

Inspired by [Nate B Jones' tweet](https://x.com/natebjones/status/2100097467429904797).

Two variants ship from this repo. Same rules, different models.

| Variant | Planner (Main) | Coordinator | Workers and bulldozer |
| --- | --- | --- | --- |
| `nategpt` | Astra | Luna, medium | Astra, high |
| `nateclaude` | Claude Fable 5.1 | Sonnet 5.5, medium | Claude Fable 5.1, high |

## Contents

| Path | Role |
| --- | --- |
| `SKILL.md` | nategpt skill: planning, handoff, execution, completion rules |
| `agents/nategpt-luna.md` | Persistent coordinator (Luna, medium) |
| `agents/nategpt-astra.md` | Fresh implementation worker (Astra, high) |
| `agents/nategpt-bulldozer.md` | Bounded unblock / milestone reviewer (Astra, high) |
| `nateclaude/SKILL.md` | nateclaude skill, the same rules on Anthropic models |
| `nateclaude/agents/nateclaude-coordinator.md` | Persistent coordinator (Sonnet 5.5, medium) |
| `nateclaude/agents/nateclaude-worker.md` | Fresh implementation worker (Fable 5.1, high) |
| `nateclaude/agents/nateclaude-bulldozer.md` | Bounded unblock / milestone reviewer (Fable 5.1, high) |

## Install (omp)

```sh
git clone https://github.com/danieljohnmorris/nategpt-skill
cd nategpt-skill
mkdir -p ~/.omp/agent/skills/nategpt ~/.omp/agent/skills/nateclaude ~/.omp/agent/agents
cp SKILL.md ~/.omp/agent/skills/nategpt/
cp agents/*.md ~/.omp/agent/agents/
cp nateclaude/SKILL.md ~/.omp/agent/skills/nateclaude/
cp nateclaude/agents/*.md ~/.omp/agent/agents/
```

Install one variant or both; the agent names do not collide.

Claude Code equivalent: put each `SKILL.md` under `~/.claude/skills/<name>/` and the agent files under `~/.claude/agents/`. Model strings in the agent frontmatter are omp-specific, so adjust them to your provider's names.

## Use

nategpt, in a fresh Astra session:

```
/skill:nategpt <build request>
```

nateclaude, in a fresh Claude session:

```
/skill:nateclaude <build request>
```

Or: `Use nategpt mode for this build.` / `Use nateclaude mode for this build.`

Don't use it for small tasks where delegation costs more than the work.

## Notes

- Agent frontmatter lists model fallbacks in preference order; never silently substitute an unavailable model.
- Requires two-level agent nesting (Main → Luna → Astra). Don't raise global depth or concurrency limits for it.
- Handoff and status ledger live in a timestamped run directory so a compaction or restart can recover.
