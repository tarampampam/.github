# Rules for TypeScript and JavaScript Code

## Commands

Usual commands for development and CI:

```bash
npm run build    # build for production (outputs to ../dist/)
npm run lint     # run all linters (tsc + eslint)
npm run fmt      # format the source code (prettier + eslint --fix)
npm run test     # run unit tests (vitest)
npm run generate # run codegen tasks
```

Check the actual `"scripts"` values in `package.json`.

## What not to do

- **Don't edit `*.gen.*` files** - they are overwritten on the next `npm run generate` run.
- **Don't use `// eslint-disable-line`, `// eslint-disable-next-line`, `/* eslint-disable */`, or `// @ts-ignore`** -
  fix the underlying issue instead of suppressing it.
- **Don't omit braces for conditionals and loops**.
- **Don't make unrequested changes** to files outside the scope of the current task.

### LLM-friendly Docs

Prefer official documentation. If LLM-optimized docs are available, use them as the primary source:

- **Mantine** – https://mantine.dev/llms.txt (full: https://mantine.dev/llms-full.txt)
- **React** – https://react.dev/llms.txt
- **Vite** – https://vite.dev/llms.txt
- **Vitest** – https://vitest.dev/llms.txt

For other libraries, check if they have LLM-optimized docs or use the regular docs.

## Conventions

Follow the current code style and conventions as much as possible. If you think a new pattern or convention is
needed, report it.

## Imports

- Use the alias-based imports (e.g. `~/shared`) over relative (`../`) ones, when the project is configured for it.
- Barrel exports via `index.ts` by default.

## TypeScript

Strict mode on with extras: `noUnusedLocals`, `noUnusedParameters`, `noUncheckedIndexedAccess`, `noImplicitOverride`,
`verbatimModuleSyntax`. The full config is in `tsconfig.json`.

### Types

- Prefer `unknown` over `any`.
- Narrow types explicitly.

### Errors

- Never swallow errors.
- Always handle async failures.

## TypeScript + React

- Return type: `React.JSX.Element` when possible.
- Use functional components only when possible.
- Naming: PascalCase for components, camelCase for hooks (`use` prefix), SNAKE_UPPER_CASE for constants and enums.
- CSS modules (`*.module.css`) for scoped styles, global CSS only when necessary.
- Use React context/providers for global state, keep state local by default.

### Code Organization

- Keep components small and focused.
- Extract logic into hooks.
- Avoid large files (>300 lines) unless justified.

## Testing

### Mocking

Prefer real implementations, but use mocks when they simplify tests or improve determinism. A mock is only
justified when it:

- Crosses a real network boundary
- Depends on browser APIs unavailable in the test environment
- Introduces non-determinism (timers, random values, dates)
- Makes the test setup disproportionately complex

Avoid adding new `devDependencies` unless strictly necessary.

### Test structure

Prefer `describe` / `test` blocks (over `it`):

```ts
describe('...', () => {
  test('...', () => {
    // ...
  })
})
```

Prefer table-driven tests for repeated logic, where the same test is run with different inputs/expected values:

```ts
test.each([
  { input: 1, expected: 2 },
  { input: 2, expected: 4 },
])('doubles $input to $expected', ({ input, expected }) => {
  expect(double(input)).toBe(expected)
})
```

### Scope and coverage

- Cover the **happy path** and **a few meaningful error cases** - not every branch
- Do not write tests for trivial UI state rather than meaningful behavior (color changes, visibility toggles,
  loading spinners, etc.)
- The codebase is actively evolving; avoid over-specifying behavior that may change
- Tests should be concise and signal intent, not chase 100% coverage

### Failing tests and business logic validation

When writing tests, treat them as a specification of the expected business logic - not just a mechanical coverage
exercise. Before finalizing any test, reason explicitly about whether the assertion reflects the **correct intended
behavior**, not just the current behavior of the code.

When a test fails after being written, **do not default to fixing the test**. Instead, investigate the failure to
determine whether it indicates a bug in the implementation or an issue with the test itself. Only fix the test if
you are fully confident the source code is correct and the test itself is the mistake.

## Opportunistic code review

Whenever you read existing source files - to write tests, implement a feature, refactor, or for any other reason -
treat every file you open as an implicit code review target.

Assess the code for **production-readiness**: logic bugs, security issues, copy/paste errors - only the things that
would be problematic in production, don't be noisy about minor style inconsistencies or non-ideal patterns that
don't cause actual issues. Only report clear, actionable findings - do not speculate.

> Do not silently apply fixes found during review unless they are trivially safe and directly related to the primary
> task. Propose non-trivial fixes in the notes and apply only after confirmation.
