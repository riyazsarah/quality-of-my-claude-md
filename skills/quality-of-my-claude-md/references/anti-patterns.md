# Anti-Pattern Catalog

Research basis: [ETH Zurich — Evaluating AGENTS.md](https://arxiv.org/abs/2602.11988)

## AP-001: Redundancy

**Research finding:** Instructions that restate what the agent can discover from config files add noise without value.

**Detection heuristic:**
- Instruction mentions a setting that exists in a project config file (eslint, prettier, tsconfig, pyproject.toml, etc.)
- Instruction describes a directory structure the agent can see by listing files
- Instruction explains what a dependency does when the agent can read package.json/requirements.txt

**Evidence template:** `"Redundant: already specified in <config_file> at line <N> (<setting_name>=<value>)"`

**Confidence criteria:**
- **High:** Exact setting match found in config file (e.g., "use single quotes" matches `singleQuote: true` in .prettierrc)
- **Medium:** Semantic match (e.g., "use camelCase" matches eslint naming-convention rule with camelCase option)
- **Low:** Partial overlap (e.g., "run tests before committing" matches a pre-commit hook in .husky/)

**Examples:**
- "Use single quotes for strings" when .prettierrc has `singleQuote: true`
- "Use Jest for testing" when jest is in devDependencies and jest.config.* exists
- "Source code is in src/" when the directory visibly exists

---

## AP-002: Anxiety Induction

**Research finding:** Overly broad, vague, or threatening rules cause some models to obsessively re-read context files, increasing token usage without improving output.

**Detection heuristic:**
- Excessive use of "ALWAYS", "NEVER", "CRITICAL", "IMPORTANT" without specific actionable detail
- Vague instructions like "be careful with..." or "make sure to consider..."
- Instructions that describe outcomes rather than actions: "ensure code quality" vs. "run eslint before committing"
- Repeated emphasis markers (bold + caps + exclamation) on non-safety instructions

**Evidence template:** `"Anxiety-inducing: vague instruction '<text>' lacks actionable specifics — agent cannot determine concrete compliance criteria"`

**Confidence criteria:**
- **High:** Instruction uses emphasis markers but contains no actionable verb or specific condition
- **Medium:** Instruction has an actionable verb but is too broad to verify compliance
- **Low:** Instruction uses strong emphasis but IS actionable — may be intentional for critical guardrails

**Examples:**
- "ALWAYS ensure best practices are followed" (vague, unverifiable)
- "Be VERY careful when modifying database schemas" (no specific action)
- "IMPORTANT: Consider all edge cases" (impossible to verify completeness)

---

## AP-003: Convention Overload

**Research finding:** When >15 formatting/style rules are listed, the overhead makes primary tasks harder. Agents spend tokens processing conventions instead of solving problems.

**Detection heuristic:**
- Count instructions classified as type `convention`
- Flag when total convention-type instructions exceed 15
- Flag individual convention instructions that overlap with linter/formatter enforcement

**Evidence template:** `"Convention overload: <N> convention-type instructions found (threshold: 15). <M> are already enforced by tooling."`

**Confidence criteria:**
- **High:** >20 convention instructions AND >50% are enforceable by tooling
- **Medium:** 15-20 convention instructions OR >50% enforceable by tooling
- **Low:** 10-15 convention instructions with most being non-tooling-enforceable

**Examples:**
- 25 bullet points about code style when eslint + prettier are configured
- Separate instructions for indentation, quotes, semicolons, trailing commas — all in .prettierrc

---

## AP-004: Non-Actionable Boilerplate

**Research finding:** Organization identity, mission statements, team philosophy, and historical context provide no signal the agent can use to write better code.

**Detection heuristic:**
- Instructions classified as type `boilerplate`
- Text containing organizational identity with no code-relevant action
- Sections about team values, company history, or mission without connection to coding decisions
- "About us" or "Overview" sections that don't inform any technical decision

**Evidence template:** `"Non-actionable boilerplate: '<text>' provides organizational context but no coding guidance the agent can act on"`

**Confidence criteria:**
- **High:** Section contains zero actionable verbs (no "use", "run", "avoid", "prefer", "never", "always")
- **Medium:** Section has 1-2 actionable items buried in 5+ lines of context
- **Low:** Section is mostly context but contains genuinely useful background for decision-making

**Examples:**
- "We are a global QSR technology company serving millions of customers"
- "Our engineering philosophy values simplicity and pragmatism"
- Team member lists, org chart references, mission statements

---

## AP-005: Discoverable Info

**Research finding:** Well-documented repos gain little from context files. Information already in README, standard tool output, or project structure is wasted context.

**Detection heuristic:**
- Instruction content that overlaps with README.md sections
- Build/run commands that match package.json scripts
- Architecture descriptions that match directory listing output
- Technology stack lists that match dependency manifests

**Evidence template:** `"Discoverable: '<instruction_summary>' is already documented in <source_file> — agent can find this without context file"`

**Confidence criteria:**
- **High:** Exact content match with README.md, package.json scripts, or CI workflow steps
- **Medium:** Semantic overlap — same information expressed differently
- **Low:** Partial overlap — context file adds nuance beyond what's discoverable

**Examples:**
- "To run tests: `npm test`" when package.json has a `test` script
- "The project uses React 18" when react@18 is in dependencies
- Build instructions that mirror CI workflow steps

---

## AP-006: Verbose Instructions

**Research finding:** Context files increase inference costs 20%+. Every unnecessary word costs tokens. Terse, specific instructions are more effective than verbose explanations.

**Detection heuristic:**
- Instruction blocks exceeding 50 words for a single rule
- Instructions that explain "why" at length when the "what" is sufficient
- Repeated information within the same instruction block
- Instructions that could be expressed in <15 words but use 50+

**Evidence template:** `"Verbose: instruction is <N> words — could be expressed in ~<M> words without losing meaning"`

**Confidence criteria:**
- **High:** Instruction exceeds 80 words AND a <20-word version captures the same actionable content
- **Medium:** Instruction is 50-80 words AND can be shortened by >50%
- **Low:** Instruction is 30-50 words AND contains some justification that may be useful

**Examples:**
- A 100-word paragraph explaining that environment variables should be used for configuration, when "Use environment variables for all runtime configuration — never hardcode" suffices
- Multi-sentence explanation of why force-pushing is bad when "Never force-push to main/master/release branches" covers it

---

## AP-007: Missing Guardrails

**Research finding:** The highest-value content in context files is guardrails — things the agent must NEVER do. Repos without guardrails miss the primary benefit of context files.

**Detection heuristic:**
- Check for ABSENCE of critical guardrails in the file
- Essential guardrails to check for:
  - Secret/credential protection (never commit .env, API keys, etc.)
  - Destructive git operations (never force-push to main)
  - Test requirements (run tests before committing)
  - PII handling (never log personal data)
  - File deletion safety (avoid rm -rf or equivalent)

**Evidence template:** `"Missing guardrail: no instruction found for '<guardrail_category>' — this is a high-value addition"`

**Confidence criteria:**
- **High:** 3+ essential guardrail categories completely absent from the file
- **Medium:** 1-2 essential guardrail categories missing
- **Low:** All essential categories present but some could be more specific

**Examples:**
- No mention of secrets/credentials handling in a repo with .env files
- No mention of test requirements in a repo with a test suite
- No mention of PII protection in a repo that handles user data

---

## AP-008: Stale Instructions

**Research finding (practical addition):** Instructions referencing files, tools, dependencies, or patterns that no longer exist in the repo are actively harmful — they cause the agent to search for or use things that don't exist.

**Detection heuristic:**
- Instructions referencing file paths that don't exist on disk
- Instructions mentioning dependencies not in the manifest (package.json, requirements.txt, pom.xml)
- Instructions referencing tools or commands that aren't installed or configured
- Instructions mentioning API endpoints, environment variables, or config keys that don't exist

**Evidence template:** `"Stale: references '<entity>' which does not exist in the current repo (<checked_location>)"`

**Confidence criteria:**
- **High:** Referenced file path does not exist AND no similar file found
- **Medium:** Referenced dependency is not in manifest but a similar one exists (possible rename)
- **Low:** Referenced entity may exist under a different name or location

**Examples:**
- "Tests are in `__tests__/` directory" when tests are actually in `test/`
- "Use `@company/old-logger`" when the dependency is now `@company/new-logger`
- "Run `yarn lint:css`" when the script was removed from package.json
