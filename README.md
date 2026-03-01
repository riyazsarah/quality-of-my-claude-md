# quality-of-my-claude-md

A [Claude Code](https://docs.anthropic.com/en/docs/claude-code) skill that audits **any AI coding agent context file** against research-backed quality criteria. Works with all major agentic coding tools — not just Claude.

### Supported Context Files

| Tool | Context File | Auto-Detected |
|------|-------------|---------------|
| **Claude Code** | `CLAUDE.md` (any directory level) | Yes |
| **Claude Code** | `~/.claude/CLAUDE.md` (global) | Yes |
| **GitHub Copilot** | `.github/copilot-instructions.md` | Yes |
| **Cursor** | `.cursorrules` | Yes |
| **Windsurf (Codeium)** | `.windsurfrules` | Yes |
| **Generic / Multi-tool** | `agents.md` / `AGENTS.md` | Yes |

The skill scans your repo for **all** of these, audits each one independently, and detects **cross-tool duplication** when you have multiple files with overlapping instructions.

**Research basis:** [ETH Zurich (arxiv:2602.11988)](https://arxiv.org/abs/2602.11988) found that auto-generated context files reduce agent success rates ~2% and increase costs 20%+. The best context file contains only information the agent cannot get elsewhere.

## What It Does

- Detects anti-patterns (redundancy, boilerplate, stale references, vague instructions)
- Finds leaked secrets and credentials
- Identifies cross-tool duplication and recommends a single-source-of-truth strategy
- Scores file quality across 6 research-backed dimensions
- Produces a line-by-line audit with KEEP / REMOVE / REVISE / ADD verdicts
- Generates a proposed diff and applies fixes after your confirmation

## Installation

Copy the skill directory into your Claude Code skills folder:

```bash
# Clone this repo
git clone https://github.com/riyazsarah/quality-of-my-claude-md.git

# Option A: Install globally (audits any project)
cp -r quality-of-my-claude-md/skills/quality-of-my-claude-md ~/.claude/skills/

# Option B: Install per-project
cp -r quality-of-my-claude-md/skills/quality-of-my-claude-md /path/to/your/project/.claude/skills/
```

## Usage

Trigger the skill in Claude Code with any of these phrases:

```
evaluate my claude.md
audit my agents.md
check claude.md quality
is my CLAUDE.md helping?
optimize my context file
review my CLAUDE.md
audit context files
check my .cursorrules
improve my copilot instructions
```

The skill auto-discovers all context files in your repo regardless of which phrase you use.

## Modes

| Flag | Mode | What It Does | Time |
|------|------|--------------|------|
| *(default)* | Smart Hybrid | Static analysis + targeted codebase probing for flagged items | ~2-3 min |
| `--quick` | Pattern Linter | Static analysis only, no codebase probing | ~30s |
| `--deep` | Full Audit | Full codebase probing for every instruction | ~3-5 min |
| `--benchmark` | Comparative | All phases + experimental A/B benchmark (experimental) | ~10-15 min |

Example: `audit context files --deep`

## What You Get

1. **Score Card** — overall grade (A-F) with 6 dimension scores
2. **Security Warnings** — any leaked secrets or credentials detected
3. **Cross-Tool Duplication Report** — if you have multiple context files with overlapping content, the skill surveys which tools you use and recommends a primary source of truth
4. **Line-by-Line Audit** — every instruction block with verdict (KEEP / REMOVE / REVISE / ADD) and evidence
5. **Proposed Diff** — ready-to-apply changes with inline justification

## Scoring Dimensions

| Dimension | Weight | What It Measures |
|-----------|--------|------------------|
| Signal-to-Noise Ratio | 25% | % of instructions providing unique, non-discoverable value |
| Redundancy Score | 20% | Overlap with project configs (ESLint, Prettier, pyproject.toml, etc.) |
| Specificity | 20% | How actionable and concrete each instruction is |
| Guardrail Coverage | 15% | Presence of critical safety guardrails (secrets, destructive ops, tests, PII, file safety) |
| Conciseness | 10% | Word efficiency per instruction block |
| Freshness | 10% | % of referenced files/deps that actually exist |

## Cross-Tool Deduplication

If you use multiple AI coding tools, you probably have overlapping instructions across files. The skill:

1. Detects exact and semantic duplicates across all discovered context files
2. Asks which tools you actively use (Claude, Copilot, Cursor, Windsurf)
3. Recommends a primary file as source of truth
4. Offers to generate slim reference files for secondary tools that point to the primary

## No Context Files? No Problem

If the skill finds zero context files, it offers to generate a starter context file tailored to your project's ecosystem (Node.js, Python, Java, or generic).

## Ecosystem Support

The skill detects your project ecosystem and checks context file instructions against your actual config files:

- **Node.js/TypeScript** — checks against `package.json`, `tsconfig.json`, `.eslintrc`, `.prettierrc`, Jest/Vitest config, Husky hooks
- **Python** — checks against `pyproject.toml`, Ruff/Black/Flake8 config, pytest, mypy
- **Java** — checks against `pom.xml`, `build.gradle`, Checkstyle, PMD
- **General** — checks against README, CI/CD workflows, `.editorconfig`, `.gitignore`

## References & Further Reading

- **[ETH Zurich — Evaluating AGENTS.md: Are Repository-Level Context Files Helpful for Coding Agents?](https://arxiv.org/abs/2602.11988)** — The research paper this skill is built on. Finds that poorly written context files reduce agent success rates ~2% and increase inference costs 20%+.
- **[Delete your CLAUDE.md (and your AGENT.md too) — Theo / t3.gg](https://www.youtube.com/watch?v=GcNu6wrLTJc)** — Video breakdown of why most context files hurt more than they help, and what to do instead.
- **[The Complete Guide to AI Agent Memory Files (CLAUDE.md, AGENTS.md, and Beyond)](https://hackernoon.com/the-complete-guide-to-ai-agent-memory-files-claudemd-agentsmd-and-beyond)** — Comprehensive guide covering all major context file formats and how to organize them.
- **[Improve Your AI Code Output with AGENTS.md — Builder.io](https://www.builder.io/blog/agents-md)** — Practical tips on writing effective agent instructions with real examples.
- **[AGENTS.md: One File to Guide Them All — Layer5](https://layer5.io/blog/ai/agentsmd-one-file-to-guide-them-all/)** — Making the case for AGENTS.md as the cross-tool standard.
- **[Creating the Perfect CLAUDE.md for Claude Code — Dometrain](https://dometrain.com/blog/creating-the-perfect-claudemd-for-claude-code/)** — Focused guide on writing effective CLAUDE.md files.
- **[Agentic Coding with Claude Code and Cursor — Softcery](https://softcery.com/lab/softcerys-guide-agentic-coding-best-practices)** — Best practices for context files, workflows, and MCP integration across tools.

## License

[MIT](LICENSE)
