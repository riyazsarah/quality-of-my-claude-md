# Contributing to quality-of-my-claude-md

Thanks for your interest in contributing! This project welcomes contributions from anyone looking to improve AI coding agent instruction file quality.

## How to Contribute

1. **Fork the repo** and create your branch from `main`
2. **Make your changes** — see areas for contribution below
3. **Test your changes** — install the skill locally and run an audit to verify it works
4. **Open a Pull Request** against `main` with a clear description of what you changed and why

## Areas for Contribution

- **New anti-pattern detections** — found a pattern that hurts agent performance? Add it to `references/anti-patterns.md`
- **Ecosystem support** — add config file mappings for Go, Rust, Ruby, PHP, .NET, or other ecosystems in `references/ecosystem-configs.md`
- **Secret patterns** — add new regex patterns for credential detection in `references/secret-patterns.md`
- **Scoring refinements** — improvements to dimension weights or calculation formulas in `references/scoring-rubric.md`
- **Starter templates** — add ecosystem-specific starter templates in `appendix/examples.md`
- **Documentation** — improve README, fix typos, add usage examples
- **Bug reports** — open an issue describing what you expected vs what happened

## Project Structure

```
skills/quality-of-my-claude-md/
├── SKILL.md                      # Main skill definition and phases
├── appendix/
│   ├── evaluation.md             # Test scenarios
│   └── examples.md               # Output format templates and starter templates
└── references/
    ├── anti-patterns.md          # 8 anti-pattern detection heuristics
    ├── ecosystem-configs.md      # Config file mappings per ecosystem
    ├── scoring-rubric.md         # 6 scoring dimensions and grade calculation
    └── secret-patterns.md        # Regex patterns for credential detection
```

## Guidelines

- Keep changes focused — one concern per PR
- Reference the ETH Zurich research (arxiv:2602.11988) when adding new anti-patterns or scoring changes
- Test with real repos before submitting — install the skill and run `audit context files` to verify
- Don't add tool-specific or org-specific assumptions — the skill should work for any project

## Reporting Issues

Open a GitHub issue with:
- What you were auditing (which context file type)
- What mode you used (default, --quick, --deep)
- What happened vs what you expected
- Your project ecosystem (Node.js, Python, Java, etc.)

## License

By contributing, you agree that your contributions will be licensed under the MIT License.
