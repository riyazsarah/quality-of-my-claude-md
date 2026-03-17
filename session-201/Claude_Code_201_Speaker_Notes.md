# Claude Code 201 — Speaker Notes & Practical Demo Guide

**Session:** Intermediate – Configuring Claude Code for Your Team
**Date:** Tuesday, 18 March 2026 · 11:30 AM – 12:30 PM
**Presenter:** Riyaj Shaikh (riyaj_shaikh@epam.com)

---

## Slide 1 — Title

**Speaker Notes:**
Welcome everyone. This is the 201 session — intermediate level. I'm assuming you've used Claude Code at least a few times and are comfortable with basics like running `claude` in a terminal, asking it questions, and letting it edit files. Today we move into *team configuration* — how to make Claude Code remember your project, connect it to your tools, and automate repetitive tasks. By the end of this hour, you'll have five new skills you can apply to your projects immediately.

---

## Slide 2 — Agenda

**Speaker Notes:**
We're covering five topics, each building on the previous one. CLAUDE.md is how you tell Claude what to do. Auto Memory is how Claude tells *itself* what to remember. Context management is about the window of information Claude works with — and what happens when it fills up. MCP connects Claude to external services. And Hooks let you automate actions at specific lifecycle points. Each section has a hands-on demo you can follow along with.

---

## Slide 3–4 — CLAUDE.md Deep Dive

**Speaker Notes:**
Let's start with the most impactful file in your repo: CLAUDE.md. Every time Claude Code starts a session, it reads this file before doing anything else. Think of it as a system prompt that lives in your codebase.

**Key talking points:**
- The file hierarchy matters: managed policy → project → user → subdirectory. Project-level is the most important for teams because it's shared via git.
- The `@path` import syntax is powerful. You can reference `@README.md` or `@docs/api-guide.md` to pull in existing documentation.
- `.claude/rules/` lets you split a large CLAUDE.md into topic files with path-scoping. For example, a `frontend.md` rule that only activates when Claude touches `src/components/**/*.tsx`.
- The 200-line guideline is critical. In tests, adherence drops noticeably above 200 lines because important rules get buried.
- `claudeMdExcludes` is essential for monorepos where other teams' CLAUDE.md files pollute your context.

**Key quote from official docs:** "CLAUDE.md files are loaded into the context window at the start of every session, consuming tokens alongside your conversation. Because they're context rather than enforced configuration, how you write instructions affects how reliably Claude follows them."

### Hands-On Demo 1: Create & Optimize a CLAUDE.md

**Setup:** Have a sample project open (a small Python or Node.js app).

```bash
# Step 1: Auto-generate from your codebase
cd your-project/
claude
# Inside Claude Code:
/init
```

**What to show:**
1. Run `/init` and watch Claude analyze the codebase
2. Review the generated CLAUDE.md — point out build commands, test instructions, conventions
3. Edit it live: add a specific rule like "Always use `pytest -x --tb=short` for tests"
4. Show the `@` import syntax:

```markdown
# In CLAUDE.md
See @README.md for project overview
See @package.json for available commands

# Team standards
@docs/coding-standards.md
```

5. Create a path-specific rule:

```bash
mkdir -p .claude/rules
```

```markdown
# .claude/rules/api.md
---
paths:
  - "src/api/**/*.py"
---

# API Development Rules
- All endpoints must include input validation with Pydantic
- Use standard error response format from src/api/errors.py
- Include OpenAPI docstrings on every route
```

6. Verify with `/memory` — show all loaded files

**Audience exercise:** Each person runs `/init` on their own project and adds 3 custom rules.

---

## Slide 5–6 — Auto Memory

**Speaker Notes:**
Now let's talk about the *other* memory system. CLAUDE.md is what *you* write for Claude. Auto Memory is what *Claude* writes for itself. When you correct Claude — "use pnpm, not npm" — it saves that note so it doesn't make the same mistake next session. This shipped in Claude Code v2.1.59 (early 2026) and is on by default.

**Key talking points:**
- Claude doesn't save every session. It decides what's genuinely useful for future conversations.
- Storage is per-project, keyed to the git repo. All worktrees share one memory directory.
- MEMORY.md is the index file — first 200 lines loaded every session. Topic files are loaded on demand.
- You can (and should) audit memory periodically. It's all plain markdown.
- Subagents can maintain their own auto memory too.

**Key quote from Medium (Joe Njenga, Feb 2026):** "What surprised me most is that Claude doesn't just dump everything into memory. It curates — keeping build commands, debugging patterns, and style preferences, while ignoring one-off conversations."

**Key quote from Medium (Brent W. Peterson, Feb 2026):** "Automatic memory is not learning. What Claude remembers when you're not looking is just compressed context recall — useful patterns it encountered, not deep understanding."

