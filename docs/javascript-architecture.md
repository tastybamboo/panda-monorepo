# JavaScript Architecture

This document describes the JavaScript architecture used across all Panda gems.

## Overview

All Panda gems use an **importmap-based architecture** with individual ES modules. JavaScript files are served as individual modules via custom Rack middleware — **no compilation or bundling required**.

**Key principle:** JavaScript is NOT compiled. Individual `.js` files are served directly from `app/javascript/panda/[gem]/` in each gem.

## How It Works

1. **ModuleRegistry** — Each gem registers its JavaScript paths during engine initialization
2. **JavaScriptMiddleware** — Intercepts `/panda/*` requests and serves files from registered gems
3. **Importmap** — Browser loads modules using native ES imports
4. **No Build Step** — Files served directly, no webpack/esbuild/rollup needed

## Request Flow

```
Browser Request → GET /panda/core/controllers/toggle_controller.js
                ↓
JavaScriptMiddleware → Finds file in panda-core's app/javascript/
                ↓
Serves raw ES module → Content-Type: application/javascript
```

## Importmap Configuration

Each gem defines its JavaScript imports in `config/importmap.rb`:

```ruby
# panda-core/config/importmap.rb
pin "panda/core/application", to: "/panda/core/application.js"
pin_all_from Panda::Core::Engine.root.join("app/javascript/panda/core/controllers"),
             under: "controllers", to: "/panda/core/controllers"
```

```ruby
# panda-cms/config/importmap.rb
pin "panda/cms/application", to: "/panda/cms/application.js"
pin_all_from Panda::CMS::Engine.root.join("app/javascript/panda/cms/controllers"),
             under: "controllers", to: "/panda/cms/controllers"
```

## File Structure

```
app/javascript/panda/[gem]/
├── application.js          # Main entry point
├── controllers/            # Stimulus controllers
│   ├── toggle_controller.js
│   ├── menu_controller.js
│   └── ...
└── vendor/                 # Vendored dependencies (panda-core only)
    └── @hotwired--stimulus.js
```

## Benefits

- **No compilation** — Faster development iteration
- **Better caching** — Individual files = granular browser caching
- **Simpler debugging** — Source maps not needed, files are unmodified
- **Consistent across gems** — All Panda modules use same pattern
- **Native ES modules** — Leverages browser-native module loading

## Verification

```bash
# Check importmap is loaded (in Rails console)
Rails.application.config.importmap.draw

# Verify JavaScript files are accessible
curl http://localhost:3000/panda/cms/application.js

# Check ModuleRegistry
bundle exec rake app:panda:registered_modules
```

## Legacy Note

You may find old compiled bundle files (like `panda-core-0.1.16.js`) in test directories. These are **legacy artifacts** from before the importmap migration and should be ignored/deleted. Old rake tasks like `panda_cms:assets:compile` are also obsolete.
