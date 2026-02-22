# CSS Compilation Guide

This document consolidates all CSS compilation documentation for the Panda ecosystem.

## Overview

Panda uses a unified CSS compilation system that compiles Tailwind CSS for **all registered Panda modules** into a single consolidated CSS file. The ModuleRegistry system allows each Panda gem to register its view templates and components, which are then automatically included in the CSS compilation.

## The ModuleRegistry System

Each Panda gem registers itself during engine initialization:

```ruby
# In panda-core/lib/panda/core/engine.rb
Panda::Core::ModuleRegistry.register(
  gem_name: "panda-core",
  engine: "Panda::Core::Engine",
  paths: {
    builders: "app/builders/panda/core/**/*.rb",
    components: "app/components/panda/core/**/*.rb",
    helpers: "app/helpers/panda/core/**/*.rb",
    views: "app/views/panda/core/**/*.erb",
    javascripts: "app/assets/javascript/panda/core/**/*.js"
  }
)
```

When CSS compilation runs, it automatically discovers and includes **all loaded modules** (panda-core, panda-cms, cms-pro, etc.).

## Compiling CSS

### From a host app (preferred)

```bash
bundle exec rake panda:compile_css
```

Run from neurobetter or another host app that loads all panda modules. This ensures Tailwind sees every template across all gems.

### From the monorepo root

```bash
bin/panda css compile
```

Uses panda-core's dummy app instead of a host app. May not see classes from host-app-specific templates.

### From any panda gem directory

```bash
bundle exec rake app:panda:compile_css

# Or from spec/dummy directory
bundle exec rake panda:compile_css

# Or via the convenience wrapper
bin/compile-css
```

## What Happens During Compilation

1. Task queries ModuleRegistry for all loaded Panda modules
2. Builds Tailwind CLI command scanning all registered file paths
3. Compiles using Tailwind CSS v4 with minification
4. **Always outputs to panda-core** (auto-locates the gem):
   - `core/public/panda-core-assets/panda-core.css` (~72 KB)
   - `core/public/panda-core-assets/panda-core-{version}.css`

**Key insight:** Running the task from panda-cms will still output CSS to panda-core's directory. The task finds panda-core using `Gem::Specification.find_by_name` and puts the CSS there automatically.

## Included Content

The compiled CSS includes Tailwind classes from:

- **panda-core**: Admin UI, forms, buttons, layouts, components
- **panda-cms**: CMS views and components (when loaded)
- **cms-pro**: Pro features (when loaded)
- **Any registered modules**: Via ModuleRegistry

This single-file approach reduces HTTP requests and ensures consistent styling across the entire Panda ecosystem.

## Tree-Shaking Behaviour

Tailwind CSS v4 performs aggressive tree-shaking based on ALL scanned files:

- Compiling from **a host app** (which scans core + cms + pro files) produces a smaller, more optimized CSS file (~50 KB minified)
- Compiling from **panda-core alone** produces a larger file (~72 KB minified) because Tailwind includes utilities that MIGHT be used by unknown consumers

When scanning all modules together, Tailwind knows exactly what's used and removes unused utilities (e.g., unused color variables, container sizes, font weights, margin utilities).

## When to Compile

- After changing Tailwind classes in any Panda gem
- Before creating a release
- When tests need updated CSS
- After adding new components or views

## After Compilation

1. Commit and push the CSS changes in panda-core
2. Run `bin/panda deps sync` or `bundle update panda-core` in downstream gems to pick up the new commit

## Tailwind Configuration

- **Source:** `app/assets/tailwind/application.css`
- **Version:** Tailwind CSS v4 (CSS-based config with `@theme`)
- **Themes:** `default` (purple), `sky` (blue)

## Why panda-cms/public/panda-core-assets/ is empty

This is expected! panda-cms doesn't generate its own CSS file. All Panda ecosystem CSS is consolidated in panda-core for reduced HTTP requests, consistent styling, and a single source of truth for the design system.

## Host App Integration (neurobetter)

When CSS changes are made in any panda gem:

```bash
# From the neurobetter directory
rake panda:compile_css
```

This compiles CSS and updates the panda-core gem in neurobetter's bundle. Remember to restart the Rails server afterwards.