### Hands-On Demo 2: Explore & Shape Auto Memory

```bash
# Start a session
claude

# Inside Claude Code:
/memory
# → Browse CLAUDE.md files loaded
# → Toggle auto memory on/off
# → Open auto memory folder
```

**What to show:**
1. Run `/memory` and show the menu
2. Open the auto memory folder — browse MEMORY.md
3. Deliberately teach Claude something:
   ```
   > Remember that this project uses Black with line-length 88 for formatting
   ```
   Watch for "Writing memory" in the output.
4. Start a *new* session and verify:
   ```bash
   claude -c  # continue most recent
   # or
   claude      # fresh session
   > What formatter should I use for this project?
   ```
   Claude should recall the Black preference.
5. Show how to edit memory manually:
   ```bash
   cat ~/.claude/projects/<your-project>/memory/MEMORY.md
   ```

**Audience exercise:** Ask Claude to remember a specific convention, end the session, start fresh, and verify recall.

---

## Slide 7–8 — Context Management, Compaction & Rewind

**Speaker Notes:**
Here's where things get technical. Claude has a fixed context window. Everything fits inside it: your conversation, file contents, CLAUDE.md, skills, MCP tools. As you work, it fills up. When it does, Claude compacts — automatically summarizing older content to make room. Understanding this is crucial because it explains why Claude sometimes "forgets" instructions from earlier in the conversation.

**Key talking points:**
- Context holds: conversation history + file contents + command outputs + CLAUDE.md + skills + MCP tool definitions + system instructions
- Compaction clears older tool outputs first, then summarizes the conversation
- CLAUDE.md *fully survives* compaction — it's re-read from disk. But conversational instructions don't.
- The checkpoint system snapshots every file before editing. Press `Esc Esc` to rewind.
- Checkpoints are session-local, separate from git. They cover file changes only.
- `/compact focus on the API changes` lets you guide what gets preserved.
- `CLAUDE_AUTOCOMPACT_PCT_OVERRIDE` environment variable lets you control the compaction threshold (1-100%).

**Key insight from claudefa.st:** "As of early 2026, the buffer has been reduced to ~33,000 tokens (16.5%) giving you roughly 12K more usable space. Claude Code manages context automatically as you approach the limit."

**Key insight from Medium (Code Coup, Feb 2026):** Context recovery hooks can automatically save and restore context before/after compaction, preventing work loss during long sessions.

### Hands-On Demo 3: Context & Checkpoints in Action

```bash
# Start a session and check context usage
claude

# Inside Claude Code:
/context
# → Shows token breakdown: conversation, tools, CLAUDE.md, etc.
```

**What to show:**
1. Run `/context` — explain the token breakdown
2. Make a deliberate file edit:
   ```
   > Add a hello_world function to main.py
   ```
3. Show the checkpoint was created:
   ```
   > Now add a goodbye_world function
   ```
4. Demonstrate rewind:
   - Press `Esc Esc` (double-tap Escape)
   - Show the menu: restore code, conversation, or both
   - Rewind to the state before goodbye_world
5. Show `/compact`:
   ```
   /compact focus on the main.py changes
   ```
6. After compaction, verify CLAUDE.md rules still work — ask Claude to follow a specific rule from your CLAUDE.md.

**Audience exercise:** Create a file, make 3 changes, then rewind to the first change. Verify the file state.

---

## Slide 9–10 — MCP (Model Context Protocol)

**Speaker Notes:**
MCP is what transforms Claude Code from a coding assistant into a fully connected development tool. It's an open standard — the same protocol works in Claude Desktop, VS Code, and other tools. You can connect Claude to GitHub for PR reviews, Sentry for error monitoring, PostgreSQL for data queries, Slack for team communication, and hundreds more.

**Key talking points:**
- Three transports: HTTP (recommended for cloud), SSE (deprecated), stdio (local tools)
- Three scopes: local (just you, this project), project (shared via .mcp.json), user (all your projects)
- OAuth 2.0 support for authenticated services
- Tool Search automatically defers MCP tools when they'd consume >10% of context window — massive for teams with many integrations
- `.mcp.json` supports environment variable expansion: `${API_KEY}`, `${VAR:-default}`
- Managed MCP (`managed-mcp.json`) lets IT admins control which servers employees can use

**Key insight from X/Twitter (Claude Code Changelog):** "Claude Code CLI 2.1.76 added MCP elicitation support — MCP servers can now request structured input mid-task via an interactive dialog."

### Hands-On Demo 4: Connect Claude to GitHub

**Prerequisites:** GitHub account, a repo with at least one open issue or PR.

```bash
# Step 1: Add the GitHub MCP server
claude mcp add --transport http github https://api.githubcopilot.com/mcp/

# Step 2: Authenticate
claude
/mcp
# → Select GitHub → Authenticate in browser

# Step 3: Use it
> Review PR #42 and suggest improvements
> Show me all open issues assigned to me
> Create a new issue titled "Add dark mode support" with label "enhancement"
```

