---
description: Run tests for a specific gem with detailed output
---

Run tests for **$1** gem:

## Pre-Test Setup
1. Navigate to `$1` directory
2. Verify gem has spec directory and tests

## Database Check (for Rails engines)
3. Check if this gem has database dependencies
4. If yes, check test database status:
   - Does test database exist?
   - Are there pending migrations?
   - Run: `RAILS_ENV=test bundle exec rails db:migrate:status`

5. If migrations pending:
   - Run: `PATH="/Users/james/.local/share/mise/installs/ruby/3.4.5/bin:$PATH" RAILS_ENV=test bundle exec rails db:migrate`

## Run Tests
6. Execute RSpec with proper PATH:
   ```bash
   PATH="/Users/james/.local/share/mise/installs/ruby/3.4.5/bin:$PATH" bundle exec rspec --format documentation
   ```

7. If specific test file provided as $2, run only that file:
   ```bash
   PATH="/Users/james/.local/share/mise/installs/ruby/3.4.5/bin:$PATH" bundle exec rspec $2 --format documentation
   ```

## Analyze Results
8. **If all tests pass**:
   - Show summary with green checkmark
   - Return to root directory

9. **If tests fail**:
   - Show detailed failure output
   - Identify the failing spec files
   - Ask: "Would you like me to investigate these failures with the rspec-analyzer subagent?"

10. **If tests are pending**:
    - List pending specs
    - Ask if these should be implemented

## Return
11. `cd /Users/james/Projects/panda`

## Usage Examples
- `/test-gem cms` - Test all of panda-cms
- `/test-gem cms spec/models/page_spec.rb` - Test specific file

**Tip**: This uses `--format documentation` for detailed output. For faster runs with less output, edit the command to use `--format progress`.
