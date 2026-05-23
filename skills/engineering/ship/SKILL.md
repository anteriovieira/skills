---
name: ship
description: Orchestrates agents to implement, review, and merge ready-for-agent GitHub issues in parallel using worktrees. Use when the user says "ship", "ship issues", "ship the backlog", "tackle the backlog", "implement open issues", or asks to autonomously implement GitHub issues as a team.
---

# Ship Open Issues as a Team

You are the **orchestrator**. Coordinate a team to analyze, implement, review, and merge open GitHub issues.

---

## Phase 1 — Analyze

Fetch only triaged, agent-ready issues and recent commits:

```bash
gh issue list --state open --label "ready-for-agent" --json number,title,body,labels,comments \
  --jq '[.[] | {number, title, body, labels: [.labels[].name], comments: [.comments[].body]}]'
git log -n 10 --format="%H%n%ad%n%B---" --date=short
```

Only issues labeled `ready-for-agent` are in scope. Issues with `needs-triage` or no triage label have not been reviewed by a maintainer and must be skipped — do not implement them even if they look straightforward.

Build a dependency graph among the `ready-for-agent` issues. Issue B is **blocked by** A if:
- B requires code or infrastructure A introduces
- B and A modify overlapping files/modules (likely merge conflicts)
- B depends on a decision or API shape A will establish

An issue is **unblocked** if it has zero blocking dependencies among the `ready-for-agent` set.

PRDs with implementation issues linking to them are **not workable** — only their children are.

Select all unblocked issues. If every issue is blocked, pick the single highest-priority candidate (fewest/weakest dependencies).

For each selected issue, create an isolated worktree:

```bash
git worktree add ../ship-issue-{number} -b issue-{number}
```

Pass the worktree path and branch name to each member agent.

---

## Phase 2 — Agents (parallel)

For each selected issue, dispatch a **member** agent in parallel via the Task tool. Each member receives `{number}`, `{worktree_path}` (e.g. `../ship-issue-{number}`), and `{branch}` (e.g. `issue-{number}`).

> You implement exactly ONE issue inside `{worktree_path}`. Do not touch other worktrees or `main`.
>
> All commands run from `{worktree_path}`.
>
> 1. `gh issue view {number} --comments`. If a parent PRD is referenced, pull it too.
> 2. Explore the repo. Pay extra attention to test files that touch the relevant code.
> 3. Implement using RGR: **RED** (one failing test) → **GREEN** (minimum implementation) → **REPEAT** until done → **REFACTOR**.
> 4. Run `pnpm typecheck` and `pnpm test`. Both must pass.
> 5. Commit with a message that:
>    - Starts with `chore:`
>    - States task completed + PRD reference
>    - Lists key decisions
>    - Lists files changed
>    - Notes blockers/follow-ups
> 6. If the task is not complete, leave a comment on the issue with progress — do **not** close it.
> 7. Output exactly `<promise>COMPLETE</promise>` when finished.

When an agent finish the work, call the next agent to review it.

---

## Phase 3 — Reviewers (parallel)

For each worktree created, dispatch a **Reviewer** agent in parallel. Each receives `{number}` and `{worktree_path}`.

> You enhance code clarity, consistency, and maintainability **without changing functionality**.
>
> All commands run from `{worktree_path}`.
>
> 1. Establish baseline: `pnpm typecheck && pnpm test` must pass before you change anything.
> 2. Read `git diff main..HEAD`. Find anything dodgy — fragile logic, unchecked assumptions, tricky conditions, implicit coercions, missing guards. For each, write a test that tries to break it. If you break it, fix it.
> 3. Stress-test edge cases for every changed code path:
>    - Empty arrays/strings, zero, negatives
>    - Missing optional fields, null, undefined
>    - Rapid repeated calls, race conditions, state mutating mid-operation
>    - Off-by-one in loops/slices
>    - Regressions in adjacent functionality
> 4. Apply code quality improvements: reduce nesting, remove redundancy, clearer names, consolidate related logic, drop comments that restate obvious code, avoid nested ternaries (prefer switch or if/else), clarity over brevity.
> 5. Maintain balance — don't over-simplify into clever-but-opaque code, don't combine too many concerns, don't remove helpful abstractions.
> 6. Follow `CLAUDE.md` coding standards.
> 7. Re-run `pnpm typecheck && pnpm test`.
> 8. If you made changes, commit starting with `ship: Review -` describing the refinements. If the code was already clean, do nothing.

---

## Phase 4 — Merge

Dispatch a single **Merger** agent with the list of `(number, branch, worktree_path)` tuples.

> You integrate reviewed branches into `main` and close their issues.
>
> All commands run from the **main worktree** (the original repo directory, not any issue worktree). The branches already exist in the shared git repo — no need to enter the issue worktrees.
>
> For each `(number, branch, worktree_path)`:
> 1. `git merge {branch} --no-edit`
> 2. If conflicts arise, resolve them intelligently — read both sides and choose correctly.
> 3. `pnpm typecheck && pnpm test`. If tests fail, fix before moving on.
>
> After all branches are merged:
> 1. Make a single summary commit: `merge: merge issues {numbers}`
> 2. Remove each worktree: `git worktree remove {worktree_path} --force`
> 3. Delete the merged branches: `git branch -d {branch}`
>
> Then close each issue: `gh issue close {number} --comment "Merged via {branch}"`. If closing an issue completes a parent PRD (all children now closed), close the parent too.

---

## Phase 5 — Report

Summarize: issues completed, worktrees merged, issues still blocked.

---

## Hard rules

- Agents run **one issue per branch** — never overlap.
- Agents do **not** close issues; only the merger does.
- Commit messages start with `chore:` for Agents, `ship: Review -` for reviewers.
- `pnpm typecheck` and `pnpm test` must pass before any commit.