**What to show:**
1. Run `claude mcp add` — show the command
2. Inside Claude, run `/mcp` to see the server listed
3. Authenticate via the browser flow
4. Ask Claude to interact with GitHub:
   - List recent PRs
   - Read an issue
   - Create a new issue
5. Show `claude mcp list` to see all configured servers
6. Show the scope options:
   ```bash
   # Share with team (creates .mcp.json)
   claude mcp add --transport http --scope project github https://api.githubcopilot.com/mcp/

   # Personal, all projects
   claude mcp add --transport http --scope user github https://api.githubcopilot.com/mcp/
   ```

**Alternative quick demo (if no GitHub access):**
```bash
# File system MCP server — works locally, no auth needed
claude mcp add --transport stdio fs -- npx -y @modelcontextprotocol/server-filesystem /tmp/demo
```

**Audience exercise:** Add the GitHub MCP server, authenticate, and ask Claude to list open issues from a repo.

---

## Slide 11–12 — Introduction to Hooks

**Speaker Notes:**
Hooks are the automation layer. They're deterministic scripts that fire at specific lifecycle events. Unlike asking Claude to "always format code after editing" (which it might forget), a hook *guarantees* Prettier runs after every file write. They're configured in settings.json and support four types: shell commands, HTTP endpoints, LLM prompts, and full subagents.

**Key talking points:**
- Hooks are NOT LLM-generated. They're user-defined scripts with deterministic behavior.
- Key events: PreToolUse (can block), PostToolUse (can react), Notification (can alert), Stop (can validate), SessionStart (can inject context)
- Exit code 0 = allow, exit code 2 = block, stderr becomes Claude's feedback
- Matchers use regex to filter: `"Edit|Write"` only fires for file edits, not Bash commands
- `/hooks` is the read-only browser — edit settings.json directly to configure
- Prompt hooks (`type: "prompt"`) ask a model for yes/no decisions
- Agent hooks (`type: "agent"`) spawn a subagent with tool access for verification

**Key insight from the official docs:** "Hooks provide deterministic control over Claude Code's behavior, ensuring certain actions always happen rather than relying on the LLM to choose to run them."

**Key insight from X/Twitter community:** The most popular hook patterns are: (1) auto-format with Prettier/Black, (2) block edits to .env files, (3) desktop notifications when Claude is idle, (4) re-inject context after compaction.

### Hands-On Demo 5: Build Three Hooks

**Hook 1: Auto-format Python with Black**

Create `.claude/settings.json`:

```json
{
  "hooks": {
    "PostToolUse": [
      {
        "matcher": "Edit|Write",
        "hooks": [
          {
            "type": "command",
            "command": "FILE=$(cat | jq -r '.tool_input.file_path'); if [[ \"$FILE\" == *.py ]]; then black --quiet \"$FILE\" 2>/dev/null; fi"
          }
        ]
      }
    ]
  }
}
```

**What to show:**
1. Create the settings file
2. Ask Claude to create a Python file with messy formatting
3. Watch Black auto-format it after the edit
4. Verify with `/hooks` — show the hook listed under PostToolUse

**Hook 2: Block edits to protected files**

Add to `.claude/settings.json`:

```json
{
  "hooks": {
    "PreToolUse": [
      {
        "matcher": "Edit|Write",
        "hooks": [
          {
            "type": "command",
            "command": "FILE=$(cat | jq -r '.tool_input.file_path // empty'); for pat in .env secrets.yaml credentials; do if [[ \"$FILE\" == *\"$pat\"* ]]; then echo \"Blocked: $FILE is protected\" >&2; exit 2; fi; done; exit 0"
          }
        ]
      }
    ]
  }
}
```

**What to show:**
1. Add the hook
2. Ask Claude to modify a `.env` file
3. Watch it get blocked — Claude receives the "Blocked" feedback
4. Claude adjusts and suggests you edit it manually

**Hook 3: Desktop notification when Claude is idle**

Add to `~/.claude/settings.json` (user-level):

```json
{
  "hooks": {
    "Notification": [
      {
        "matcher": "",
        "hooks": [
          {
            "type": "command",
            "command": "osascript -e 'display notification \"Claude needs your attention\" with title \"Claude Code\"'"
          }
        ]
      }
    ]
  }
}
```

**What to show:**
1. Add the hook to user settings
2. Ask Claude to do something that requires permission
3. Switch to a different app
4. Get the macOS notification popup

**Audience exercise:** Create the auto-format hook for your preferred formatter (Prettier, Black, rustfmt, gofmt) and test it.

---

## Slide 13 — Closing & Q&A

