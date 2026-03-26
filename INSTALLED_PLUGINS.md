# Installed Claude Code Plugins

Third-party plugins installed on the system. Use `claude plugin add <url>` to reinstall on a new workstation.

## superpowers-marketplace

**Source:** `https://github.com/obra/superpowers-marketplace`

| Plugin | Version | Description |
|--------|---------|-------------|
| superpowers | 4.0.3 | Core skills: TDD, debugging, collaboration patterns |
| superpowers-chrome | 1.6.2 | Chrome DevTools Protocol access via browsing skill |
| elements-of-style | 1.0.0 | Writing guidance based on Strunk's Elements of Style |
| episodic-memory | 1.0.15 | Semantic search for past conversations across sessions |
| superpowers-lab | 0.1.0 | Control interactive CLI tools through tmux automation |
| superpowers-developing-for-claude-code | 0.3.1 | Skills for developing Claude Code plugins and extensions |
| double-shot-latte | 1.1.5 | Auto-continue without "Would you like me to continue?" |

## context-engineering-kit

**Source:** `https://github.com/NeoLabHQ/context-engineering-kit`

| Plugin | Version | Description |
|--------|---------|-------------|
| reflexion | 1.1.4 | Self-refine and reflect on LLM output |
| code-review | 1.0.8 | Multi-agent codebase and PR review |
| git | 1.2.0 | Commit, PR, worktree commands |
| tdd | 1.1.0 | Test-driven development commands and skills |
| sadd | 1.2.0 | Subagent-driven development with quality gates |
| ddd | 1.0.0 | Domain-driven development best practices |
| sdd | 2.1.1 | Specification-driven development workflow |
| kaizen | 1.0.0 | Root cause analysis (5 Whys, Cause & Effect) |
| customaize-agent | 1.3.3 | Writing and refining Claude Code commands/skills |
| docs | 1.2.0 | Project documentation commands |
| tech-stack | 1.0.0 | Language/framework best practices setup |
| mcp | 1.2.1 | MCP server integration setup |
| fpf | 1.1.1 | First Principles Framework for structured reasoning |

## taches-cc-resources

**Source:** `https://github.com/glittercowboy/taches-cc-resources`

| Plugin | Version | Description |
|--------|---------|-------------|
| taches-cc-resources | 1.0.0 | Prompt engineering, MCP servers, subagents, hooks |

## thedotmack (claude-mem)

**Source:** `https://github.com/thedotmack/claude-mem`

| Plugin | Version | Description |
|--------|---------|-------------|
| claude-mem | 10.5.2 | Persistent memory system - context compression across sessions |

## claude-plugins-official

Built-in marketplace (may not require manual install).

Official Anthropic plugin marketplace. Contains many plugins — install the marketplace and enable individually.

## claude-code-lsps

**Source:** `https://github.com/Piebald-AI/claude-code-lsps`

LSP integrations for TypeScript, Python, Go, Rust, and more.

---

## Quick Setup Script

```bash
# Install all marketplaces on a new workstation
claude plugin add https://github.com/obra/superpowers-marketplace
claude plugin add https://github.com/NeoLabHQ/context-engineering-kit
claude plugin add https://github.com/glittercowboy/taches-cc-resources
claude plugin add https://github.com/thedotmack/claude-mem
claude plugin add claude-plugins-official  # Built-in marketplace
claude plugin add https://github.com/Piebald-AI/claude-code-lsps

# Install this personal plugin
claude plugin add https://github.com/JediNight/JediSkills
```

> URLs verified from Git remotes on 2026-03-26.
