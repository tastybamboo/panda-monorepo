# Claude Code Workflows for Panda CMS

This document describes the custom slash commands and subagents configured for the Panda CMS monorepo.

## Overview

The Panda CMS project uses Claude Code with custom workflows tailored for Rails engine development in a monorepo structure. These tools help automate common tasks and provide specialized assistance for complex operations.

## Quick Reference

### Slash Commands

| Command | Purpose | Example |
|---------|---------|---------|
| `/gem-release $1` | Complete gem release workflow | `/gem-release cms` |
| `/bump-version $1 $2` | Bump gem version | `/bump-version core 1.2.0` |
| `/check-dependencies` | Audit gem dependency compatibility | `/check-dependencies` |
| `/test-all` | Run tests for all gems | `/test-all` |
| `/test-gem $1 [$2]` | Test specific gem or file | `/test-gem cms spec/models/page_spec.rb` |
| `/test-quick $1` | Fast test with fail-fast | `/test-quick cms` |
| `/migration-create $1 $2` | Create migration with namespacing | `/migration-create cms AddPublishedAt` |
| `/rails-console $1 [$2]` | Open Rails console | `/rails-console cms test` |
| `/route-audit` | Audit routes across engines | `/route-audit` |
| `/db-reset $1` | Reset test database | `/db-reset cms` |

### Subagents

| Agent | When It's Used | Invoke Manually |
|-------|----------------|-----------------|
| `rspec-analyzer` | Test failures need investigation | "Use rspec-analyzer subagent" |
| `migration-expert` | Creating/reviewing migrations | "Use migration-expert subagent" |
| `gem-dependency-resolver` | Version conflicts or dependency issues | "Use gem-dependency-resolver" |

---

## Slash Commands in Detail

### Gem Management

#### `/gem-release $1`
**Complete release workflow for a gem**

This command handles the entire release process:
1. Checks current version
2. Prompts for new version number
3. Runs full test suite
4. Updates version.rb and CHANGELOG
5. Runs `bundle update` (per CLAUDE.md)
6. Creates git commit and tag
7. Builds gem
8. Publishes to RubyGems (with confirmation)

**Example:**
```
/gem-release cms
```

**When to use:**
- Ready to publish a new version to RubyGems
- All tests passing
- CHANGELOG updated

**Safety features:**
- Stops if tests fail
- Requires confirmation before publishing
- Follows CLAUDE.md rules (no local paths, bundle update)

---

#### `/bump-version $1 $2`
**Update gem version without full release**

Simpler than `/gem-release` - just updates the version number and runs bundle update.

**Example:**
```
/bump-version core 1.2.0
```

**When to use:**
- Preparing for release but not ready to publish yet
- Updating version for dependent gems to reference
- Development version bumps

---

#### `/check-dependencies`
**Audit gem dependency compatibility**

Scans all gemspecs and creates a dependency matrix showing:
- Which gems depend on which
- Required vs actual versions
- Version mismatches
- Conflicts

**Example:**
```
/check-dependencies
```

**When to use:**
- Before releases
- After bumping gem versions
- When bundle install fails mysteriously
- Planning major version updates

**What it catches:**
- panda-cms requires `panda-core ~> 1.0.0` but core is at `1.1.0`
- Circular dependencies
- Local path references (forbidden by CLAUDE.md)

---

### Testing Workflows

#### `/test-all`
**Run tests for all gems in the monorepo**

Runs RSpec for every gem (cms, core, editor, generator, panda-ci, panda-cms-pro) and provides a summary table.

**Example:**
```
/test-all
```

**When to use:**
- Before creating pull requests
- After making changes that affect multiple gems
- Before releases
- CI simulation

**What it does:**
- Checks database migrations for each gem
- Runs migrations if needed
- Executes tests with proper PATH
- Reports pass/fail/pending for each
- Suggests using rspec-analyzer for failures

**Note:** This can take several minutes!

---

#### `/test-gem $1 [$2]`
**Test a specific gem with detailed output**

Tests one gem with documentation format for detailed feedback.

**Examples:**
```
/test-gem cms
/test-gem cms spec/models/page_spec.rb
/test-gem core spec/models/user_spec.rb:42
```

**When to use:**
- After making changes to a specific gem
- Debugging test failures
- Before committing gem-specific changes

