# phpdb-qa-tools

Shared [Mago](https://mago.carthage.software/) + PHPUnit configuration for php-db components.

This package replaces `laminas/laminas-coding-standard` (PHP_CodeSniffer) across the php-db
organization with a single toolchain covering linting, formatting, and static analysis,
configured once and inherited everywhere.

## Prerequisite: install Mago

Mago is a self-contained static binary and is **not** delivered through Composer.
Install it once per machine:

```sh
curl --proto '=https' --tlsv1.2 -sSf https://carthage.software/mago.sh | bash
# or
brew install mago
# or
cargo install mago
```

The shared configuration pins the expected Mago version, so a stale or too-new
binary is flagged immediately.

## Installation

```sh
composer require --dev php-db/phpdb-qa-tools
```

## Usage

### 1. Mago

Create a `mago.toml` in your repository root that extends the shared base and
adds only the project-specific facts:

```toml
extends = "vendor/php-db/phpdb-qa-tools/mago.toml"
php-version = "8.2.0"

[source]
paths = ["src", "test"]
includes = ["vendor"]

[analyzer]
class-initializers = [
    "PhpDb\\ResultSet\\AbstractResultSet::initialize",
]
```

Merge semantics: nested tables merge deeply, arrays concatenate (parent first),
and child scalars win — so you can tighten or relax individual rules locally
without forking the whole standard.

### 2. PHPUnit

Copy the strict baseline into your repository (PHPUnit has no config inheritance):

```sh
cp vendor/php-db/phpdb-qa-tools/templates/phpunit.xml.dist .
```

Standard test-suite layout: `test/unit`, `test/integration`, `test/asset`.

### 3. Composer scripts

Add the standard scripts to your `composer.json`:

```json
{
    "scripts": {
        "check": ["@cs-check", "@static-analysis", "@test"],
        "cs-check": ["mago format --check", "mago lint"],
        "cs-fix": ["mago format", "mago lint --fix"],
        "static-analysis": "mago analyze",
        "test": "phpunit --colors=always --testsuite \"unit test\"",
        "test-integration": "phpunit --colors=always --testsuite \"integration test\"",
        "test-coverage": "phpunit --colors=always --coverage-clover clover.xml"
    }
}
```

### 4. CI

This repository ships a reusable QA workflow
([`.github/workflows/continuous-integration.yml`](.github/workflows/continuous-integration.yml)).
A consuming repository's entire CI file becomes:

```yaml
# .github/workflows/ci.yml
name: CI

on:
  push:
  pull_request:

jobs:
  qa:
    uses: php-db/phpdb-qa-tools/.github/workflows/continuous-integration.yml@main
    with:
      php-versions: '["8.2", "8.3", "8.4", "8.5"]'
      run-integration: false
```

The workflow installs Mago, then runs `composer cs-check`, `composer static-analysis`,
and `composer test` across the PHP version matrix. Pin `@main` to a tag (e.g. `@1.0.0`)
once released.

## Documentation

- [Migration guide](docs/migration.md) — moving a repository off laminas-coding-standard.
- [Rule rationale](docs/rules.md) — why the non-default choices are what they are.
- [Workflow architecture](docs/workflow-architecture.md) — planned job-split design for DB-backed integration tests, Codecov, and Infection.

## License

BSD-3-Clause. See [LICENSE](LICENSE).
