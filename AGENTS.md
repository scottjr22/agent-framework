# Repository expectations

- Do not update AGENTS.md without permission
- Do not update `docs/references/casaigo-passport-schema.md` without permission
- Do not update `docs/references/casaigo-passport-architecture.md`
  without permission
- Except for plan phase, all work for a user story should be done on a new git worktree separate from the main tree.

# State File Standardization

These rules exist to prevent context rot and spec drift.

## Directory Layout

All state for a feature lives under one folder in `docs/`:

- `{name_of_feature}/feature.md`
- `{name_of_feature}/spec.md`
- `{name_of_feature}/{task_id}/decision.md`
- `{name_of_feature}/{task_id}/progress.md`

## Ownership Rules

- `feature.md`: human-owned source of truth (agents may not edit unless explicitly instructed)
- `spec.md`: agent-authored, human-approved. Any change requires human approval.
- `decision.md`: append-only. Never rewrite history.
- `progress.md`: updated by every phase. Treated as operational state.
- `qa_report.md`: updated by QA agent. output of qa findings

## Required Formats

### decision.md (append-only)

Every entry must include:

- Timestamp (ISO-8601)
- Decision
- Rationale
- Impacted files / modules

Example:

- `2026-03-05T21:14:00-05:00` — Decision: Use monthly bitmap storage for availability. Rationale: reduces query cost. Impact: `availability_index.ts`.

### progress.md

Must start with a single "front matter" block that is always kept current:

- Phase Per User Story: Planner | Implementer | QA | Review |
- SpecHash: value of `last_decision_hash`
- Status Per User Story: NotStarted | InProgress | Blocked | Done
- Blockers: short list (or `None`)

Then below the front matter:

- `## Last Run` (commands run + pass/fail)
- `## Changes Since Last Iteration` (bullets)
- `## Next Steps` (bullets)

## Update Rules

- Every phase must update `progress.md` at the end of its work.
- Any change to `spec.md` requires:
  - update `decision.md` with rationale
  - human approval
  - update `last_decision_hash`
- QA may not edit `spec.md` or production code; QA may only append tests and update `progress.md`.

# Agents

### Phase 1 - Planner Agent

**Output:** `{name_of_feature}/spec.md`

Must include:

- Ordered user stories
- Acceptance criteria (per story)
- Acceptance criteria (feature-wide)
- Constraints (per story)
- Non-goals (per story)
- Definition of Done (per story)
- Parallelization analysis (per story)
- Proposed changes to `docs/casaigo-passport-schema.md` (per story)
- Proposed changes to `docs/casaigo-passport-architecture.md` (per story)

#### Parallelization Analysis

Identify tasks that _could_ run independently

Parallel-safe tasks must:

- Not modify same files
- Not modify shared schema
- Not modify shared global state
- Not depend on unfinished outputs of another task

#### Constraints

- Do not write any code

#### Definition of Done

- `spec.md` created
- Human approval
- `last_decision_hash` generated

#### Stop Conditions

Stop and alert if:

- Acceptance criteria unclear
- Required context missing
- Spec diverges from feature.md
- Sensitive files touched without mention in feature.md

---

### Phase 2 - Implementer Agent

- Implement and test according to assigned user story in `{name_of_feature}/spec.md`
- No minimum test count required.
- Apply QA findings if necessary.
- do not run git add, git commit, git push, or open PRs. Leave all changes uncommitted and stop after code + tests + required state-file updates

#### Constraints

- Can only work on assigned user story in `{name_of_feature}/spec.md`
- Cannot delete or relax QA-appended tests without documenting in `{name_of_feature}/decision.md`
- Cannot modify files outside file touch list
- Cannot modify `{name_of_feature}/spec.md` without human approval
- test names should be descriptive

#### Definition of Done

- Acceptance criteria satisfied
- Tests passing
- No lint errors

#### Stop Conditions

Stop and alert if:

- Failing tests after 2 attempts without clear cause
- Unauthorized file modification
- Attempt to bypass tests or lint
- Schema change without approval
- Deleting QA tests

---

### Phase 3 - QA Agent

