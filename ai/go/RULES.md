# Rules for Go Code

## Errors

Wrap errors with context when it adds value for debugging - e.g. `fmt.Errorf("operation: %w", err)`.
Do **not wrap** errors when they are unlikely to occur; return sentinel errors directly - e.g.
`if _, err := buf.Write(data); err != nil { return err }`.

Define sentinel errors at package level: `var ErrNotFound = errors.New("not found")` when they are expected to be
checked by callers. Otherwise, return wrapped errors with context.

### Error variable naming

Prefer `xErr` where `x` is a short alias for the operation, rather than reusing a single `err` variable that gets
overwritten, especially when descriptive error variables when multiple errors are in scope. This makes it harder
to accidentally check the wrong error.

```go
ping, pingErr := some.Ping(ctx)
if pingErr != nil { ... }

data, decodeErr := json.Unmarshal(...)
if decodeErr != nil { ... }

n, wErr := buf.Write(b)
if wErr != nil { ... }
```

The exception is an inline `if` with a short-lived error that never needs to be referenced again - use the standard
short form there:

```go
if err := DoSomething(ctx); err != nil { ... }
```

This is a style convention, not a hard lint rule.

## Interfaces

* Define interfaces in the consumer package.
* Keep them minimal.
* Add compile-time assertions: `var _ Interface = (*Impl)(nil) // ensure Impl implements Interface`

## Code comments

**Exported declarations** must have a doc comment that:

- Starts with the name of the declared identifier (standard Go godoc convention).
- Describes the primary purpose - including edge cases and the motivation behind non-obvious behavior.
- Omits parameter-by-parameter or error-by-error enumeration; only mention a specific error or param if it is
  genuinely surprising or load-bearing for a caller.
- Ends with a period.
- Uses only technical English.
- Uses only a standard short hyphen (`-`) where a separator is needed - never an em dash, never an arrow (`←` or `→`).

```go
// Queue enqueues the item with the given ID for processing. It returns an error if the item cannot be enqueued due to
// transient issues (e.g. DB failure). The method is idempotent and safe to call multiple times with the same ID.
func (h *Handler) Queue(ctx context.Context, id string) error { ... }

// ErrNotFound is returned by DB query methods when the requested record does not exist.
var ErrNotFound = errors.New("not found")

// Update holds an optional field update. A zero-value Update carries no change and is ignored by query builders - use
// NewUpdate to set an explicit value.
type Update[T any] struct { ... }
```

**Inline comments** inside function bodies:

- Use only when the code is genuinely non-obvious; omit them when the code is self-explanatory.
- Explain *why*, not *what* - a reader can see what the code does; they cannot always see why it was written that way.
- Start with a lowercase letter and end without a period.
- Use only a standard short hyphen (`-`) where needed - never an em dash, never an arrow (`←` or `→`).

```go
// not found is a valid outcome here - the caller treats absence as permission to proceed
if errors.Is(err, db.ErrNotFound) {
  return nil
}

// use WithoutCancel so the cleanup activity survives context cancellation triggered by the parent workflow
ctx = context.WithoutCancel(ctx)
```

**What not to comment**: obvious assignments, type conversions, or standard library calls; information the function
or variable name already conveys; every branch of a switch or if-chain unless a specific branch has a non-obvious
reason.

## Linter Rules

* No `fmt.Print*`, `print`, `println`
* No global variables
* No `init()` without justification
* Line length ≤ 120
* Import order: std → external → internal

For more specific rules, refer to the project's linter configuration (e.g. `.golangci.yml`).

## Testing

### Structure

- External test package: `package foo_test`, not `package foo` (prevents accidental access to unexported identifiers).
- One `_test.go` file per tested type - e.g. `utils.go` tests in `utils_test.go`, `exec.go` tests in `exec_test.go`.
- Both the outer `t.Parallel()` and the inner `t.Parallel()` (inside `t.Run`) are strictly recommended.
- Map-based table-driven tests - maps (not slices) provide random ordering which surfaces ordering-dependent bugs.
- The map key is the test case name; the value is an anonymous struct with inputs (`give*`) and expectations
  (`want*` / `checkErr`).
- Prefer helper functions for assertions over third-party libraries (when possible) - this keeps test dependencies
  minimal. **Use `github.com/stretchr/testify` or similar libraries if they are already widely used in the project**.

### Unit test pattern

Follow the existing testing style in the project. Use the pattern below only if no clear pattern exists.

