# Panda Monorepo

A development workspace for the Panda CMS ecosystem, automatically managing multiple gem repositories and providing unified tooling.

## Quick Start

### Initial Setup

```bash
# Clone this repository
git clone git@github.com:tastybamboo/panda-monorepo.git panda
cd panda

# Run the initialization script
./init
```

The `init` script will:
- ✅ Auto-discover all panda-* gems from the tastybamboo GitHub organization
- ✅ Clone gems you have access to into `gems/` directory
- ✅ Clone automation tools into `automation/` directory
- ✅ Clone supporting repositories (generator)
- ✅ Install git hooks (including lefthook where configured)
- ✅ Set up the `panda` CLI tool

### Adding to Your PATH

For easy access to the `panda` CLI from anywhere:

```bash
# Add to ~/.zshrc or ~/.bashrc
export PATH="$HOME/Projects/panda/bin:$PATH"

# Reload your shell
source ~/.zshrc
```

## Directory Structure

```
panda/                          # Main monorepo (this repository)
├── README.md                   # This file
├── CLAUDE.md                   # Claude Code configuration
├── init                        # Bootstrap script (auto-discovers repos)
│
├── bin/                        # Executable wrappers
│   ├── panda                   # CLI tool wrapper
│   ├── setup                   # Refresh everything (calls init)
│   └── setup-hooks             # Install git hooks
│
├── scripts/                    # CLI tool implementation
│   └── panda                   # Main panda CLI tool
│
├── docs/                       # Documentation
│   ├── claude-code-workflows.md
│   └── git-hooks.md
│
├── .claude/                    # Claude Code config
│   ├── commands/               # Slash commands for development
│   └── agents/                 # Specialized agents
│
├── gems/                       # Auto-cloned panda gems (gitignored)
│   ├── panda-core/
│   ├── panda-cms/
│   ├── panda-editor/
│   └── panda-cms-pro/          # (optional, if you have access)
│
├── automation/                 # CI/CD and automation tools (gitignored)
│   ├── ci-tooling/
│   └── panda-assets-verify-action/
│
├── generator/                  # Rails app generator (gitignored)
└── sites/                      # Demo sites & consuming apps (gitignored)
    └── your-app/               # Your Rails apps using Panda gems go here
```

