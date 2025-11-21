---
description: Run tests for all gems in the monorepo
---

Run tests across all Panda gems:

## Setup
1. List all testable gems: cms, core, editor, generator, ci-tooling, cms-pro
2. Track results for each gem

## For Each Gem
For cms, core, editor, generator, ci-tooling, cms-pro:

3. **Navigate to gem directory**: `cd $GEM_NAME`

4. **Check database state** (for gems with Rails engines):
   - Check if test database exists
   - Check for pending migrations: `RAILS_ENV=test bundle exec rails db:migrate:status`
   - If migrations pending, run: `RAILS_ENV=test bundle exec rails db:migrate`

5. **Run tests** with proper PATH:
   ```bash
   PATH="/Users/james/.local/share/mise/installs/ruby/3.4.5/bin:$PATH" bundle exec rspec --format progress
   ```

6. **Capture results**:
   - Total examples
   - Failures
   - Pending
   - Duration

7. **If failures**: Save detailed failure output for analysis

## Summary Report
8. Create a summary table:
   ```
   Gem              | Examples | Passed | Failed | Pending | Duration | Status
   -----------------|----------|--------|--------|---------|----------|--------
   panda-cms        | 245      | 245    | 0      | 0       | 12.5s    | ✓
   panda-core       | 180      | 178    | 2      | 0       | 8.2s     | ✗
   panda-editor     | 95       | 95     | 0      | 3       | 5.1s     | ✓
   ```

9. **Overall status**: Pass/Fail

10. **If any failures**:
    - Suggest using the `rspec-analyzer` subagent for deep investigation
    - Show first few failure messages
    - Provide command to re-run just the failures

11. **Return to root directory**: `cd /Users/james/Projects/panda`

## Notes
- This command runs ALL tests - it may take several minutes
- For faster feedback, test individual gems with `cd GEM && bundle exec rspec`
- Database migrations are handled automatically per CLAUDE.md patterns
