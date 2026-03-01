# Ecosystem Config Mappings

Use this reference during Phase 3 (Active Probing) to check which config files contain settings that might make context file instructions redundant.

## Ecosystem Detection

Detect the primary ecosystem by checking for these files (in priority order):

| Ecosystem | Detection Files |
|-----------|----------------|
| **Node.js / TypeScript** | `package.json`, `tsconfig.json`, `node_modules/` |
| **Python** | `pyproject.toml`, `setup.py`, `setup.cfg`, `requirements.txt`, `Pipfile` |
| **Java** | `pom.xml`, `build.gradle`, `build.gradle.kts` |
| **General** | Always checked regardless of ecosystem |

A repo may match multiple ecosystems (e.g., monorepo with Node.js + Python). Check all matching ecosystems.

---

## Node.js / TypeScript

### Config Files to Check

| Config File | Glob Pattern | What It Configures |
|-------------|-------------|-------------------|
| `package.json` | `package.json` | Dependencies, scripts, engines, workspaces |
| `tsconfig.json` | `tsconfig*.json` | TypeScript compiler options |
| ESLint | `.eslintrc*`, `eslint.config.*`, `.eslintignore` | Linting rules |
| Prettier | `.prettierrc*`, `prettier.config.*` | Formatting rules |
| Jest | `jest.config.*`, `package.json#jest` | Test framework config |
| Vitest | `vitest.config.*`, `vite.config.*` | Test framework config |
| Husky | `.husky/*` | Git hooks |
| lint-staged | `.lintstagedrc*`, `package.json#lint-staged` | Pre-commit lint config |
| `.npmrc` | `.npmrc` | npm configuration |
| `.nvmrc` / `.node-version` | `.nvmrc`, `.node-version` | Node version |
| `.editorconfig` | `.editorconfig` | Editor settings |

### Common Instruction-to-Config Mappings

| Instruction Pattern | Config File | Config Key/Setting |
|--------------------|-------------|-------------------|
| "Use single/double quotes" | `.prettierrc*` | `singleQuote` |
| "Use semicolons / no semicolons" | `.prettierrc*` | `semi` |
| "Max line length is N" | `.prettierrc*` | `printWidth` |
| "Use tabs / spaces for indentation" | `.prettierrc*` | `useTabs`, `tabWidth` |
| "Trailing commas" | `.prettierrc*` | `trailingComma` |
| "Use camelCase for variables" | `.eslintrc*` | `@typescript-eslint/naming-convention` |
| "No unused variables" | `.eslintrc*` | `@typescript-eslint/no-unused-vars` |
| "No console.log" | `.eslintrc*` | `no-console` |
| "Strict TypeScript" | `tsconfig.json` | `strict: true` |
| "Use path aliases" | `tsconfig.json` | `paths` |
| "Node version 18/20/22" | `.nvmrc`, `.node-version`, `package.json#engines` | version string |
| "Use Jest for testing" | `package.json#devDependencies` | `jest` present |
| "Run tests before commit" | `.husky/pre-commit` | test command present |
| "Run lint before commit" | `.husky/pre-commit` or `.lintstagedrc*` | lint command present |

---

## Python

### Config Files to Check

| Config File | Glob Pattern | What It Configures |
|-------------|-------------|-------------------|
| `pyproject.toml` | `pyproject.toml` | Build system, dependencies, tool config |
| `setup.cfg` | `setup.cfg` | Package metadata, tool config |
| `setup.py` | `setup.py` | Package setup (legacy) |
| Flake8 | `.flake8`, `setup.cfg#flake8`, `pyproject.toml#tool.flake8` | Linting |
| Pylint | `.pylintrc`, `pyproject.toml#tool.pylint` | Linting |
| Ruff | `ruff.toml`, `pyproject.toml#tool.ruff` | Linting + formatting |
| Black | `pyproject.toml#tool.black` | Formatting |
| isort | `pyproject.toml#tool.isort`, `.isort.cfg` | Import sorting |
| pytest | `pytest.ini`, `pyproject.toml#tool.pytest`, `setup.cfg#tool:pytest` | Test config |
| mypy | `mypy.ini`, `pyproject.toml#tool.mypy` | Type checking |
| tox | `tox.ini` | Test environments |
| pre-commit | `.pre-commit-config.yaml` | Git hooks |

### Common Instruction-to-Config Mappings

| Instruction Pattern | Config File | Config Key/Setting |
|--------------------|-------------|-------------------|
| "Max line length is N" | `.flake8` / `pyproject.toml` | `max-line-length` / `line-length` |
| "Use Black for formatting" | `pyproject.toml#tool.black` | presence |
| "Sort imports with isort" | `pyproject.toml#tool.isort` | presence |
| "Use type hints" | `mypy.ini` / `pyproject.toml#tool.mypy` | `strict = true` |
| "Use pytest for testing" | `pyproject.toml#dependencies` or `#dev-dependencies` | `pytest` present |
| "Python version 3.x" | `pyproject.toml#project.requires-python` | version string |
| "Use snake_case" | `.pylintrc` / Ruff config | naming convention rules |

---

## Java

### Config Files to Check

| Config File | Glob Pattern | What It Configures |
|-------------|-------------|-------------------|
| Maven | `pom.xml` | Dependencies, plugins, build config |
| Gradle | `build.gradle`, `build.gradle.kts` | Build config |
| Checkstyle | `checkstyle.xml`, `config/checkstyle/*` | Code style |
| SpotBugs | `spotbugs-exclude.xml` | Bug detection |
| `.editorconfig` | `.editorconfig` | Editor settings |
| PMD | `pmd-ruleset.xml` | Static analysis |

### Common Instruction-to-Config Mappings

| Instruction Pattern | Config File | Config Key/Setting |
|--------------------|-------------|-------------------|
| "Use Java N" | `pom.xml` | `maven.compiler.source/target` |
| "Follow Google style" | `checkstyle.xml` | `google_checks.xml` reference |
| "Max line length N" | `checkstyle.xml` | `LineLength` module |
| "Use JUnit 5" | `pom.xml` / `build.gradle` | `junit-jupiter` dependency |
| "Use Mockito" | `pom.xml` / `build.gradle` | `mockito-core` dependency |

---

## General (All Ecosystems)

### Config Files to Check

| Config File | Glob Pattern | What It Configures |
|-------------|-------------|-------------------|
| README | `README.md`, `README.rst`, `README.txt` | Project documentation |
| CI/CD | `.github/workflows/*.yml`, `.gitlab-ci.yml`, `Jenkinsfile` | Build/test/deploy pipelines |
| `.editorconfig` | `.editorconfig` | Cross-editor settings |
| `.gitignore` | `.gitignore` | Ignored files |
| Docker | `Dockerfile`, `docker-compose.yml`, `compose.yml` | Container config |
| Makefile | `Makefile`, `makefile` | Build targets |

### Common Instruction-to-Config Mappings

| Instruction Pattern | Config File | Config Key/Setting |
|--------------------|-------------|-------------------|
| "Never commit .env files" | `.gitignore` | `.env` entry present |
| "Run tests in CI" | `.github/workflows/*.yml` | test step present |
| "Use indentation of N spaces" | `.editorconfig` | `indent_size` |
| "Use UTF-8 encoding" | `.editorconfig` | `charset = utf-8` |
| "Trim trailing whitespace" | `.editorconfig` | `trim_trailing_whitespace = true` |
| "Branch naming convention" | `.github/workflows/*.yml` | branch filter patterns |
