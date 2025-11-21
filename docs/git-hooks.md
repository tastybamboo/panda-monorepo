# Git Hooks for Panda Monorepo

## Overview

The Panda monorepo uses Git hooks to prevent common mistakes when working with multiple gems. These hooks run automatically at specific points in your Git workflow to catch issues before they reach the remote repository.

## Pre-Push Hook: Path Reference Checker

### What It Does

The pre-push hook automatically scans all panda gem Gemfiles for local path references before allowing a push to the remote repository. This prevents a common issue where developers forget to change development path references to version references before releasing or pushing.

### Why This Matters

Local path references like `gem "panda-core", path: "../core"` work great for local development but cause problems when:
- Code is pushed to CI/CD pipelines (can't access local paths)
- Other developers clone the repository (paths don't exist on their machines)
- Gems are published to RubyGems (path references are meaningless)

### Installation

Install the hook by running from the repository root:

```bash
bin/setup-hooks
```

This creates a pre-push hook in `.git/hooks/pre-push` that will run automatically before each `git push`.

Alternatively, run the complete setup which includes hook installation:

```bash
bin/setup
```

### What It Checks

The hook intelligently scans Gemfiles in these directories:
- `core/` (panda-core gem)
- `cms/` (panda-cms gem)
- `editor/` (panda-editor gem)
- `panda-cms-pro/` (panda-cms-pro gem)
- `panda-ci/` (panda-ci gem)

**Smart Filtering** (v2.0): The hook automatically:
- ✓ **Skips separate repositories**: Directories with their own `.git/` folder are skipped
- ✓ **Only checks tracked files**: Only scans files tracked by the current repository
- ✓ **Clear feedback**: Shows what was checked vs. skipped

This means:
- When pushing from the **monorepo root** (settings repo), it skips nested gem repos
- When pushing from **within a gem directory**, it checks that gem's Gemfile
- Install the hook in each gem repo where you want this protection

### Example Output

#### ✅ Successful Push (No Issues - Monorepo Root)
```bash
$ git push

🔍 Checking for local path references in panda gem Gemfiles...
⊳ Skipping core (separate repository)
⊳ Skipping cms (separate repository)
⊳ Skipping editor (separate repository)
⊳ Skipping panda-cms-pro (separate repository)

✅ No panda gem Gemfiles in this repository
   Skipped 4 separate repositories
```

#### ✅ Successful Push (No Issues - Within Gem)
```bash
$ cd cms && git push

🔍 Checking for local path references in panda gem Gemfiles...

✅ All checked Gemfiles are clean (1 file checked)
```

#### ❌ Blocked Push (Issues Found)
```bash
$ git push

🔍 Checking for local path references in panda gem Gemfiles...

❌ VIOLATION FOUND: cms/Gemfile
   Found local path references:
   → gem "panda-core", path: "../core"

   Please change to version references before pushing:
   gem "panda-core", "~> 0.X.Y"

❌ VIOLATION FOUND: panda-cms-pro/Gemfile
   Found local path references:
   → gem "panda-core", path: "../core"
   → gem "panda-cms", path: "../cms"
   → gem "panda-editor", path: "../editor"

   Please change to version references before pushing:
   gem "panda-core", "~> 0.X.Y"
   gem "panda-cms", "~> 0.X.Y"
   gem "panda-editor", "~> 0.X.Y"

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
🚫 PUSH BLOCKED
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

Found 2 gem(s) with local path references.

To fix this:
  1. Update the Gemfile(s) to use version references
  2. Ensure the referenced gems are published to RubyGems
  3. Run 'bundle install' to update Gemfile.lock
  4. Commit the changes
  5. Try pushing again

Tip: For local development, you can use path references,
but they must be changed to version references before pushing.
```

### How to Fix Issues

When the hook blocks your push, you need to:

1. **Update the Gemfile** from path reference to version reference:
   ```ruby
   # Before (local development)
   gem "panda-core", path: "../core"

   # After (for pushing)
   gem "panda-core", "~> 0.3.0"
   ```

2. **Update dependencies**:
   ```bash
   bundle install
   ```

3. **Commit the changes**:
   ```bash
   git add Gemfile Gemfile.lock
   git commit -m "Update panda-core to version reference"
   ```

4. **Try pushing again**:
   ```bash
   git push
   ```

### Bypassing the Hook (Not Recommended)

In exceptional cases where you need to bypass the hook, you can use:

```bash
git push --no-verify
```

**⚠️ Warning**: Only use this if you understand the consequences. Pushing path references will likely cause CI failures and issues for other developers.

### Workflow Pattern

Here's the recommended workflow when working with the panda gems:

#### During Development
```ruby
# Gemfile - Use path references for active development
gem "panda-core", path: "../core"
gem "panda-cms", path: "../cms"
```

#### Before Releasing/Pushing
```ruby
# Gemfile - Switch to version references
gem "panda-core", "~> 0.3.0"
gem "panda-cms", "~> 0.8.0"
```

#### After Pulling Changes
```ruby
# Gemfile - Switch back to path references for development
gem "panda-core", path: "../core"
gem "panda-cms", path: "../cms"
```

### Automation Tips

You can create git aliases to make this easier:

```bash
# Add to your ~/.gitconfig
[alias]
  check-paths = !.git/hooks/pre-push
  push-safe = "!f() { .git/hooks/pre-push && git push \"$@\"; }; f"
```

Then use:
```bash
git check-paths  # Manually check for path references
git push-safe    # Check and push in one command
```

## Troubleshooting

### Hook Not Running

If the hook doesn't run automatically:

1. **Check if it's installed**:
   ```bash
   ls -la .git/hooks/pre-push
   ```

2. **Verify it's executable**:
   ```bash
   chmod +x .git/hooks/pre-push
   ```

3. **Test manually**:
   ```bash
   .git/hooks/pre-push
   ```

### Hook Fails with Permission Error

```bash
chmod +x .git/hooks/pre-push
```

### Need to Update the Hook

Re-run the setup script:
```bash
bin/setup-hooks
```

It will prompt before overwriting an existing hook.

## Adding More Gems

If you add a new panda gem to the monorepo, update both:

1. **The hook script** (`.git/hooks/pre-push`):
   ```bash
   PANDA_GEMS=(
     "core"
     "cms"
     "editor"
     "panda-cms-pro"
     "panda-ci"
     "your-new-gem"  # Add here
   )
   ```

2. **The setup script** (`bin/setup-hooks`):
   Same array in the HOOK_CONTENT section.

Then re-run `bin/setup-hooks` to update your local hook.

## Technical Details

### How the Hook Works

1. **Trigger**: Runs automatically before `git push` completes
2. **Scan**: Uses `grep` to search for pattern `gem.*"panda-.*path:.*"\.\.`
3. **Report**: Collects all violations and displays them with colors
4. **Exit**: Returns non-zero exit code to block the push if violations found

### Pattern Matching

The hook looks for lines matching:
```
gem "panda-*", path: "../*"
```

This catches:
- `gem "panda-core", path: "../core"`
- `gem 'panda-cms', path: "../cms"`
- Variations with different whitespace

### Exit Codes

- `0`: All checks passed, push allowed
- `1`: Violations found, push blocked

## Related Documentation

- [Repository README](../README.md) - Main repository documentation
- [CLAUDE.md](../CLAUDE.md) - Claude Code instructions (mentions this hook)
- [Release Process](release-process.md) - When to use path vs version references

## Questions?

If you have questions about the git hooks or suggestions for improvements:
1. Open an issue in the repository
2. Update this documentation with your findings
3. Share the improvement with the team
