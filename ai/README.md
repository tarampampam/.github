# AI Rules

This directory contains shared rules for AI coding agents. These files are **not used by this repository directly** —
they serve as a single source of truth and are referenced from other repositories via public URLs.

## How to Use

### In a project's `AGENTS.md`

Reference the global rules by URL and let agents load them on demand:

```markdown
# AGENTS - Project Rules

> Read this file AND the global rules before making any code changes -
> https://tarampampam.github.io/.github/ai/AGENTS.md (mirror -
> <https://raw.githubusercontent.com/tarampampam/.github/refs/heads/master/ai/AGENTS.md>).

// ...project-specific rules and guidelines...
```

The global `AGENTS.md` itself already instructs agents to load language-specific rules - so you only need to link
to `AGENTS.md`.

### Public URLs

| File          | GitHub Pages                                         | Raw (mirror)                                                                           |
|---------------|------------------------------------------------------|----------------------------------------------------------------------------------------|
| `AGENTS.md`   | https://tarampampam.github.io/.github/ai/AGENTS.md   | https://raw.githubusercontent.com/tarampampam/.github/refs/heads/master/ai/AGENTS.md   |
| `go/RULES.md` | https://tarampampam.github.io/.github/ai/go/RULES.md | https://raw.githubusercontent.com/tarampampam/.github/refs/heads/master/ai/go/RULES.md |
| `ts/RULES.md` | https://tarampampam.github.io/.github/ai/ts/RULES.md | https://raw.githubusercontent.com/tarampampam/.github/refs/heads/master/ai/ts/RULES.md |

Two URLs are provided per file because some agent environments restrict access to certain domains.

## Example of Usage

```diff
diff --git a/.github/copilot-instructions.md b/.github/copilot-instructions.md
--- /dev/null
+++ b/.github/copilot-instructions.md
@@ -0,0 +1,8 @@
+# GitHub Copilot Instructions
+
+At the very start of every session, before doing anything else, fetch and read the following files in order:
+
+1. https://tarampampam.github.io/.github/ai/AGENTS.md (mirror:
+ <https://raw.githubusercontent.com/tarampampam/.github/refs/heads/master/ai/AGENTS.md>) - global agent rules
+2. `AGENTS.md` (repo root) - project-specific rules
+
+Do not make any code changes until the applicable rules above have been loaded and understood.
diff --git a/AGENTS.md b/AGENTS.md
--- /dev/null
+++ b/AGENTS.md
@@ -0,0 +1,132 @@
+# AGENTS - Project Rules
+
+> Read this file AND the global rules before making any code changes -
+> https://tarampampam.github.io/.github/ai/AGENTS.md (mirror -
+> <https://raw.githubusercontent.com/tarampampam/.github/refs/heads/master/ai/AGENTS.md>).
+
+## Instruction Priority
+
+1. This file (`AGENTS.md` in this repository)
+2. Global rules (external URLs)
+3. Other documentation
+
+If rules conflict, follow the highest priority source.
+
+> TODO: write project-specific rules here
+
+## Offline Fallback Rules
+
+> Apply these only if the external rule URLs above are inaccessible. The external rules are authoritative.
+
+### Go
+
+- Wrap errors with context: `fmt.Errorf("operation: %w", err)`. Return sentinel errors directly when they are unlikely.
+- Use `xErr` naming when multiple errors are in scope (e.g. `readErr`, `writeErr`); use `if err := ...; err != nil` for single short-lived errors.
+- Interfaces in the consumer package; keep them minimal; add `var _ Interface = (*Impl)(nil)` compile-time assertions.
+- Exported declarations must have a doc comment starting with the identifier name, ending with a period.
+- No `fmt.Print*` / `print` / `println`; no global variables; no `init()` without justification.
+- Line length ≤ 120 characters.
+- Test files: `package foo_test` (external); one `_test.go` per tested file; both outer and inner `t.Parallel()`; map-based table-driven tests with `give*` / `want*` keys.
+
+### TypeScript / React
+
+- No `// eslint-disable*` or `// @ts-ignore` - fix the root cause instead.
+- No braces omission in conditionals/loops.
+- Prefer `unknown` over `any`; narrow types explicitly.
+- React components: functional only, return type `React.JSX.Element`, PascalCase names.
+- Hooks: `use` prefix, camelCase. Constants/enums: `SNAKE_UPPER_CASE`.
+- CSS modules (`*.module.css`) for scoped styles.
+- Tests: `describe` / `test` blocks; table-driven with `test.each` for repeated logic; cover happy path + key error cases.
diff --git a/README.md b/README.md
--- a/README.md
+++ b/README.md
@@ -205,6 +205,10 @@ The following flags are supported:

+## 🤖 AI Agent Instructions
+
+See [AGENTS.md](AGENTS.md) for detailed guidelines for AI agents working with this repository.
+
 ## License

```
