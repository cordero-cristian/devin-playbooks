Playbook: Run the Spec Kit (SDD) Workflow

## Overview
Take a feature idea through the full GitHub Spec Kit Spec-Driven Development cycle for any Spec Kit project: **specify → clarify → plan → tasks → analyze → implement**, with review gates between phases. Spec Kit is installed under `.specify/`, and each command is exposed either as an agent skill (e.g. `speckit-specify`) or as a slash command, depending on the integration the project was initialized with. When enabled, git hooks (feature branch creation, auto-commits, agent-context refresh) fire automatically around each phase per `.specify/extensions.yml`. Feature artifacts land in `specs/NNN-feature-name/`.

## What's Needed From User
Core inputs (required):
- **Feature description** — one or more sentences describing what to build. This becomes the `$ARGUMENTS` input to every phase.
- **Acceptance criteria** — the concrete, verifiable conditions that define "done" for this feature. These anchor the spec, are validated by `analyze`, and are what implementation must satisfy.

Additional context (optional):
- **`{{ADDITIONAL_CONTEXT}}`** — any extra context the user wants to provide: constraints, scope hints (e.g. `full`, `backend-only`, `frontend-only`), relevant files/links/prior art, non-goals, or whether to stop after `plan`/`tasks` for review vs. run straight through `implement`. Fold this into the feature description passed to each phase.

## Procedure
1. Verify the Spec Kit install and prepare the environment: confirm `.specify/` exists (and `.specify/extensions.yml` if extensions are used) and that the project's Spec Kit commands are available; stop and tell the user if the project is not Spec Kit-initialized. Confirm a project constitution exists (e.g. `.specify/memory/constitution.md`) — if it does not, run the `constitution` command first (or ask the user for principles). Install dependencies and run the project's build/setup so environment issues surface before implementation (check README/AGENTS.md/contributing docs for the exact commands).
2. Run the `specify` command with the feature description as `$ARGUMENTS`, including the acceptance criteria and any `{{ADDITIONAL_CONTEXT}}` so they are captured in the spec. If the git extension is enabled, the `before_specify` hook creates a numbered feature branch (`NNN-feature-name`); the phase writes `specs/NNN-feature-name/spec.md`. Confirm the branch (or current branch) and spec were created, and that the acceptance criteria are reflected in the spec.
3. **Required — clarify**: run the `clarify` command to resolve underspecified areas (up to 5 targeted questions encoded back into the spec). Auto-answer questions you can confidently infer from the description, acceptance criteria, and codebase, and **record each inferred answer as an explicit assumption in the spec**; surface only the genuinely ambiguous questions to the user and wait for their answers. Do not proceed to planning until `spec.md` has no unresolved `[NEEDS CLARIFICATION]` markers.
4. **Review gate — spec**: share a concise summary of `spec.md`, the clarifications, and any recorded assumptions with the user and get approval before planning. Abort if rejected.
5. Run the `plan` command with the same description. It generates `plan.md` plus design artifacts (commonly `research.md`, `data-model.md`, `contracts/`, `quickstart.md`) and, when the agent-context hook is enabled, refreshes the agent context file (e.g. `AGENTS.md`). Confirm the artifacts exist.
6. **Review gate — plan**: share a concise summary of `plan.md` and design artifacts; get approval before generating tasks. Abort if rejected.
7. Run the `tasks` command to generate a dependency-ordered `tasks.md` for the feature.
8. **Required — analyze**: run the `analyze` command for a non-destructive cross-artifact consistency check across `spec.md`, `plan.md`, and `tasks.md`. Resolve any high-severity findings (update the relevant artifact and re-run) before implementing. Optionally run `checklist` to unit-test requirements quality.
9. Run the `implement` command to execute `tasks.md` in dependency order. When git auto-commit hooks are enabled, let them record progress; do not fight them.
10. Validate: run the project's test/lint/build suite (find the exact commands in README/AGENTS.md/CONTRIBUTING or CI config) and fix failures until green, then confirm every acceptance criterion is satisfied by the implementation.
11. Open a PR from the feature branch, referencing the `specs/NNN-feature-name/` artifacts, and share the link. Do not squash or discard the Spec Kit auto-commits.

## Specifications
- A `specs/NNN-feature-name/` directory exists containing at least `spec.md`, `plan.md`, and `tasks.md`.
- `clarify` was run and `spec.md` has no unresolved `[NEEDS CLARIFICATION]` markers.
- `analyze` was run and reports no unresolved high-severity inconsistencies.
- Implementation matches `tasks.md` and satisfies every acceptance criterion; work happens on the Spec Kit feature branch, not the default branch.
- Validation: the project's test/lint/build suite passes locally and CI is green on the PR.
- Deliverable: a PR into the working base branch with the full Spec Kit artifact set plus implementation, and a summary linking the spec/plan/tasks.

## Advice and Pointers
- If a review gate is rejected, `analyze` surfaces high-severity findings you cannot resolve within the current artifacts, or a phase fails repeatedly, stop and escalate to the user with a concise summary rather than forcing the workflow forward.
- Command invocation depends on the integration: some projects expose the phases as agent skills (e.g. `speckit-specify`), others as slash commands (e.g. `/speckit.specify`). Use whichever the project provides; the phase order is identical.
- Run the phases strictly in order; each consumes the previous phase's artifacts. Do not skip `specify`/`plan`.
- Pass the same feature description as `$ARGUMENTS` to each phase so branch/feature resolution stays consistent.
- If the git extension is enabled it auto-commits before/after phases (see `.specify/extensions/git/git-config.yml`); expect several `[Spec Kit] ...` commits — this is normal.
- Feature/branch numbering is typically `sequential` (`001`, `002`, ...), continuing from existing `specs/` directories. When several features are developed concurrently (e.g. parallel sessions off the same base), each independently picks the same next number and collides — run features sequentially, or renumber the branch/`specs/NNN-*` dir before merging.
- `constitution` is a one-time, project-level step. Only run it if the project has no constitution yet, or the user explicitly wants to change project principles.
- If phase resolution seems off, inspect the current feature dir and available docs with `.specify/scripts/bash/check-prerequisites.sh --json` (or the PowerShell equivalent under `.specify/scripts/`).

## Forbidden Actions
- Do not run `implement` before `tasks.md` exists, or `plan` before an approved `spec.md`.
- Do not skip the `clarify` or `analyze` steps — both are mandatory in this workflow.
- Do not commit secrets or credentials, and do not run live/production flows unless the user explicitly requests it.
- Do not hand-edit generated artifacts to force analyze/checks to pass — fix the underlying spec/plan/tasks and re-run the relevant phase.
- Do not disable or bypass the Spec Kit git hooks, and do not commit directly to the default branch.
