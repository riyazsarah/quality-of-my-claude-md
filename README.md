# quality-of-my-claude-md

A Claude Code skill that audits AI coding agent context files (`CLAUDE.md`, `agents.md`, `copilot-instructions.md`, `.cursorrules`, `.windsurfrules`) against research-backed quality criteria. It detects anti-patterns, redundancy, stale instructions, leaked secrets, and cross-tool duplication — then produces a quality score card, line-by-line audit, and proposed diff.

**Research basis:** [ETH Zurich (arxiv:2602.11988)](https://arxiv.org/abs/2602.11988) found that auto-generated context files reduce agent success rates ~2% and increase costs 20%+. The best context file contains only information the agent cannot get elsewhere.

## Installation

Copy the skill directory into your Claude Code skills folder:

```bash
# Clone this repo
git clone https://github.com/riyazsarah/quality-of-my-claude-md.git

# Copy the skill into your project or global skills
cp -r quality-of-my-claude-md/skills/quality-of-my-claude-md ~/.claude/skills/
```

Or copy directly into a project:

```bash
cp -r quality-of-my-claude-md/skills/quality-of-my-claude-md /path/to/your/project/.claude/skills/
```

## Usage

Trigger the skill in Claude Code with any of these phrases:

- `evaluate my claude.md`
- `audit my agents.md`
- `check claude.md quality`
- `is my CLAUDE.md helping?`
- `optimize my context file`
- `review my CLAUDE.md`
- `quality of my claude md`
- `audit context files`

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
3. **Cross-Tool Duplication Report** — if you have multiple context files with overlapping content
4. **Line-by-Line Audit** — every instruction block with verdict (KEEP / REMOVE / REVISE / ADD) and evidence
5. **Proposed Diff** — ready-to-apply changes with inline justification

## Scoring Dimensions

| Dimension | Weight | What It Measures |
|-----------|--------|------------------|
| Signal-to-Noise Ratio | 25% | % of instructions providing unique, non-discoverable value |
| Redundancy Score | 20% | Overlap with project configs, README, CI |
| Specificity | 20% | How actionable and concrete each instruction is |
| Guardrail Coverage | 15% | Presence of critical safety guardrails (secrets, destructive ops, tests, PII, file safety) |
| Conciseness | 10% | Word efficiency per instruction block |
| Freshness | 10% | % of referenced files/deps that actually exist |

## Supported Context Files

- `CLAUDE.md` (any directory level)
- `agents.md` / `AGENTS.md`
- `.github/copilot-instructions.md`
- `.cursorrules`
- `.windsurfrules`

## No Context Files? No Problem

If the skill finds zero context files, it offers to generate a starter `CLAUDE.md` tailored to your project's ecosystem (Node.js, Python, Java, etc.).

## License

[MIT](LICENSE)
