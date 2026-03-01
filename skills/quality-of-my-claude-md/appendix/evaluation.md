# Evaluation

## Trigger Testing

| Scenario | Input | Expected Behavior |
|----------|-------|-------------------|
| Trigger — positive | "evaluate my claude.md" | Skill activates, runs default (smart hybrid) mode |
| Trigger — positive | "audit my agents.md" | Skill activates, runs default mode |
| Trigger — positive | "check claude.md quality" | Skill activates, runs default mode |
| Trigger — positive | "is my CLAUDE.md helping?" | Skill activates, runs default mode |
| Trigger — positive | "optimize my context file" | Skill activates, runs default mode |
| Trigger — positive | "review my CLAUDE.md" | Skill activates, runs default mode |
| Trigger — positive | "analyze my context files" | Skill activates, runs default mode |
| Trigger — positive | "audit context files --quick" | Skill activates in quick mode |
| Trigger — positive | "evaluate claude.md --deep" | Skill activates in deep mode |
| Trigger — positive | "check claude.md --benchmark" | Skill activates in benchmark mode |
| Trigger — negative | "write a CLAUDE.md from scratch" | Skill does NOT activate |
| Trigger — negative | "what's in my CLAUDE.md?" | Skill does NOT activate (just read the file) |
| Trigger — negative | "create a new context file" | Skill does NOT activate |
| Trigger — negative | "what time is it?" | Skill does NOT activate |

## Edge Case Scenarios

| ID | Scenario | Input | Expected Behavior |
|----|----------|-------|-------------------|
| EC-001 | Empty context file | CLAUDE.md exists but is 0 bytes | Reports score 0/100, recommends populating or deleting |
| EC-002 | Large context file | CLAUDE.md with 600+ instruction blocks | Caps analysis at 200 blocks, warns user, suggests --deep mode |
| EC-003 | Malformed config | .eslintrc exists but contains invalid JSON | Skips that config check, logs warning, continues with remaining checks |
| EC-004 | Multiple AI tool files | CLAUDE.md + agents.md + .cursorrules all present | Analyzes each independently, runs cross-tool dedup analysis |
| EC-005 | Guardrails-only file | CLAUDE.md contains only forbidden-action and guardrail instructions | Scores well on signal-to-noise, notes the focused approach positively |
| EC-006 | Heavy duplication | 3 context files with 80%+ identical content | Flags as heavy duplication, prioritizes single-source-of-truth recommendation |

## Mode-Specific Scenarios

| Mode | Flag | Phases Run | Verification |
|------|------|------------|-------------|
| Quick | `--quick` | 1, 2, 5, 6 | No codebase probing occurs. Redundancy findings show lower confidence. |
| Deep | `--deep` | 1, 2, 3, 5, 6 | Full codebase probing. Config file overlap verified. High-confidence findings. |
| Benchmark | `--benchmark` | 1, 2, 3, 4, 5, 6 | Experimental phase 4 runs. Results labeled as estimates. |
| Default | *(none)* | 1, 2, targeted 3, 5, 6 | Only flagged items get probed in Phase 3. Balance of speed and accuracy. |

## Cross-Tool Dedup Scenarios

| Scenario | Files Present | Expected |
|----------|--------------|----------|
| Single file | CLAUDE.md only | No dedup analysis. Normal audit only. |
| Two files, no overlap | CLAUDE.md + .cursorrules with different content | Reports both independently, no duplication found. |
| Two files, overlap | CLAUDE.md + agents.md with 50%+ same content | Detects duplicates, asks tool usage, recommends primary file. |
| Three+ files | CLAUDE.md + agents.md + copilot-instructions.md | Full dedup matrix, recommends consolidation strategy. |

## Error Scenarios

| Scenario | Expected |
|----------|----------|
| No context files in repo | Detects ecosystem, offers to generate starter CLAUDE.md |
| Permission denied on config file | Skips that check, logs warning, continues |
| Binary file in glob results | Skips, continues with text files |
| User declines all fixes | Exits cleanly with report only |
| User approves partial fixes | Applies only selected category of changes |

## Fix Application Scenarios

| User Choice | Expected |
|-------------|----------|
| "Apply all changes" | All REMOVE/REVISE/ADD verdicts applied via Edit tool |
| "Apply removals only" | Only REMOVE verdicts applied, REVISE and ADD skipped |
| "Skip — report only" | No files modified, report displayed |
| Reference file generation approved | Tool-specific reference files created pointing to primary |
| Reference file generation declined | No reference files created |
