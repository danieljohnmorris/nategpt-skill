---
name: nategpt
description: Use when the user requests nategpt mode or an Astra-planned build coordinated by Luna with fresh Astra workers. Keeps coordination history on Luna and gives Astra bounded implementation and unblock assignments.
tags: [orchestration, models, workflow]
---

# Nategpt

An opt-in workflow for substantial builds with independent work packages. Astra plans, a persistent Luna agent coordinates, and fresh Astra agents implement and resolve blockers. This is a skill and custom-agent workflow, not a built-in CLI mode. Do not apply it to small tasks where delegation costs more than the work.

## Installed roles

- Main: Astra planner and final reporter. Start a fresh session on `openai-codex/gpt-6-astra` for this workflow. If Main uses another model, disclose that and ask the user to select Astra before claiming the exact workflow is active.
- `nategpt-luna`: `openai-codex/gpt-5.6-luna:medium`, persistent coordinator.
- `nategpt-astra`: `openai-codex/gpt-6-astra:high`, fresh implementation worker.
- `nategpt-bulldozer`: `openai-codex/gpt-6-astra:high`, bounded unblock or milestone review.

The custom agents live in `~/.omp/agent/agents/`. They do not replace normal task agents or change global model roles. Task model overrides and project agents can override them: verify resolved models before calling a run nategpt. Never silently substitute an unavailable model. Existing approvals, repository rules and tool policies still apply.

## Plan and handoff

1. Main scopes the repository and owns decomposition. Read the relevant skills. Define observable acceptance gates before edits. Independent packages need distinct file ownership and agreed interfaces; shared-file changes have one integration owner.
2. Write a durable handoff in a unique run directory under `~/.omp/agent/nategpt-runs/`. Use a timestamp and unique suffix to avoid collisions. Include the absolute repository path, request, constraints, non-goals, evidence, acceptance gates, package dependency graph, exact file ownership, interface contracts, verification commands and environment limitations. Do not include credentials or the full conversation.
3. Write a sibling `status.md` with package states, owners, active agent IDs, evidence paths, decisions, blockers and next actions. Luna alone maintains it. Main keeps the user-facing todo list. Repository-specific tracking requirements still apply.
4. The handoff must identify integration ownership and when an Astra review is required. Workers skip formatting, linting, builds and tests during concurrent edits; Luna runs the agreed validation after each integration boundary. Regression checks must actually exercise the changed behaviour.
5. Spawn `nategpt-luna` with the absolute handoff and status paths. The assignment must instruct Luna to execute the whole approved build, not merely return another plan. OMP's default two-level depth supports Main -> Luna -> Astra; do not raise depth or concurrency globally. If the effective configuration blocks nesting, report the setting rather than pretending the workflow ran.

## Execution

Luna retains dependency and decision history. New implementation packages receive fresh `nategpt-astra` contexts. Give each worker only its package, shared interfaces, relevant paths and acceptance criteria. Source files remain evidence: workers inspect them rather than trusting summaries blindly. Exploration that needs delegation uses `scout` under the normal tool policy.

Use real concurrency only for independent packages. Do not spawn a target number of agents. Batch independent work, with one writer per file. Follow the installed harness concurrency cap; keep batches modest rather than creating idle agents.

Luna requests a fresh `nategpt-bulldozer` for an unresolved technical blocker, repeated failure, contract conflict, or a completed milestone needing independent review. Send a short evidence packet: symptom, exact errors, attempted remedies, relevant paths, ownership and the decision needed. A bulldozer can implement a bounded fix only after Luna transfers file ownership and competing writers stop. It reports evidence and exits. It never watches by repeatedly polling Luna or accumulates the full build transcript.

If Luna stops responding or reports a coordination deadlock, Main may launch a fresh bulldozer with the latest status and handoff to diagnose it. Main should not ingest every worker transcript or run a second competing coordinator. Use event delivery and `hub` messages; no busy polling. Do not enable an Astra advisor for Luna: an advisor accumulates transcript history and defeats this context separation. Existing Main advisors are separate usage and must be included in cost reporting.

Main must remain active until the coordinator finishes or an actual external prerequisite blocks the work. Do not deliver a final answer just because Luna was launched: child lifetimes depend on the parent session. A process restart is not automatic recovery. On resume, read the durable status, discover live agents before sending messages, reconcile repository state, and continue from verified work without duplicating writers.

## Completion and measurement

Luna integrates, runs the agreed checks, updates status and returns a compact final report with changed paths, acceptance evidence, failures and external blockers. Main checks the evidence against the original request and reports only verified results. An agent yielding successfully is not acceptance proof.

Use OMP's recorded usage for Main, Luna, workers, bulldozers and advisors. Separate model token/cost estimates from actual billing, especially on Codex subscriptions. No savings claim without a comparable baseline. Fresh agents still load system instructions, tools and repository context; they do not have zero input overhead. Luna's usable context is whatever the selected provider reports, and compaction may still be required.

## Invocation

In a fresh Astra OMP session:

`/skill:nategpt <build request>`

Natural-language trigger: `Use nategpt mode for this build.`

This skill does not change iTerm startup, normal OMP defaults, approval settings or provider credentials.
