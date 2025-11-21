# Claude Code Instructions

## Initial Setup

- **New to this repository?** Run `bin/setup` to configure everything automatically:
  - Clones/updates all panda gem projects
  - Installs git hooks for safety checks
  - Syncs Claude Code settings
- If `bin/setup` or `bin/setup-hooks` don't exist, suggest running them to the user

## Development Guidelines

- Never push to GitHub with Gemfiles which reference a local path
  - **Automated Protection**: A git pre-push hook automatically prevents pushing panda gems with path references
  - Run `bin/setup-hooks` to install the hook if not already set up (or use `bin/setup` for complete setup)
  - The hook checks: core, cms, editor, cms-pro, and ci-tooling directories
- When we update version.rb in any project, we need to also run a bundle update
- Always run "yamllint -c .yamllint ." if you make changes to .yml or .yaml files.

## PR Readiness Checker Agent

- Use the `pr-readiness-checker` agent when preparing to raise pull requests
- The agent validates code changes, runs tests, checks linting, and ensures CI requirements are met
- **Important**: The agent should always merge in the latest main branch before pushing changes
- This agent helps prevent CI failures by running equivalent checks locally first
- When making changes to panda- gems, check through all panda- gems (including panda-cms, panda-core, panda-editor, cms-pro, etc.) to see if changes are required there too.
- The panda gems shouldn't have knowledge about containing/host apps, and should not reference them directly.
- In panda-*, all icons have to be fa-solid or fab (for brands) because it's fontawesome free
- 1. Always commit the PostgreSQL schema.rb (since PostgreSQL is your primary database):
  git restore spec/dummy/db/schema.rb
  2. Use db:schema:load in CI ✅ Already done!
    - CI uses db:schema:load which loads the schema regardless of which database generated it
    - This is why the fix I made earlier works
  3. Document for contributors:
    - Use PostgreSQL for local development when running migrations
    - SQLite is fine for testing, but don't commit the schema.rb changes it generates