**Features:**
- Handles database migrations automatically
- Uses documentation format for clear output
- Offers to investigate failures with rspec-analyzer
- Can run specific files or even specific line numbers

---

#### `/test-quick $1`
**Fast test run with fail-fast**

Runs tests but stops at the first failure for rapid feedback.

**Example:**
```
/test-quick cms
```

**When to use:**
- After small changes - catch errors immediately
- Before committing - quick sanity check
- TDD workflow - fix one failure at a time

**When NOT to use:**
- Before releases (use `/test-all`)
- When you need to see all failures

---

### Rails Development

#### `/migration-create $1 $2`
**Create Rails migration with proper namespacing**

Generates a migration and ensures it follows Panda conventions:
- Proper table prefixes (`panda_cms_`, `panda_core_`, etc.)
- Reversible migrations
- Appropriate indexes
- Foreign key constraints

**Examples:**
```
/migration-create cms AddPublishedAtToPages
/migration-create core CreateUsers
/migration-create cms AddUserRefToPages user:references
```

**What it does:**
1. Generates migration with rails generate
2. Reviews the migration for best practices
3. Checks table naming (must use panda_* prefix)
4. Runs migration in test environment
5. Tests rollback for reversibility

**Best practices enforced:**
- Table naming: `panda_cms_pages` not `pages`
- Indexes on foreign keys
- Reversible changes
- NOT NULL handled carefully

---

#### `/rails-console $1 [$2]`
**Open Rails console in a gem's context**

Starts a Rails console for a specific gem with helpful tips displayed.

**Examples:**
```
/rails-console cms
/rails-console core test
/rails-console cms production
```

**When to use:**
- Debugging data issues
- Testing model associations
- Experimenting with code
- Checking configuration

**Pro tip:** The command shows helpful console commands before launching:
```ruby
# Check loaded models
Panda::CMS.constants

# Test associations
Page.first.try(:user)

# Enable query logging
ActiveRecord::Base.logger = Logger.new(STDOUT)
```

---

#### `/route-audit`
**Analyze routes across all Rails engines**

Examines routing in all engines and checks for:
- Duplicate route names
- Overlapping paths
- Route conventions (admin under /admin, etc.)
- Potential conflicts

**Example:**
```
/route-audit
```

**When to use:**
- After adding new controllers
- Before releases
- Debugging routing issues
- Documenting the API

**Output includes:**
- Routes by engine
- Admin vs public routes
- Conflicts and warnings
- Suggestions for improvements

---

#### `/db-reset $1`
**Reset test database (drop, create, migrate)**

Nuclear option for database issues. Destroys and recreates the test database.

**Example:**
```
/db-reset cms
```

**When to use:**
- Test database is in a weird state
- Migrations are out of sync
- Starting fresh after major schema changes
- Tests failing mysteriously due to database

**Safety:**
- Asks for confirmation first
- Only affects test database
- Offers to run tests after reset

**Common scenario:**
```
Tests failing with weird errors?
↓
/db-reset cms
↓
/test-quick cms
↓
All green! ✓
```

---

## Subagents in Detail

Subagents are specialized AI assistants with their own context windows. They handle complex tasks while keeping your main conversation clean.

### `rspec-analyzer`
**Deep analysis of test failures**

**Expertise:**
- RSpec patterns and best practices
- Rails engine testing complexities
- Database transaction issues
- Fixture and factory patterns
- Common Rails testing pitfalls

**When it's invoked:**
- Claude sees test failures
- You ask to investigate test errors
- You manually invoke: "Use rspec-analyzer subagent to investigate these failures"

**What it does:**
1. Reads failure output carefully
2. Examines test code
3. Investigates implementation
4. Checks database state
5. Looks for common issues:
   - N+1 queries causing timeout
   - Transaction issues in system tests
   - Timezone problems
   - Fixture/factory conflicts
   - Missing test data
   - Spring holding stale code

**Output format:**
```
## Test Failure Analysis

**Failing Test**: spec/models/page_spec.rb:42
**Error**: Validation failed: Title can't be blank

## Root Cause
The test creates a Page without a title, but we added a
presence validation in commit abc123. The test needs updating.

## Recommended Fix
Update the test to include a title:
let(:page) { create(:page, title: "Test Page") }

## Prevention
Run full test suite after adding validations.
```

