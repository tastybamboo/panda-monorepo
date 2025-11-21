---
description: Create a Rails migration in a gem with proper namespacing
---

Create migration **$2** in gem **$1**:

## Generate Migration
1. Navigate to `$1` directory
2. Generate the migration:
   ```bash
   PATH="/Users/james/.local/share/mise/installs/ruby/3.4.5/bin:$PATH" bundle exec rails generate migration $2
   ```

3. Find and read the generated migration file

## Review & Enhance
4. **Check table naming**:
   - Ensure tables use proper prefix: `panda_{gem_name}_tablename`
   - Example: In `cms` gem, use `panda_cms_pages`, not just `pages`

5. **Verify migration quality**:
   - Is it reversible? (Has `def change` or both `up` and `down`)
   - Are indexes added for foreign keys?
   - Are NOT NULL constraints appropriate?
   - Are default values needed?

6. **Show the migration** to user with any suggested improvements

7. **Ask**: "Does this migration look correct? Should I make any adjustments?"

## Apply Migration
8. If approved, run in test environment:
   ```bash
   PATH="/Users/james/.local/share/mise/installs/ruby/3.4.5/bin:$PATH" RAILS_ENV=test bundle exec rails db:migrate
   ```

9. **Verify migration applied**:
   - Check `db:migrate:status`
   - Confirm table/column exists

10. **Test reversibility**:
    - Ask: "Should I test rollback? (recommended)"
    - If yes: Run `db:rollback STEP=1` then `db:migrate` again

## Finalize
11. Return to root: `cd /Users/james/Projects/panda`
12. Remind: Update model/specs if needed

## Usage Examples
- `/migration-create cms AddPublishedAtToPages`
- `/migration-create core CreateUsers`
- `/migration-create cms AddUserRefToPages user:references`

**Best Practice**: Use descriptive migration names that explain what changes. Follow Rails conventions for automatic column type detection.
