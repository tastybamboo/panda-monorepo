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
- Always run `bundle update --all` before committing changes
- Always run "yamllint -c .yamllint ." if you make changes to .yml or .yaml files.

## Panda CLI Tool

A monorepo CLI is available at `~/Projects/panda/bin/panda`.
Run from `~/Projects/panda`:

- `bin/panda css compile` — Compile CSS for all panda gems (uses panda-core's dummy app)
- `bin/panda gems list` — List all panda gems in the monorepo
- `bin/panda deps graph` — Show dependency graph in topological order
- `bin/panda deps check` — Check if gem lock files are in sync with upstream
- `bin/panda deps sync` — Update and push Gemfile.lock for all downstream gems

### When to run `bin/panda deps sync`

Run `bin/panda deps sync` after merging a PR in any upstream panda gem (e.g. panda-core, panda-editor).
It walks the dependency graph in topological order, runs `bundle update` for each downstream gem,
and commits/pushes the updated Gemfile.lock. This keeps all gems pointing at the latest revisions.

### Panda CSS Compilation

When Tailwind classes change in any panda gem template, CSS must be recompiled:

1. **From a host app** (preferred): `bundle exec rake panda:compile_css`
   - Run from neurobetter or another host app that loads all panda modules
   - This ensures Tailwind sees every template across all gems
2. **From the monorepo root**: `panda css compile`
   - Uses panda-core's dummy app instead of a host app
   - May not see classes from host-app-specific templates
3. Output goes to `panda-core/public/panda-core-assets/panda-core.css`
4. Commit and push the CSS changes in `panda-core`
5. Run `bin/panda deps sync` or `bundle update panda-core` in downstream gems to pick up the new commit

## PR Readiness Checker Agent

- Use the `pr-readiness-checker` agent when preparing to raise pull requests
- The agent validates code changes, runs tests, checks linting, and ensures CI requirements are met
- **Important**: The agent should always merge in the latest main branch before pushing changes
- This agent helps prevent CI failures by running equivalent checks locally first
- When making changes to panda- gems, check through all panda- gems (including panda-cms, panda-core, panda-editor, cms-pro, etc.) to see if changes are required there too.
- The panda gems shouldn't have knowledge about containing/host apps, and should not reference them directly.
- In panda-*, all icons have to be fa-solid or fab (for brands) because it's fontawesome free

## Admin UI: ViewComponent Requirements

All admin pages across panda gems and host apps (neurobetter, etc.) **must** use panda-core ViewComponents. Never write raw HTML with inline Tailwind classes for admin screens.

### Required Components for Admin Pages

| Component | Purpose | Usage |
|-----------|---------|-------|
| `ContainerComponent` | Page wrapper with heading | Every admin page must be wrapped in this |
| `SearchFilterBarComponent` | Search input + filter dropdowns | Use for any page with search/filter UI. Provides `renders_many :filters` slot and `select_classes` helper |
| `TableComponent` | Data tables with column DSL | Use instead of raw `<table>` HTML. Provides alternating rows, hover states, rounded card style |
| `TagComponent` | Status badges | Use for status/state indicators (`:active`, `:draft`, `:inactive`, `:auto`) |
| `EmptyStateComponent` | Empty state with icon/title | Use when a table/list has no results |
| `PaginationComponent` | Page numbers with prev/next | Use for any paginated listing |
| `ButtonComponent` | Styled buttons/links | Use for action buttons. For `button_to` (form submissions), apply ButtonComponent's CSS classes directly |
| `PanelComponent` | Content panels | Use for grouped form sections |

### Anti-Patterns (Never Do This)

- **No hardcoded color classes**: Never use `bg-blue-600`, `focus:border-blue-500`, etc. Use components which use `primary-600` design tokens
- **No raw `<table>` HTML**: Always use `TableComponent` with `table.column("Name") { |row| row.name }`
- **No inline status badges**: Never write `<span class="bg-green-100 text-green-800">`. Use `TagComponent`
- **No raw search forms**: Use `SearchFilterBarComponent` instead of building search/filter forms from scratch
- **No raw `<div class="p-6">` wrappers**: Use `ContainerComponent`

### Example: Standard Admin Index Page

```erb
<%%= render Panda::Core::Admin::ContainerComponent.new do |component| %>
  <%% component.with_heading_slot(text: "Items", level: 1) %>

  <%%= render Panda::Core::Admin::SearchFilterBarComponent.new(
    url: admin_items_path,
    search_value: params[:q],
    search_placeholder: "Search items...",
    show_clear: params[:q].present? || params[:status].present?
  ) do |bar| %>
    <%% bar.with_filter do %>
      <%%= select_tag :status, options_for_select([["All", ""], ["Active", "active"]], params[:status]),
        class: bar.select_classes %>
    <%% end %>
  <%% end %>

  <%% if @items.any? %>
    <%%= render Panda::Core::Admin::TableComponent.new(term: "item", rows: @items) do |table| %>
      <%% table.column("Name") { |item| item.name } %>
      <%% table.column("Status") { |item| render Panda::Core::Admin::TagComponent.new(text: item.status.humanize, status: item.active? ? :active : :draft) } %>
    <%% end %>
  <%% else %>
    <%%= render Panda::Core::Admin::EmptyStateComponent.new(
      icon: "fa-solid fa-box", title: "No items found",
      description: "Try adjusting your search criteria."
    ) %>
  <%% end %>

  <%%= render Panda::Core::Admin::PaginationComponent.new(
    page: @page, total_pages: @total_pages, total_count: @total_count,
    per_page: PER_PAGE, item_name: "items"
  ) %>
<%% end %>
```

### Reference Implementations

- **Gold standard**: `gems/panda-core/app/views/panda/core/admin/users/index.html.erb`
- **Host app example**: `sites/neurobetter/app/views/admin/members/onboarding/index.html.erb`
- **Components source**: `gems/panda-core/app/components/panda/core/admin/`
- 1. Always commit the PostgreSQL schema.rb (since PostgreSQL is your primary database):
  git restore spec/dummy/db/schema.rb
  2. Use db:schema:load in CI ✅ Already done!
    - CI uses db:schema:load which loads the schema regardless of which database generated it
    - This is why the fix I made earlier works
  3. Document for contributors:
    - Use PostgreSQL for local development when running migrations
    - SQLite is fine for testing, but don't commit the schema.rb changes it generates