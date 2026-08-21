---
name: no-slop
description: Score, audit, and fix AI-generated slop in code, diffs, PRs, or docs in one pass — score each category 0-5 with file:line evidence, propose a fix list ordered delete → consolidate → rewrite, get explicit user confirmation, then apply. Use when asked to clean up, de-slop, fix slop, review, audit, grade, or check code/diffs/PRs/docs for AI slop, vibe-coded smell, or quality after AI generation.
---

# No Slop

Slop is not "AI-written code." It is code that compiles, passes tests, and still can't be trusted: every failure grows a new fallback, every migration keeps the old path "just in case," every doc explains why instead of how. Fixing it means deleting the extra paths, not adding a new one on top. Audit with evidence, not vibes.

## Scope

| Scope | Meaning | Resolve with |
| --- | --- | --- |
| Uncommitted | working tree: staged + unstaged, not yet committed | `git diff HEAD` |
| Unpushed | local commits ahead of the remote branch | `git diff @{u}...HEAD` |
| PR / branch | this branch vs. its base (merge-base diff) | `git diff $(git merge-base <base> HEAD)...HEAD`, or `gh pr diff` if a PR is open |
| Whole repo | everything, no diff | scan the tree directly |

If the user names a scope, use it. Otherwise resolve in this priority order and use the first one that's non-empty: uncommitted → unpushed → PR/branch → whole repo. State the resolved scope at the top of your output so the user can override it.

## Workflow

1. **Scope** — resolve using the table above.
2. **Score** — score each category below 0–5, citing file:line evidence for anything above 1. Any single category at 5 is a blocker regardless of the average.
3. **Propose** — present a fix list ordered delete → consolidate → rewrite, using the format below. **Stop and get explicit confirmation before editing anything.**
4. **Apply** — only the confirmed items, in that order.
5. **Verify & report** — run existing tests/build if available; state exactly what you deleted and changed; then self-audit: *"what still looks AI-generated?"* (comment/PR tells, process leakage) and fix the remaining tells.

## Scoring rubric

- **0 Sharp** — one path per operation, no dead weight, docs match audience.
- **1 Pragmatic** — a couple of justified adapters/flags with clear contracts.
- **2 Slight slop** — one unnecessary env var, one duplicated helper, one narrative comment.
- **3 Visible slop** — multiple categories show real smell; a reviewer has to guess which path runs.
- **4 Heavy slop** — several categories at 3+; failure behavior is undocumented and inconsistent.
- **5 Slopmax** — nobody can say which of N paths runs in production.

Overall score = average of categories below. Any single category at 5 is a blocker regardless of the average.

## Categories

| Category | What to check | Strong slop evidence |
| --- | --- | --- |
| Env & config sprawl | new env vars, flags, config knobs | added "just in case," nothing reads them yet, or the value has only one correct answer |
| Fallback hydra | try/catch chains, retries, resolvers | 2+ fallback paths for one operation; final catch returns a plausible-looking default |
| Legacy & dead branches | deprecated/legacy/compat code | old path kept "for safety" after a migration; commented-out or unreachable code |
| Error honesty | catch blocks, broad excepts | swallowed exceptions, no logging, silent success on failure |
| Duplication | new helpers/types vs. existing ones | a near-identical function already exists two directories away |
| Comment & process leakage | comments, docstrings, dead attempts | comment narrates the diff or restates code instead of giving a non-obvious *why*; intermediate attempts left in (exploratory code, superseded snippets, commented-out experiments); decision history (rejected alternatives, "we tried X first") written into code comments |
| Docs & PR audience fit | README, help text, PR descriptions, 对客文档 vs. internal notes | customer-facing doc or PR description explains decision process (rejected alternatives, "never merged" states, attempt history) instead of final behavior |
| Structural erosion | file/function size, ownership | one file or function knows every subsystem; god class/function |

## Comment & PR discipline

- **Comments: non-obvious reason only.** A comment must explain something the code doesn't (why not how). If it restates the code or narrates the change, delete it.
- **No intermediate attempts.** Never keep superseded snippets, exploratory code, or "previous approach" blocks. The codebase shows only the final state.
- **PR descriptions: final behavior only.** Describe what the code does now and any trade-off invisible in the diff (e.g. "chose X over Y because Z"). Never mention rejected alternatives, never-merged states, or how you got there. Decision history belongs in an ADR/changelog, not in the PR or the code.
- **No decision-process leakage in code/docs/comments.** If a reader can reconstruct *how* the code was written (attempts, pivots, dead ends) from comments, docs, or the PR, that's slop.

## AI tells in comments & PRs (borrowed from unslop)

- AI vocabulary: *Additionally, crucial, delve, enhance, foster, intricate, landscape (abstract), pivotal, robust, seamless, showcase, underscore, vibrant*. Replace with plain words.
- "Not just X, but Y." / rule of three / synonym cycling (pick one word and repeat it).
- Em-dash overuse; chatbot phrases ("This PR enhances...", "Let me know if...", "Of course!").
- Puffery and promotional words ("streamlined", "empowered", "groundbreaking") — state what happened, don't market it.
- Self-audit after fixing: *"What still makes this look AI-generated?"* Fix remaining tells.

## Score report format (show before any proposal)

```markdown
## Verdict
Slop score: X/5 — one direct sentence.

## Scorecard
| Category | Score | Evidence |
| --- | ---: | --- |

## Top risks
1. file:line — the finding and why it matters (ranked by severity, not by category order)

## Clean target
One or two sentences describing the boring, single-path version.
```

## Fix list format (show this before touching code)

```markdown
## Fix list for [scope]
1. [delete] file:line — dead/legacy/fallback branch, why it's safe to remove
2. [consolidate] file:line + file:line — duplicate helpers to merge, keep which one
3. [rewrite] file:line — swallowed error / narrative comment / config to fix

Proceed with all, or pick numbers?
```

Wait for the user's answer before editing anything.

## Fix rules

- Delete before you add — most slop is inert; removing it changes no runtime behavior.
- Collapse fallback chains to the one path that should run; make failure explicit (typed error or clear log), never a silent default.
- When merging duplicates, keep the more-used or better-tested version and update its callers.
- If a customer-facing doc or PR description has decision history worth keeping, move it into an internal note (ADR/changelog) instead of just deleting it.
- Never introduce a new env var, fallback, or shim while fixing — that's slop again.

## Rules

- Evidence only: cite file paths and symbols, not impressions.
- Rank by severity: a swallowed error in a payment path outranks a narrative comment.
- Size ≠ slop. A large file can be sharp if it has one clear owner and no dead paths.
- Score and propose in a single pass; never apply changes without explicit confirmation.
