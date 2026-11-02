# staged-subagent-delivery

| Field | Value |
| --- | --- |
| Tool | OpenCode |
| Source | Custom (`~/.config/opencode/skills/staged-subagent-delivery`) |
| Status | Active |
| Invocation | Loaded when asked to deliver a batch of backlog items via subagents |

## Purpose

Delivers a batch of related backlog items (e.g. a tier in `current-work.md`) through five subagent stages. The main agent orchestrates and does not implement.

## Inputs

- Backlog document and which items or tier to deliver
- Repo rules (`AGENTS.md`) and verification commands (typecheck, lint, test, build, web checks, e2e)

## Stages

Parallel within a stage, with a hard barrier between stages.

1. **Plan** (parallel, read-only): one `general` subagent per item. Each plan lists files, signatures, decisions, tests, docs and verification, and flags collisions with sibling plans. Shared resources such as migration numbers are reserved up front.
2. **Implement** (parallel): items that touch the same files are grouped into one agent with a stated order. Each prompt carries the full plan and shared-tree rules (targeted edits only, no `git stash/checkout/reset`, no repo-wide formatters).
3. **Review and test** (parallel): read-only reviewers classify findings HIGH/MEDIUM/LOW. Testers run all checks plus runtime verification on a scratch DB, non-default port and random secrets, and kill every server they start.
4. **Fix** (one agent): fixes HIGH findings, or the correctness-affecting MEDIUMs if there are no HIGHs, with a test per fix and regenerated artifacts. LOWs are skipped.
5. **Final review and test** (parallel): checks the fixes for regressions, runs the full suite, e2e, format check, smoke test and `git status`. Pre-existing failures are recorded separately from new ones.

## Wrap-up

1. Update the backlog doc: mark items DONE, add a status block and a follow-ups list of unfixed MEDIUM/LOW findings.
2. Draft the commit message (by theme) and show it. Commit only when asked.
3. Stage only this work, leave unrelated untracked files out, never push.

## Rules of thumb

- Wait for all agents in a stage before starting the next
- Planners and reviewers never edit; implementers never review their own work
- Report honestly what was not run, what was hand-verified and what remains open

## Notes

- TODO: record which backlogs or tiers it has been used on
- TODO: note stage timings or tweaks that worked well