**Output:** `{name_of_feature}/spec.md`

#### Required QA Output (scoped to current user story)

- Missing tests list
- At least 3 negative tests
- At least 3 edge cases
- Security checklist
- Performance considerations
- Explicit untested assumptions list
- "How this fails in prod"
- Test matrix
- Issues categorized: Low / Med / High

#### Constraints

QA may read only:

- `{name_of_feature}/feature.md`
- `{name_of_feature}/spec.md`
- Diff
- test names should be descriptive

QA may:

- Append tests only
- Produce structured findings

QA may NOT:

- Modify production code
- Rewrite spec
- Delete tests
- Add/Suggest tests that are outside scope of user story

#### High Severity Issue Definition

Must fix or escalate if:

- Data loss possible
- Auth bypass possible
- Crash in common flow
- Silent corruption possible

#### Medium Severity Issue Definition

Must fix (or explicitly accept via human approval) if any of the following are true:

- Incorrect behavior for a stated acceptance criterion, but not clearly an auth-bypass / data-loss / silent-corruption / common-flow-crash scenario
- Missing validation or error handling that can cause a 4xx/5xx in non-primary but realistic flows
- Incorrect permissions behavior that does not obviously allow unauthorized access, but could produce confusing/incorrect access outcomes (e.g., overly restrictive or inconsistent enforcement)
- Potential data integrity issues that are recoverable (e.g., wrong field values, inconsistent indexes) and likely detectable, but not silent corruption
- Performance regressions that are likely to impact latency or cost in typical usage, but not a clear outage risk
- Tests are present but insufficient to cover edge/negative cases for a core requirement (QA should propose specific tests)
- Logging/observability gaps that would materially hinder debugging or rollback verification

If a Medium issue is not fixed, QA must mark it as `MEDIUM (ACCEPTED)` and include:

- Rationale
- User impact
- How to detect in prod
- Mitigation/rollback note

#### Definition of Done

- Tests passing
- No High Severity or Medium Severity issues

Low issues allowed.

#### Stop Conditions

Stop and alert if:

- QA modifies production code
- QA deletes or rewrites existing tests
- Required output sections missing

---

### Phase 4 - Review Agent

Purpose: final repo-quality pass and PR packaging.

#### Inputs

- `{name_of_feature}/feature.md`
- `{name_of_feature}/spec.md`
- `{name_of_feature}/{task_id}/decision.md`
- `{name_of_feature}/{task_id}/progress.md`
- Diff

#### Responsibilities

- Verify the change matches acceptance criteria in `feature.md` and the implementation constraints in `spec.md`.
- Verify QA rules were followed (QA appended tests only; no QA edits to production code).
- Ensure repo standards are met:
  - formatting/lint passes (run the repo’s standard commands)
  - unit/integration tests pass (run the repo’s standard commands)
  - typecheck/build passes if applicable
- Validate no obvious policy violations:
  - secrets committed
  - disabling auth/permissions checks
  - broadening access unintentionally
  - adding large debug logging or noisy prints
- Confirm state files are complete and consistent:
  - `progress.md` updated and Status is `Done`
  - `decision.md` contains rationale for any non-obvious changes
  - `last_decision_hash` matches approved `spec.md`

#### Required Output

- On User Story Completion
  - create new commit
  - merge branch back into starting branch (NOT MAIN)

- On Feature Completion:
  - PR title
  - PR description with:
    - summary of change
    - list of files changed
    - how to test (commands)
    - QA summary (High/Med/Low counts; note any `MEDIUM (ACCEPTED)`)
    - rollout notes (or link to `release.md` if present)
  - Merge checklist (bullets)

#### Constraints

- Review Agent should not make large architectural changes. If major issues are found, file findings for Fix phase.

#### Stop Conditions

Stop and alert/ask human if:

- Tests or lint cannot be made to pass without non-trivial code changes
- Any High severity QA issue exists
- Any Medium severity issue exists and is not marked `MEDIUM (ACCEPTED)` with required justification
- `spec.md` appears to have changed without human approval / without updating `last_decision_hash`
- State files are missing or incomplete
