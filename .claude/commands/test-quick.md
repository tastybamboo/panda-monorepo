---
description: Quick test run with --fail-fast to catch first failure
---

Quick test run for **$1** gem with fail-fast:

## Purpose
Run tests but stop at the first failure for rapid feedback.

## Execute
1. Navigate to `$1` directory
2. Run with fail-fast flag:
   ```bash
   PATH="/Users/james/.local/share/mise/installs/ruby/3.4.5/bin:$PATH" bundle exec rspec --format progress --fail-fast
   ```

## Report
3. **If passes**: Great! All tests pass.

4. **If fails**:
   - Show the first failure in detail
   - Show the exact command to re-run just that spec:
     ```
     cd $1 && PATH="..." bundle exec rspec spec/path/to/spec.rb:LINE --format documentation
     ```
   - Ask: "Would you like me to fix this failure?"

5. Return to root: `cd /Users/james/Projects/panda`

## When to Use
- After making small changes - catch errors immediately
- Before committing - quick sanity check
- When you want to fix issues one at a time

## When NOT to Use
- Before releases - use `/test-all` instead
- For CI verification - use full test suite
- When you need to see all failures at once
