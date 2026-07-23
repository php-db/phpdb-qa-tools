# Migrating from laminas-coding-standard

Repository-by-repository, in dependency order, using `phpdb-validator` as the template.

## Prerequisites

Install the Mago binary once per machine (see the [README](../README.md#prerequisite-install-mago)).

## Steps

### 1. Swap dev dependencies

```sh
composer remove --dev laminas/laminas-coding-standard
# also remove psalm/psalm, vimeo/psalm, or phpstan/phpstan if present
composer require --dev php-db/phpdb-qa-tools
```

### 2. Remove phpcs artifacts

```sh
rm -f phpcs.xml phpcs.xml.dist .phpcs-cache
```

Also remove any `psalm.xml` / `phpstan.neon` and their baselines.

### 3. Add the minimal mago.toml

```toml
extends = "vendor/php-db/phpdb-qa-tools/mago.toml"
php-version = "8.2.0"

[source]
paths = ["src", "test"]
includes = ["vendor"]
```

Append repository-specific `[analyzer]` settings (e.g. `class-initializers`) as needed.

### 4. Adopt the PHPUnit baseline

```sh
cp vendor/php-db/phpdb-qa-tools/templates/phpunit.xml.dist .
```

Adjust test-suite paths only if the repository cannot follow the standard
`test/unit` / `test/integration` layout.

### 5. Update composer scripts

Adopt the standard `check` / `cs-check` / `cs-fix` / `static-analysis` / `test`
scripts (see the README).

### 6. Reformat in a single isolated commit

```sh
composer cs-fix
git add -A
git commit -m "Apply mago formatting"
git rev-parse HEAD >> .git-blame-ignore-revs
git add .git-blame-ignore-revs
git commit -m "Ignore mago reformat in git blame"
```

Keeping the mechanical reformat isolated (and recorded in `.git-blame-ignore-revs`)
keeps `git blame` useful.

### 7. Triage remaining findings

Fix or explicitly suppress remaining lint/analyzer findings in follow-up commits.
Large repositories may temporarily relax specific rules in their local `mago.toml`
(child settings override the base) — open a tracking issue to remove the relaxation.

### 8. Update CI

Replace the phpcs/psalm workflow steps with the standard QA workflow (see the README).