**Example workflow:**
```
You: Run tests for cms
Claude: Tests failing in 3 specs
You: Use rspec-analyzer to investigate
RSpec Analyzer: [detailed analysis]
Claude: [receives summary, continues with fix]
```

---

### `migration-expert`
**Database migration specialist**

**Expertise:**
- Rails migration best practices
- Database schema design
- Index optimization
- Foreign key constraints
- Multi-gem database coordination
- Table namespacing in engines

**When it's invoked:**
- Creating complex migrations
- Reviewing migration safety
- Database design questions
- You manually invoke: "Use migration-expert to review this migration"

**Critical rules it enforces:**

1. **Table Naming**
   ```ruby
   # ✓ Correct
   create_table :panda_cms_pages

   # ✗ Wrong
   create_table :pages
   ```

2. **Reversibility**
   ```ruby
   # ✓ Reversible
   def change
     add_column :panda_cms_pages, :published_at, :datetime
   end

   # ⚠️ Needs up/down
   def up
     # Complex data migration
   end

   def down
     # Reverse it
   end
   ```

3. **Foreign Keys**
   ```ruby
   # ✓ With constraint
   add_reference :panda_cms_pages, :user,
     foreign_key: { to_table: :panda_core_users }

   # ✗ Without constraint
   add_reference :panda_cms_pages, :user
   ```

**Example workflow:**
```
You: I need to add a user_id to pages
Claude: I'll use the migration-expert subagent
Migration Expert: [analyzes requirements, designs migration]
Claude: Here's the migration, tested for reversibility
```

---

### `gem-dependency-resolver`
**Version conflict resolution specialist**

**Expertise:**
- Semantic versioning
- Bundler and Gemfile.lock
- Gem version constraints
- Cross-gem dependency management

**When it's invoked:**
- Bundle install fails with conflicts
- Gem version incompatibilities
- Planning version bumps
- You manually invoke: "Use gem-dependency-resolver to fix this"

**Version constraint guide:**
```ruby
# Pessimistic versioning (recommended)
~> 1.0.0   # Allows >= 1.0.0 and < 1.1.0
~> 1.0     # Allows >= 1.0.0 and < 2.0.0

# Too permissive (avoid)
>= 1.0.0   # Any version >= 1.0.0

# Exact (rarely needed)
= 1.0.0    # Only 1.0.0
```

**Common scenarios:**

**Scenario 1: Version Mismatch**
```
Problem: panda-cms requires panda-core ~> 1.0.0
         but panda-core is at version 1.1.0

Solution: Update cms gemspec:
  spec.add_dependency "panda-core", "~> 1.1.0"

Reason: 1.1.0 is minor bump (backward compatible)
```

**Scenario 2: Breaking Change**
```
Problem: panda-core updated to 2.0.0 (breaking changes)

Solution:
  1. Review what broke in panda-cms
  2. Update panda-cms code for compatibility
  3. Update cms gemspec: "panda-core", "~> 2.0.0"
  4. Bump panda-cms MAJOR version (cascade)

Reason: Breaking changes require MAJOR version bump
```

**Example workflow:**
```
You: Bundle install is failing with version conflicts
Claude: I'll use gem-dependency-resolver
Resolver: [analyzes dependency graph]
Resolver: Found conflict: cms requires core ~> 1.0.0 but core is 1.1.0
Resolver: Update cms/panda-cms.gemspec line 24
Claude: Applied fix, running bundle update
Claude: ✓ Dependencies resolved
```

---

## Workflow Examples

### Releasing a Gem

**Full Release:**
```
1. Make changes to panda-cms
2. /test-gem cms                    # Verify tests pass
3. /check-dependencies              # Ensure no conflicts
4. Update CHANGELOG.md manually
5. /gem-release cms                 # Complete release workflow
   → Prompts for version (e.g., 1.2.0)
   → Runs tests
   → Updates version.rb
   → Runs bundle update
   → Creates commit and tag
   → Builds gem
   → Asks to publish
6. Done! Published to RubyGems
```

### Fixing a Test Failure

**Investigation Workflow:**
```
1. /test-gem cms
   → 3 failures in page_spec.rb

2. "Use rspec-analyzer subagent to investigate these failures"
   → Analyzer reads failures
   → Identifies root cause: missing validation
   → Suggests fix

3. Apply the fix
4. /test-quick cms                  # Verify fix
5. /test-all                        # Ensure nothing else broke
6. Commit
```

