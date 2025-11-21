---
description: Audit routes across all Rails engines and check for conflicts
---

Audit routes across all Panda Rails engines:

## Collect Routes
For each Rails engine (cms, core):

1. Navigate to gem directory
2. Run: `PATH="/Users/james/.local/share/mise/installs/ruby/3.4.5/bin:$PATH" bundle exec rails routes`
3. Parse and categorize routes by:
   - Controller namespace
   - HTTP method
   - Path pattern
   - Route name

## Analyze
4. **Check for conflicts**:
   - Duplicate route names across engines
   - Overlapping path patterns
   - Ambiguous routes that could match multiple controllers

5. **Document route hierarchy**:
   - Show how engines are mounted (if applicable)
   - Show route precedence
   - Identify admin vs public routes

6. **Check route conventions**:
   - Are admin routes under `/admin`?
   - Are API routes under `/api`?
   - Are routes RESTful?

## Report
7. Create structured output:
   ```
   === Panda CMS Routes ===
   Admin Routes (15):
     - GET    /admin/pages          -> admin/pages#index
     - POST   /admin/pages          -> admin/pages#create
     ...

   Public Routes (8):
     - GET    /pages/:id            -> pages#show
     ...

   === Panda Core Routes ===
   Auth Routes (6):
     - GET    /sign-in              -> sessions#new
     ...

   === Conflicts ===
   ⚠️  Route name 'user_path' defined in both cms and core
   ⚠️  Path pattern '/settings' could match multiple controllers

   === Suggestions ===
   - Consider namespacing core auth routes under /auth/
   - Add route constraints for ambiguous patterns
   ```

8. **If no test app**, explain that routes should be tested in a host Rails application

9. Return to root: `cd /Users/james/Projects/panda`

## When to Use
- After adding new controllers
- Before releases
- When debugging routing issues
- When documenting the API
