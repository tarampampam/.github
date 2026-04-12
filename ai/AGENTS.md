# AGENTS - Global Rules

> Read this file before making any code changes.

This document defines shared **agent behavior** applicable to all code changes and different projects, regardless of
language or domain. It complements language-specific and project-specific conventions and patterns, which should be
followed when applicable.

## TL;DR

- Follow existing patterns
- Make minimal changes
- Do not guess - ask when unsure
- Always lint, test, and self-review
- Keep `README.md` up to date when changes are user-facing

## Language-Specific Rules (Mandatory)

Apply additional rules depending on the file type you are working with. Use any accessible URL - some environments
restrict domains.

### Go files

If you are writing or modifying any `.go` file (including `*_test.go`), you MUST read Go-specific rules before
making changes - https://tarampampam.github.io/.github/ai/go/RULES.md (mirror -
<https://raw.githubusercontent.com/tarampampam/.github/refs/heads/master/ai/go/RULES.md>).

### TypeScript / JavaScript files

If you are writing or modifying any `.ts`, `.tsx`, `.js`, `.jsx` file (including tests), you MUST read TS/JS-specific
rules before making changes - https://tarampampam.github.io/.github/ai/ts/RULES.md (mirror -
<https://raw.githubusercontent.com/tarampampam/.github/refs/heads/master/ai/ts/RULES.md>).

### Important

- Do NOT assume language-specific rules - always try to load them first
- Do NOT silently ignore missing rules
- If rules conflict with existing code, follow existing code

## Core Principles

- Prefer **existing patterns** over inventing new ones.
- Make **small, targeted changes** only.
- Do not refactor unrelated code.

Avoid introducing new abstractions unless the existing patterns are clearly insufficient, or the change significantly
improves readability, testability, or correctness. If unsure - ask.

When modifying code that affects multiple files:

- Identify all impacted files before making changes
- Update them consistently
- Ensure interfaces and tests remain valid

Prefer minimal diffs over full rewrites.

## What Not to Do

- Do not use global state (loggers, configs, etc.) when possible or without a clear precedent.
- Do not modify generated files (e.g. `*.gen.*`).
- Do not introduce new patterns without explicit approval (you may suggest them, but do not implement immediately).
- Do not suppress linter errors (`//nolint`, `eslint-disable`) without strong justification.
- Do not make changes outside the task scope.

## Agent workflow

After writing or modifying any file, always run the following steps in order before considering the task
complete.

1. **Read existing package/module files**: before writing or modifying any file, read similar code in the same
   package/module files - the one most directly analogous to what you are about to write. One or two files is
   sufficient; do not read all files in the package/module. Use them as the authoritative style reference for that
   package. Do not invent a new pattern when an existing one is already established in the package/module.
2. **Lint**: if applicable, run the linter and fix all errors and warnings.
3. **Test**: if applicable, run all tests and fix any failures.
4. **Self-review**: after all lint and test steps pass (if applicable), review the code you wrote against the
   checklist below. Re-read only the parts you modified if necessary to validate correctness.
   - [ ] Logic errors: off-by-one, wrong operator, inverted condition, unreachable branch.
   - [ ] Concurrency bugs: missing mutex, shared state access without lock, deadlock, etc.
   - [ ] Error handling: silently swallowed errors, missing errors checking or comparison, etc.
   - [ ] Security: unsanitized input in SQL/shell, credentials or secrets in code, overly permissive auth checks, etc.
   - [ ] Pre-existing bugs: report only what you naturally encountered while implementing the task - do not actively
     inspect surrounding code for unrelated issues.
5. **Update `README.md`**: if the change is user-facing, update the `README.md` file in the repo root directory to
   reflect it. This includes (but is not limited to):
   - Adding, changing, or removing a feature, behavior, or option.
   - Changes to configuration, environment variables, flags, or defaults.
   - Changes to CLI usage, API, or public interfaces.
   - Deprecations or breaking changes.
   - If no `README.md` exists in the repository, skip this step. If the change is purely internal (e.g. refactoring,
     tests, CI tweaks) and has no impact on how users interact with the project, skip this step.
6. **Update `AGENTS.md`** (optional): update the project-level `AGENTS.md` (usually at the repo root) to reflect
   anything a future agent needs to know to work effectively in this codebase.

Do not present work as finished until all steps above pass without errors or warnings.
Do not fix issues outside the current task scope without asking first.

## Agent behavior and autonomy

**Ask before acting when**:

- The task requires a design decision that isn't covered by existing conventions (e.g. a new abstraction, a new
  package/module, changes to the DB schema or API spec).
- The correct approach is ambiguous and two or more reasonable implementations exist.
- A change would affect generated files, migrations, API specs, data formats, etc. - these have a broad downstream
  impact and must be explicitly approved before running codegen.

One well-placed question saves more time than a fully-written but wrong implementation. Do not guess silently and
then discard the work - ask a focused question first.

## Accuracy

- Do not assume APIs, functions, or types exist without evidence.
- Do not invent behavior not present in the codebase.
- If unsure - ask instead of guessing.

## Priority Order

When rules conflict:

1. Existing code in the same package/module
2. Project-level `AGENTS.md`
3. This file
4. Other documents

Existing code takes precedence only if it does not violate core principles from this document.