```go
package mypkg_test

import (
  "testing"

  // ... other imports
)

func TestSomething(t *testing.T) {
  t.Parallel()

  for name, tc := range map[string]struct {
    giveMock  *mockDep   // give* prefix for all inputs
    giveInput SomeInput

    wantResult      SomeResult // want* prefix for all expectations
    wantErrContains string     // non-empty means the test expects an error
    // or checkErr func(*testing.T, error) // custom error checker function
  }{
    "success": {
      giveMock:   &mockDep{...},
      giveInput:  SomeInput{...},
      wantResult: SomeResult{...},
    },
    "dependency error": {
      giveMock:        &mockDep{err: assert.AnError},
      giveInput:       SomeInput{...},
      wantErrContains: "some operation",
    },
  } {
    t.Run(name, func(t *testing.T) {
      t.Parallel()

      { // setup defaults for optional mocks
        if tc.giveMock == nil {
          tc.giveMock = new(mockDep)
        }

        tc.giveMock.t = t
      }

      result, err := mypkg.New(tc.giveMock).Do(t.Context(), tc.giveInput)

      if tc.wantErrContains != "" {
        assertErrorContains(t, err, tc.wantErrContains)

        return
      }

      assertNoError(t, err)
      assertEqual(t, tc.wantResult, result)
    })
  }
}

func TestSomething_String(t *testing.T) {
  t.Parallel()

  for name, tt := range map[string]struct {
    giveFormat mypkg.Format
    wantString string
  }{
    "json":      {giveFormat: mypkg.JSONFormat, wantString: "json"},
    "console":   {giveFormat: mypkg.ConsoleFormat, wantString: "console"},
    "<unknown>": {giveFormat: mypkg.Format(255), wantString: "format(255)"},
  } {
    t.Run(name, func(t *testing.T) {
      // there is no `t.Parallel()` here because the test cases are lightweight
      assertEqual(t, tt.wantString, tt.giveFormat.String())
    })
  }
}

// The mock structs example below shows how to implement a mock for an interface dependency without using a mocking
// framework. The mock struct embeds the interface to satisfy it, and implements only the methods that are relevant
// for the test cases.

// --------------------------------------------------------------------------------------------------------------------

type mockDep struct {
  SomeDependencyInterface // embed to satisfy the full interface; un-implemented methods panic

  t   *testing.T
  err error           // error to return
  wantInput SomeInput // want* fields for expected arguments validated inside method bodies
}

func (m *mockDep) Do(_ context.Context, in SomeInput) (SomeResult, error) {
  if m.wantInput != (SomeInput{}) {
    assertEqual(m.t, m.wantInput, in)
  }

  return SomeResult{}, m.err
}

// Assertions functions example below shows how to implement custom assertion helpers for tests without using a
// third-party library.

// --------------------------------------------------------------------------------------------------------------------

// assertNoError fails the test if err is not nil, indicating an unexpected error occurred.
func assertNoError(t *testing.T, err error) {
  t.Helper()

  if err != nil {
    t.Errorf("unexpected error: %v", err)
  }
}

// assertErrorContains checks if the given error message contains the expected substring.
func assertErrorContains(t *testing.T, err error, substr string) {
  t.Helper()

  if err == nil {
    t.Errorf("expected error, got nil")

    return
  }

  assertContains(t, err.Error(), substr)
}

// assertContains checks if the given string contains at least one of the expected substrings.
func assertContains(t *testing.T, got string, want ...string) {
  t.Helper()

  for _, w := range want {
    if !strings.Contains(got, w) {
      t.Errorf("expected %q to contain %q", got, w)
    }
  }
}

// assertEqual checks if two values of a comparable type are equal.
// If they are not, the test fails and includes an optional message prefix.
func assertEqual[T comparable](t *testing.T, got, want T, msgPrefix ...string) {
  t.Helper()

  var p string

  if len(msgPrefix) > 0 {
    p = msgPrefix[0] + ": "
  }

  if got != want {
    t.Errorf("%sgot %v, want %v", p, got, want)
  }
}
```

**Error assertion style** - use `wantErrContains string` (substring match) as the default. Switch to
`checkErr func(*testing.T, error)` when the test must assert a specific sentinel via `errors.Is`, or needs to check
multiple properties of the error at once:

```go
checkErr: func(t *testing.T, err error) {
  assert.ErrorIs(t, err, mypkg.ErrNotFound)
  assert.ErrorContains(t, err, "user JohnDoe not found")
},
```

**When not to use a map**: if a test exercises timing, channel behavior, or other sequential logic that cannot be
cleanly expressed as independent table cases, use plain named `t.Run` subtests instead.

## Principles

- Test behavior, not implementation.
- Cover happy path + key failures.
- Avoid over-testing trivial behavior.
- Do not aim for 100% coverage.