**Key points:**
- Gems and tools are **dynamically cloned** - not committed to this repo
- The `init` script **auto-discovers** repositories from GitHub
- Private repos (like panda-cms-pro) are **optional** - skipped if no access
- **sites/** directory is for your Rails apps that consume Panda gems

## Using the Panda CLI

The `panda` CLI provides utilities for working across all gems:

```bash
# Show help
panda --help

# List all discovered gems
panda gems list

# Compile CSS for all gems
panda css compile
```

### CSS Compilation

The monorepo provides unified CSS compilation across all panda gems:

```bash
# From anywhere in the monorepo
panda css compile
```

This:
- Scans all loaded panda modules (core, cms, cms-pro, etc.)
- Compiles Tailwind CSS with all utility classes
- Outputs to `gems/panda-core/public/panda-core-assets/panda-core.css`
- Creates versioned copy for releases

## Git Hooks

### Monorepo Hook

A pre-push hook prevents accidentally pushing Gemfiles with local path references:

```ruby
# ❌ This will be blocked:
gem "panda-core", path: "../panda-core"

# ✅ Use version references instead:
gem "panda-core", "~> 0.3.0"
```

### Lefthook (Per-Repository)

The setup automatically installs lefthook in repositories that have it configured:

- Discovers `lefthook.yml` or `.lefthook.yml` in any cloned repo
- Renames `.lefthook.yml` → `lefthook.yml` (if needed)
- Runs `lefthook install` to set up hooks

Install lefthook if you don't have it:

```bash
brew install lefthook
```

## Updating Everything

To refresh all repositories and reinstall hooks:

```bash
# From anywhere in the monorepo
bin/setup

# Or directly
./init
```

This will:
- Re-discover repositories from GitHub
- Pull latest changes in existing repos
- Clone any new repositories
- Reinstall all git hooks

## Working with Sites

The `sites/` directory is for Rails applications that consume Panda gems:

```bash
# Create a new Rails app using Panda
cd sites/
rails new my-app
cd my-app

# Add Panda gems to Gemfile
# For development, use path references:
gem "panda-core", path: "../../gems/panda-core"
gem "panda-cms", path: "../../gems/panda-cms"

# For production, use version references:
gem "panda-core", "~> 0.3.0"
gem "panda-cms", "~> 0.4.0"
```

**Note:** The git pre-push hook prevents accidentally pushing path references.

## Working on Individual Gems

Each gem is a separate git repository with its own workflow:

```bash
# Navigate to a gem
cd gems/panda-core

# Install dependencies
bundle install

# Run tests (from spec/dummy for engines)
cd spec/dummy
bundle exec rspec

# Make changes and commit
git add .
git commit -m "feat: Add new feature"
git push
```

## Claude Code Integration

This monorepo includes Claude Code configuration with:

### Custom Slash Commands

- `/test-gem` - Run tests for a specific gem
- `/test-all` - Run tests across all gems
- `/gem-release` - Release workflow for gems
- `/db-reset` - Reset test databases
- `/review-wip` - Review work in progress
- And more... (see `.claude/commands/`)

### Specialized Agents

- `gem-dependency-resolver` - Resolve dependency conflicts
- `migration-expert` - Rails migration assistance
- `rspec-analyzer` - Debug test failures

## Repository Discovery

The `init` script uses GitHub CLI to auto-discover repositories:

```bash
# Requirements
brew install gh jq

# Authenticate with GitHub
gh auth login
```

Discovery process:
1. Fetches all repositories from `tastybamboo` organization
2. Filters for `panda-*` prefixed repositories
3. Marks private repos as "optional"
4. Clones repos you have access to
5. Skips repos you don't have access to (gracefully)

## Contributing

### Adding a New Panda Gem

Just create it in the tastybamboo org with a `panda-*` name:

1. Create repo: `panda-your-feature`
2. Run `./init` in the monorepo
3. It will be auto-discovered and cloned

### Adding Documentation

Documentation lives in this repository:

```bash
# Edit docs
vim docs/your-topic.md

# Commit to this repo
git add docs/
git commit -m "docs: Add your-topic guide"
git push
```

### Claude Code Configuration

Update `CLAUDE.md` for AI assistance configuration:

```bash
# Edit the shared config
vim CLAUDE.md

# Commit and push
git add CLAUDE.md
git commit -m "feat: Update Claude Code configuration"
git push
```

## Troubleshooting

### Permission Denied on Private Repos

If you see "permission denied" for panda-cms-pro or other private repos:
- This is expected if you don't have access
- The repo will be skipped automatically
- Everything else continues to work

### Lefthook Not Installing

```bash
# Install lefthook
brew install lefthook

# Re-run setup
./init
```

### Git Hooks Not Working

```bash
# Reinstall hooks
bin/setup-hooks
```

### CSS Compilation Fails

```bash
# Ensure you're in a gem directory and dependencies are installed
cd gems/panda-core
bundle install

# Try compiling again
panda css compile
```

## Architecture Notes

**Why this structure?**

- **Dynamic discovery** - No manual maintenance of projects.json
- **Graceful degradation** - Works even if you can't access all repos
- **Simple architecture** - One init script, auto-discovers everything
- **Git-native** - Each gem is a normal git repository
- **Unified tooling** - Panda CLI works across all gems

**What's gitignored?**

Everything except the monorepo infrastructure:
- ✅ Committed: init, bin/, scripts/, docs/, CLAUDE.md, README.md
- ❌ Gitignored: gems/, automation/, generator/, sites/

This keeps the monorepo lightweight and focused on tooling, not code.

## License

Each gem has its own license. This monorepo infrastructure is MIT licensed.
