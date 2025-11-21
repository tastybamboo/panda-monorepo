---
description: Check gem dependency compatibility across all Panda gems
---

Check dependency compatibility across all Panda gems:

## Scan All Gems
For each gem in the monorepo (core, editor, cms, cms-pro):

**Note:** `generator` and `ci-tooling` are not gems - generator is a Rails template and ci-tooling is Docker-based CI tooling.

1. **Read the gemspec file** (e.g., `panda-cms.gemspec`, `panda-core.gemspec`)
2. **Extract all panda-* gem dependencies** and their version constraints
3. **Read the actual version** from version.rb files:
   - `core/lib/panda/core/version.rb`
   - `editor/lib/panda/editor/version.rb`
   - `cms/lib/panda-cms/version.rb`
   - `cms-pro/lib/panda/cms/pro/version.rb`
4. **Record dependencies** in a structured format

## Analyze Compatibility
5. **Compare version constraints** across all gems:
   - If cms requires `panda-core ~> 1.0.0`, does core version match?
   - Check for conflicting version requirements
   - Identify outdated version constraints

6. **Check for circular dependencies**

7. **Verify Gemfile.lock consistency** in consuming apps (check sites/ directory)

## Check Bundle Status
8. **For each gem directory**, check if `bundle update` is needed:
   - Compare gemspec dependencies with Gemfile.lock (if it exists)
   - Check for outdated gems using `bundle outdated`
   - Flag if version.rb was recently changed but bundle wasn't updated

9. **For consuming apps in sites/**, verify:
   - Panda gem versions in Gemfile.lock match the actual gem versions
   - No stale bundler cache issues

## Report Findings
10. **Create a dependency matrix** showing:
    ```
    Gem         | Depends On          | Required Version | Actual Version | Status
    ------------|---------------------|------------------|----------------|--------
    panda-cms   | panda-core          | ~> 1.0.0        | 1.0.5         | ✓
    panda-cms   | panda-editor        | ~> 0.5.0        | 0.4.9         | ⚠️
    ```

11. **Highlight issues**:
    - Version mismatches (required vs actual)
    - Outdated constraints that should be bumped
    - Missing dependencies
    - Potential conflicts
    - Gems needing `bundle update`

12. **Suggest updates** if needed

## Validation
13. Check that no Gemfile/gemspec references local paths in production (per CLAUDE.md)
14. Provide specific `bundle update` commands for any gems that need updating
15. Note if any consuming apps in sites/ need `bundle update` after gem version changes

Provide a clear summary with actionable next steps.
