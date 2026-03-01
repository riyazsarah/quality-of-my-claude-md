# Examples

## Format 1: Score Card

```
## Context File Quality Report
Overall Grade: B (82/100)

| Dimension        | Score | Notes                                |
|------------------|-------|--------------------------------------|
| Signal-to-Noise  | 75    | 8 of 32 instructions redundant      |
| Redundancy       | 80    | 5 overlap with eslint/prettier       |
| Specificity      | 90    | Most rules are actionable            |
| Guardrails       | 85    | 4 of 5 categories covered            |
| Conciseness      | 78    | 3 verbose sections                   |
| Freshness        | 95    | 1 stale reference                    |

Key Findings:
- 25% of instructions are discoverable from existing config files
- 1 instruction references a file that no longer exists
- Missing guardrail: no mention of file deletion safety
- 3 instructions exceed 60 words and can be shortened
```

## Format 2: Line-by-Line Audit

```
### CLAUDE.md — Line-by-Line Audit

Line 5-8: "## Organization Identity — You are assisting engineers at..."
  Verdict: REMOVE
  Evidence: Non-actionable boilerplate (AP-004) — agent doesn't need org identity to write code
  Confidence: High

Line 12: "Use single quotes for strings"
  Verdict: REMOVE
  Evidence: Redundant (AP-001) — already in .prettierrc (singleQuote: true)
  Confidence: High

Line 15: "Never force-push to main, master, or release branches"
  Verdict: KEEP
  Evidence: Critical guardrail — not discoverable from any config file
  Confidence: High

Line 20-25: "When working with the authentication system, you should be very careful..."
  Verdict: REVISE
  Evidence: Anxiety-inducing (AP-002) — vague instruction. Suggest: "Auth changes require unit test for each modified auth flow"
  Confidence: Medium

Line 30: "Use @company/legacy-auth for authentication"
  Verdict: REMOVE
  Evidence: Stale (AP-008) — @company/legacy-auth is not in package.json. Current dep is @company/auth-v2
  Confidence: High

(No existing instruction)
  Verdict: ADD
  Evidence: Missing guardrail (AP-007) — no instruction about PII/personal data handling
  Recommendation: Add: "Never log PII (email, phone, name, address, payment data) to any monitoring system"
  Confidence: High
```

## Format 3: Proposed Diff

```diff
  # Project Instructions

- ## Organization Identity
- You are assisting engineers at Acme Corp, a global technology company...
- Our mission is to deliver innovative solutions...
+ # (REMOVED: Non-actionable boilerplate — agent doesn't need org identity)

  ## Code Standards
- - Use single quotes for strings
+ # (REMOVED: Already in .prettierrc — singleQuote: true)
- - Use 2-space indentation
+ # (REMOVED: Already in .prettierrc — tabWidth: 2)
  - Use TypeScript strict mode  # KEEP: Not all tsconfig settings are obvious
  - Never use `any` without a justifying comment  # KEEP: Project-specific rule

  ## Forbidden Actions
  - Never force-push to main  # KEEP: Critical guardrail
  - Never commit .env files  # KEEP: Critical guardrail
+ - Never log PII (email, phone, name, address, payment data) to monitoring  # ADD: Missing guardrail

- ## Authentication
- When working with the authentication system, you should be very careful
- to ensure that all edge cases are handled properly and that the security
- implications of any changes are thoroughly considered before merging.
+ ## Authentication
+ - Auth changes require a unit test for each modified auth flow  # REVISED: Specific and actionable

- - Use @company/legacy-auth for authentication
+ # (REMOVED: Stale — @company/legacy-auth not in package.json, replaced by @company/auth-v2)
```

## Cross-Tool Duplication Report

```
## Cross-Tool Duplication Analysis

Files analyzed: CLAUDE.md, .github/copilot-instructions.md, .cursorrules

Duplicated instructions found: 12

| # | Instruction | Found In | Recommendation |
|---|-------------|----------|----------------|
| 1 | "Never force-push to main" | CLAUDE.md, copilot-instructions.md, .cursorrules | Keep in primary file only |
| 2 | "Use TypeScript strict mode" | CLAUDE.md, copilot-instructions.md | Keep in primary file only |
| 3 | "Run tests before committing" | CLAUDE.md, .cursorrules | Keep in primary file only |

Recommendation: Use CLAUDE.md as the primary source of truth.
Other files should contain only tool-specific instructions and a reference:
  "For project coding standards, see CLAUDE.md in the repository root."
```

## Starter CLAUDE.md Templates

### Node.js / TypeScript Starter

```markdown
# Project Instructions

## Guardrails
- Never force-push to main, master, or release branches
- Never commit .env files, API keys, or credentials
- Run the full test suite before submitting changes
- Never log PII (email, phone, name, address) to any monitoring system
- Confirm before deleting files or directories; prefer reversible operations (trash over rm)

## Project-Specific Context
<!-- Add instructions here that the agent CANNOT discover from your config files -->
<!-- Examples: business logic rules, deployment procedures, team conventions not in linters -->
```

### Python Starter

```markdown
# Project Instructions

## Guardrails
- Never force-push to main, master, or release branches
- Never commit .env files, API keys, or credentials
- Run pytest before submitting changes
- Never log PII to any monitoring system
- Confirm before deleting files or directories; prefer reversible operations
- Use virtual environments for all development

## Project-Specific Context
<!-- Add instructions here that the agent CANNOT discover from pyproject.toml, .flake8, etc. -->
```

### Java Starter

```markdown
# Project Instructions

## Guardrails
- Never force-push to main, master, or release branches
- Never commit application.properties with real credentials
- Run the full test suite before submitting changes
- Never log PII to any monitoring system
- Confirm before deleting files or directories

## Project-Specific Context
<!-- Add instructions here that the agent CANNOT discover from pom.xml, checkstyle, etc. -->
```

### General Starter

```markdown
# Project Instructions

## Guardrails
- Never force-push to main, master, or release branches
- Never commit secrets, credentials, or API keys
- Run tests before submitting changes
- Never log personal data to monitoring systems
- Confirm before deleting files or directories

## Project-Specific Context
<!-- Add ONLY instructions the agent cannot discover from your existing config files -->
<!-- If it's in .eslintrc, .prettierrc, tsconfig.json, pyproject.toml, etc., don't repeat it here -->
```
