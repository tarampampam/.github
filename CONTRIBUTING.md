# Contributing Guidelines

Thank you for your interest in contributing! These guidelines apply to all repositories under 
this account to ensure a consistent and secure contribution process.

## 🧩 General Principles

* Contributions are welcome from everyone
* Keep discussions respectful and constructive
* Follow the repository's existing structure, style, and conventions
* Be concise in commits, pull requests, and comments - clarity over cleverness

## 🛠️ Submitting Changes

1. **Create a branch** for each feature or fix. Branches should be based on `master` and
   named descriptively (e.g., `feature/add-logging`, `fix/typo-readme`)
2. **Make atomic commits**. Each commit should represent a single logical change with a
   clear message
3. **Open a Pull Request (PR)**. PRs should:
   * Describe *why* the change is needed
   * Reference related issues (if applicable)
   * Pass all tests and checks before review
4. **Request a review** if applicable, and respond politely to feedback. Maintainers may ask
   for adjustments before merging

## 🧹 Code Style and Quality

All code should conform to the language's standard formatter and linting tools:

* **Go**: `go fmt`, `go vet`, and the project's configured linters
* **JavaScript / TypeScript**: `prettier` and `eslint`
* **C / C++**: `clang-format`
* **Other languages**: follow community-accepted style guides

Ensure that:

* Code builds successfully without warnings
* No trailing whitespace or unused imports remain
* Tests (if present) pass locally before submitting a PR

## 🧪 Testing

If the project includes tests:

* Add or update tests relevant to your changes
* Ensure tests run cleanly on all supported platforms
* Avoid introducing flaky or environment-dependent tests

## 🗂️ Documentation

* Update documentation, examples, or comments if your change affects usage or behavior
* Keep README or code comments accurate and minimal - avoid duplication across files

## 💬 Final Notes

Contributions of any size - from typo fixes to new features - are appreciated.
Be kind, stay curious, and help keep the ecosystem healthy and consistent.
