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
- Remove `spring` from all Gemfiles of Rails 7+ projects in this monorepo

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

Always compile CSS from a host app (e.g. neurobetter), not from panda-core directly. The ModuleRegistry pattern means each gem registers its view paths; compiling from panda-core alone misses classes from panda-cms, cms-pro, etc. Output goes to `panda-core/public/panda-core-assets/panda-core.css`.

- **From a host app** (preferred): `bundle exec rake panda:compile_css`
- **From the monorepo root**: `bin/panda css compile` (uses panda-core's dummy app)
- After compiling, commit/push CSS changes in panda-core, then run `bin/panda deps sync`

**Full details:** [docs/css-compilation.md](docs/css-compilation.md)

### JavaScript Architecture

All panda gems use importmap-based ES modules served via Rack middleware — no compilation or bundling. Files live in `app/javascript/panda/[gem]/` and are served directly.

**Full details:** [docs/javascript-architecture.md](docs/javascript-architecture.md)

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

## CI Troubleshooting

- "Ferrum::ProcessTimeoutError: Browser did not produce websocket url within 10 seconds" — these are normally JavaScript failures or asset issues in GitHub CI, not Ferrum configuration problems.

## API Security Checklist

When writing or reviewing API controllers, verify every item:

- **Authorization on all actions**: Every endpoint needs a permission check (including read-only index/show). Don't assume authentication alone is sufficient.
- **IDOR prevention**: When accepting resource IDs from the client (e.g. blob IDs, user IDs), verify ownership or require an elevated permission before using them.
- **Double-render guards**: If a helper method (e.g. `attach_og_image`) can call `render_forbidden`, add `return if performed?` after calling it in the action to prevent `AbstractController::DoubleRenderError`.
- **Strong parameters for nested hashes**: When accepting arbitrary key-value hashes (e.g. `assets: { "field" => "blob_id" }`), use `assets: {}` in `permit()`.
- **Use permitted params, not raw params**: After calling `permit()`, reference the permitted result (`payload[:assets]`) rather than `params[:collection_item][:assets]` to avoid bypassing parameter filtering.
- **Base64 size validation**: Ruby's `Base64.encode64` inserts newlines every 60 chars. Strip whitespace before measuring size: `data.gsub(/\s+/, "")`.
- **Metadata preservation on dedup**: When deduplicating records by checksum, use `||=` for metadata fields to avoid overwriting the original uploader's info.
- **Auth scheme consistency**: Match the auth scheme the server expects (`Token` for `authenticate_or_request_with_http_token`, `Bearer` for OAuth).
- **N+1 queries**: Use batch queries (`.group().count`) or eager-loading instead of calling `.count` inside loops. For in-memory filtering, memoize with `@var ||=` or extract to a method.
- **Direct blob attachment**: Prefer `item.assets.attach(blob)` over downloading and re-uploading (`io: StringIO.new(blob.download)`) to avoid double storage.