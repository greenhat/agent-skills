---
name: commit-message-style
description: "Write commit messages in the preferred style for this workspace: `type: imperative summary` with backticks around code identifiers and no scope. Use when you are about to `git commit`, splitting work into multiple commits, or when the user asks you to commit code."
---

# Commit Message Style

## Format

- Use: `type: <imperative summary>`
- Use lowercase `type`, no scope: `fix: ...`, not `fix(sdk): ...`
- Prefer short, direct, present tense; no trailing period; keep to ~72 chars when possible
- Wrap code identifiers in backticks (types, fns, flags, files, crates): ``fix: gate `Felt` behind `cfg(miden)```
- Append an issue reference at the end only when relevant: `... #629`
- For non-trivial changes, add a commit body (blank line after subject) explaining:
  - **Why** the change is necessary (problem/constraint), and
  - **What** changed at a high level (only if complex enough).
- Keep the body short (typically 1–5 lines), wrap at ~72 chars, and avoid implementation details.

## Types

- `fix:` bugfix or behavior correction
- `refactor:` behavior-preserving changes (rename, extract, restructure)
- `feature:` user-visible functionality / new capability
- `test:` tests, fixtures, golden/expected outputs
- `chore:` docs/comments, cleanup, build/CI nits, dependency housekeeping

## Verbs (prefer)

- `add`, `remove`, `fix`, `refactor`, `extract`, `rename`, `unify`, `gate`, `document`, `update`, `simplify`

## Workflow

- If asked to address multiple review remarks: make one commit per remark (smallest coherent change per commit).
- Choose the `type` by intent (behavior change vs structure vs tests vs docs).
- Prefer stable naming (don’t churn identifiers) unless the change is explicitly a rename/refactor.

## Examples (match style)

- `refactor: extract \`account_id_inputs\` helper`
- `feature: gate on-chain \`Felt\` behind \`cfg(miden)\``
- `fix: serialize u64 as two u32 limbs`
- `chore: refine doc comments`
- `test: update integration expected outputs`