### Creating a New Feature with Migration

**Feature Development:**
```
1. "I need to add published_at to pages with a published scope"

2. /migration-create cms AddPublishedAtToPages published_at:datetime
   → Migration-expert subagent reviews
   → Checks table naming
   → Adds index
   → Tests reversibility

3. Update model (Panda::CMS::Page)
   → Add scope :published

4. /test-gem cms spec/models/page_spec.rb
   → Tests pass ✓

5. /test-all                        # Full test suite
6. /check-dependencies              # Verify no issues
7. Commit and push
```

### Debugging Cross-Gem Issues

**Complex Investigation:**
```
1. Changed Panda::Core::User model
2. /test-all
   → panda-core: ✓ All pass
   → panda-cms: ✗ 5 failures
   → panda-editor: ✓ All pass

3. "Use rspec-analyzer to investigate panda-cms failures"
   → Analyzer identifies: cms tests expect old User API
   → Suggests updating test fixtures

4. Fix fixtures in panda-cms
5. /test-gem cms                    # Verify
6. /check-dependencies              # Check if version bumps needed
   → Resolver suggests: Bump panda-cms version constraint for panda-core

7. Update cms gemspec
8. bundle update                    # Per CLAUDE.md
9. /test-all                        # Final verification
10. Commit both gems
```

---

## Best Practices

### When to Use Slash Commands vs Subagents

| Use Slash Commands | Use Subagents |
|-------------------|---------------|
| Quick, repetitive tasks | Complex analysis needed |
| Known workflow | Problem investigation |
| "Do this specific thing" | "Figure out what's wrong" |
| Fast feedback | Deep dive |

**Example:**
- `/test-gem cms` - Slash command (simple)
- "Use rspec-analyzer to figure out why this test is flaky" - Subagent (complex)

### Combining Commands

Commands and subagents work together:

```
/test-all
→ Failures in multiple gems
→ "Use rspec-analyzer for the cms failures"
→ Get analysis
→ Fix issues
→ /test-quick cms
→ "Use gem-dependency-resolver to check if versions need updating"
→ Update gemspecs
→ bundle update
→ /test-all
→ All green! ✓
```

### Following CLAUDE.md Rules

All commands respect your project rules:
- ✓ Never push with local path Gemfile references
- ✓ Always run `bundle update` after version.rb changes
- ✓ Run yamllint on YAML changes
- ✓ Use pr-readiness-checker before PRs

---

## Troubleshooting

### Commands Not Available

**Problem:** Commands not showing up

**Solutions:**
```bash
# Check commands directory exists
ls -la .claude/commands

# Verify command files have .md extension
ls .claude/commands/*.md

# Restart Claude Code if needed
```

### Subagents Not Responding

**Problem:** Subagent not being invoked

**Solutions:**
- Be explicit: "Use [subagent-name] subagent to..."
- Check .claude/agents/*.md files exist
- Restart Claude Code

### Tests Failing with Database Issues

**Try this sequence:**
```
/db-reset [gem-name]
/test-quick [gem-name]
```

If still failing:
```
"Use rspec-analyzer to investigate"
```

### Bundle Install Fails

**Try this sequence:**
```
/check-dependencies
"Use gem-dependency-resolver to fix conflicts"
bundle update
```

---

## Customization

### Adding Your Own Commands

Create `.claude/commands/your-command.md`:

```markdown
---
description: What your command does
---

Your command instructions here.

Use $1, $2 for arguments.
Use $ARGUMENTS for all arguments.
```

Then use: `/your-command arg1 arg2`

### Adding Your Own Subagents

Create `.claude/agents/your-agent.md`:

```markdown
---
name: your-agent
description: When this agent should be used
tools:
  - Bash
  - Read
  - Edit
---

Your agent's system prompt here.
Explain its expertise and approach.
```

Then use: "Use your-agent subagent to..."

---

## Further Reading

- [Claude Code Documentation](https://docs.claude.com/en/docs/claude-code)
- [Slash Commands Guide](https://docs.claude.com/en/docs/claude-code/slash-commands)
- [Subagents Guide](https://docs.claude.com/en/docs/claude-code/sub-agents)
- [Semantic Versioning](https://semver.org/)

---

**Happy coding with Claude! 🐼**
