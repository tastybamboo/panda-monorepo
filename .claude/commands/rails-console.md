---
description: Open Rails console in a specific gem's context
---

Open Rails console for **$1** gem:

## Setup
1. Navigate to `$1` directory
2. Determine environment (default: development, or use $2 if provided)

## Launch Console
3. Start Rails console with proper PATH:
   ```bash
   cd $1 && PATH="/Users/james/.local/share/mise/installs/ruby/3.4.5/bin:$PATH" bundle exec rails console $ENVIRONMENT
   ```

## Pre-Console Tips
4. Before launching, show user these helpful tips:
   ```ruby
   # Useful commands in the console:

   # Check loaded models
   Panda::CMS.constants
   Panda::Core.constants

   # Test associations
   Page.first.try(:user)

   # Check configuration
   Panda::CMS.configuration

   # Reload code
   reload!

   # Find routes
   Rails.application.routes.routes.map { |r| r.path.spec.to_s }

   # Database queries (verbose)
   ActiveRecord::Base.logger = Logger.new(STDOUT)
   ```

5. Execute the console command

## Usage Examples
- `/rails-console cms` - Open development console for panda-cms
- `/rails-console core test` - Open test console for panda-core
- `/rails-console cms production` - Open production console (be careful!)

**Note**: This command will block until you exit the console. Type `exit` in the console to return to Claude Code.