**Speaker Notes:**
Let me recap. CLAUDE.md is your team's shared memory — keep it under 200 lines, use rules for path-specific instructions. Auto Memory lets Claude learn from your corrections automatically. Context management and checkpoints give you safety nets. MCP connects Claude to your entire tool stack. And Hooks automate the repetitive stuff so it happens every time, not just when Claude remembers.

The key takeaway: Claude Code is not just a chatbot in your terminal. It's a configurable agent with a memory system, an extension protocol, and an automation layer. Your job isn't to type better prompts — it's to *configure the system* so Claude works the way your team needs.

**Resources to share:**
- Official docs: https://code.claude.com/docs
- MCP servers: https://github.com/modelcontextprotocol/servers
- MCP protocol: https://modelcontextprotocol.io
- Hooks guide: https://code.claude.com/docs/en/hooks-guide
- Memory guide: https://code.claude.com/docs/en/memory

---

## Research Sources

### Official Documentation
- [How Claude remembers your project](https://code.claude.com/docs/en/memory) — CLAUDE.md, auto memory, rules, troubleshooting
- [How Claude Code works](https://code.claude.com/docs/en/how-claude-code-works) — context window, compaction, checkpoints, agentic loop
- [Automate workflows with hooks](https://code.claude.com/docs/en/hooks-guide) — hook events, types, configuration, examples
- [Connect Claude Code to tools via MCP](https://code.claude.com/docs/en/mcp) — transports, scopes, authentication, tool search
- [Best Practices for Claude Code](https://code.claude.com/docs/en/best-practices)

### Medium & Blog Articles
- [Anthropic Just Added Auto-Memory to Claude Code — MEMORY.md (I Tested It)](https://medium.com/@joe.njenga/anthropic-just-added-auto-memory-to-claude-code-memory-md-i-tested-it-0ab8422754d2) — Joe Njenga, Feb 2026
- [Automatic Memory Is Not Learning](https://medium.com/@brentwpeterson/automatic-memory-is-not-learning-4191f548df4c) — Brent W. Peterson, Feb 2026
- [Context Recovery Hook for Claude Code](https://medium.com/coding-nexus/context-recovery-hook-for-claude-code-never-lose-work-to-compaction-7ee56261ee8f) — Code Coup, Feb 2026
- [Claude Code in March 2026: The Economics of the Quota](https://medium.com/@william.couturier/claude-code-in-march-2026-the-economics-of-the-quota-792449b63edb) — William Couturier, Mar 2026
- [The Complete Guide to Setting Global Instructions for Claude Code CLI](https://naqeebali-shamsi.medium.com/the-complete-guide-to-setting-global-instructions-for-claude-code-cli-cec8407c99a0) — Naqeeb Ali Shamsi
- [Claude Code Configuration Blueprint](https://dev.to/mir_mursalin_ankur/claude-code-configuration-blueprint-the-complete-guide-for-production-teams-557p) — DEV Community
- [Claude Code Hooks Guide 2026](https://dev.to/serenitiesai/claude-code-hooks-guide-2026-automate-your-ai-coding-workflow-dde) — DEV Community
- [Claude Code Context Buffer: The 33K-45K Token Problem](https://claudefa.st/blog/guide/mechanics/context-buffer-management) — ClaudeFast

### X/Twitter Posts & Community
- [Claude Code Changelog — MCP Elicitation (v2.1.76)](https://x.com/ClaudeCodeLog/status/2032631384523641209)
- [Shraddha Bharuka — Claude Code 4-Layer System](https://x.com/BharukaShraddha/status/2030965172207317388)
- [cogsec — The Longform Guide to Everything Claude Code](https://x.com/affaanmustafa/status/2014040193557471352)
- [Behrooz Azarkhalili — Claude Memory Plugin Analysis](https://x.com/b_azarkhalili/status/2009319370884059231)

### Additional References
- [Claude Code Context Buffer Management](https://claudefa.st/blog/guide/mechanics/context-buffer-management)
- [Claude Code Session Memory: Automatic Cross-Session Context](https://claudefa.st/blog/guide/mechanics/session-memory)
- [Using CLAUDE.md Files — Claude Blog](https://claude.com/blog/using-claude-md-files)
- [Writing a Good CLAUDE.md — HumanLayer](https://www.humanlayer.dev/blog/writing-a-good-claude-md)
- [CLAUDE.md for Product Managers](https://ccforpms.com/fundamentals/project-memory)
- [Claude Code Hooks: Complete Guide with 20+ Examples](https://aiorg.dev/blog/claude-code-hooks)
- [Auto-format with Claude Code Hooks](https://martin.hjartmyr.se/articles/auto-format-with-claude-code-hooks/)
- [Claude MCP Integration Guide — Fast.io](https://fast.io/resources/claude-mcp-integration-guide/)
