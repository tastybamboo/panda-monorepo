---
description: Complete gem release workflow with version bump, testing, and publishing
---

Release gem **$1** with these steps:

## Pre-Release Checks
1. Check current version in `$1/lib/panda/$1/version.rb`
2. Ask user for version bump type using AskUserQuestion:
   - **patch** - Bug fixes (e.g., current: 1.2.3 → new: 1.2.4)
   - **minor** - New features, backwards compatible (e.g., current: 1.2.3 → new: 1.3.0)
   - **major** - Breaking changes (e.g., current: 1.2.3 → new: 2.0.0)
   - **other** - Custom version number (free text entry)
3. Calculate and show the new version number based on selection
4. Verify all tests pass before proceeding
5. Check git status - ensure working directory is clean

## Version Update
6. Update `version.rb` with the new version
7. Per CLAUDE.md: Run `bundle update` in the root panda directory
8. Update relevant `CHANGELOG.md` if it exists:
   - Fetch latest changes: `git fetch origin main --tags`
   - Find last release tag: `git describe --tags --abbrev=0 --match "v*" 2>/dev/null || echo "Initial release"`
   - Get unreleased commits from origin/main: `git log <last_tag>..origin/main --oneline --no-merges`
   - Format commits into changelog grouped by type (feat, fix, chore, etc.)
   - Add new version section at top with date and unreleased commits

## Testing
9. Run full test suite for the gem using the proper PATH and bundle exec rspec
10. If test failures occur, STOP and report issues
11. Run yamllint if any YAML files were modified (per CLAUDE.md)

## Git Operations
12. Stage changes: `git add $1/lib/panda/$1/version.rb Gemfile.lock`
13. Stage CHANGELOG if updated
14. Create commit: `chore: Release panda-$1 v{NEW_VERSION}`
15. Create git tag: `v{NEW_VERSION}`
16. Push git commits and tags: `git push && git push --tags`

## Monitor CI/CD Pipeline
17. Monitor GitHub Actions workflow using `gh run watch` or `gh run list`
18. Wait for all checks to pass (tests, linting, asset pipeline, gem build)
19. Once GitHub Actions completes successfully, wait additional 60-90 seconds for RubyGems publication
20. Poll RubyGems in 5-10 second intervals to confirm new version is available: `gem search panda-$1 -r`
21. Verify the new version appears on RubyGems.org

## Post-Release
22. Once gem is confirmed on RubyGems, identify dependent gems in the monorepo
23. For each dependent gem, run `bundle update panda-$1` in that gem's directory
24. Show which gems were updated and their new lock file versions
25. Summarize what was released and next steps

**IMPORTANT**: Never skip the testing phase. Never push to GitHub with Gemfiles that reference local paths (per CLAUDE.md).
