---
description: Reset test database for a gem (drop, create, migrate)
---

Reset test database for **$1** gem:

## Confirmation
1. **IMPORTANT**: This will destroy all test data
2. Ask user: "This will drop and recreate the test database. Continue? (yes/no)"
3. If not confirmed, abort

## Reset Process
4. Navigate to `$1` directory

5. Execute reset sequence with proper PATH:
   ```bash
   PATH="/Users/james/.local/share/mise/installs/ruby/3.4.5/bin:$PATH" RAILS_ENV=test bundle exec rails db:drop
   ```

6. Create fresh database:
   ```bash
   PATH="/Users/james/.local/share/mise/installs/ruby/3.4.5/bin:$PATH" RAILS_ENV=test bundle exec rails db:create
   ```

7. Run all migrations:
   ```bash
   PATH="/Users/james/.local/share/mise/installs/ruby/3.4.5/bin:$PATH" RAILS_ENV=test bundle exec rails db:migrate
   ```

8. Optionally load schema instead (faster):
   ```bash
   PATH="/Users/james/.local/share/mise/installs/ruby/3.4.5/bin:$PATH" RAILS_ENV=test bundle exec rails db:schema:load
   ```

## Verify
9. Check migration status:
   ```bash
   PATH="/Users/james/.local/share/mise/installs/ruby/3.4.5/bin:$PATH" RAILS_ENV=test bundle exec rails db:migrate:status
   ```

10. Confirm all migrations are "up"

## Post-Reset
11. Suggest running a quick test to verify everything works:
    - "Would you like me to run `/test-quick $1` to verify the database is working?"

12. Return to root: `cd /Users/james/Projects/panda`

## When to Use
- Test database is in a weird state
- Migrations are out of sync
- Starting fresh after major schema changes
- Tests are failing due to database issues

## Usage
- `/db-reset cms` - Reset panda-cms test database
- `/db-reset core` - Reset panda-core test database

**Tip**: If tests are failing mysteriously, try this command first. Database state issues are a common culprit.
