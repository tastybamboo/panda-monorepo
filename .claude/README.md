# Claude Code Configuration for Panda CMS

This directory contains custom slash commands, subagents, and hooks for the Panda CMS monorepo.

## Directory Structure

```
.claude/
├── README.md                    # This file
├── commands/                    # Slash commands
│   ├── gem-release.md          # Complete gem release workflow
│   ├── bump-version.md         # Version bump helper
│   ├── check-dependencies.md   # Dependency audit
│   ├── test-all.md             # Run all tests
│   ├── test-gem.md             # Test specific gem
│   ├── test-quick.md           # Quick fail-fast tests
│   ├── migration-create.md     # Create Rails migration
│   ├── rails-console.md        # Open Rails console
│   ├── route-audit.md          # Audit routes
│   └── db-reset.md             # Reset test database
├── agents/                      # Subagents
│   ├── rspec-analyzer.md       # Test failure analysis
│   ├── migration-expert.md     # Migration specialist
│   └── gem-dependency-resolver.md  # Version conflict resolver
└── hooks/                       # Automation hooks
    └── pre-prompt.sh           # Syncs CLAUDE.md from GitHub
```

## Quick Start

### Using Slash Commands

In Claude Code, type `/` to see available commands:

```
/gem-release cms              # Release panda-cms gem
/test-all                     # Run all tests
/check-dependencies           # Audit gem dependencies
/migration-create cms AddPublishedAt  # Create migration
```

### Using Subagents

Subagents are invoked automatically or manually:

```
"Use rspec-analyzer subagent to investigate these test failures"
"Use migration-expert to review this migration"
"Use gem-dependency-resolver to fix version conflicts"
```

## Full Documentation

See [/docs/claude-code-workflows.md](../docs/claude-code-workflows.md) for:
- Detailed command documentation
- Subagent capabilities
- Workflow examples
- Best practices
- Troubleshooting

## Adding Custom Commands

1. Create a `.md` file in `commands/`
2. Add frontmatter with description
3. Write your prompt with `$1`, `$2` for arguments
4. Use: `/your-command-name`

Example:
```markdown
---
description: What your command does
---

Your command instructions here.
Use $1 for first argument.
```

## Adding Custom Subagents

1. Create a `.md` file in `agents/`
2. Add frontmatter with name, description, tools
3. Write system prompt explaining expertise
4. Use: "Use your-agent-name subagent to..."

Example:
```markdown
---
name: your-agent
description: When to invoke this agent
tools:
  - Bash
  - Read
---

You are an expert in...
```

## Project-Specific Notes

- All commands respect CLAUDE.md rules (no local paths, bundle update, etc.)
- Commands use proper PATH for mise-managed Ruby
- Database migrations follow RAILS_ENV=test patterns
- Table naming uses proper prefixes (panda_cms_, panda_core_, etc.)

## Getting Help

- `/help` in Claude Code for built-in help
- See docs/claude-code-workflows.md for full guide
- Check [Claude Code docs](https://docs.claude.com/en/docs/claude-code)
