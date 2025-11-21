# Contributing to Panda Monorepo

Thank you for your interest in contributing to the Panda CMS ecosystem! This guide will help you get started with contributing to the monorepo infrastructure and the individual Panda gems.

## Table of Contents

- [Code of Conduct](#code-of-conduct)
- [Getting Started](#getting-started)
- [Development Workflow](#development-workflow)
- [Contributing to Individual Gems](#contributing-to-individual-gems)
- [Contributing to Monorepo Infrastructure](#contributing-to-monorepo-infrastructure)
- [Code Quality Standards](#code-quality-standards)
- [Pull Request Process](#pull-request-process)
- [Reporting Issues](#reporting-issues)

## Code of Conduct

This project adheres to the Contributor Covenant Code of Conduct. By participating, you are expected to uphold this code. Please read [CODE_OF_CONDUCT.md](CODE_OF_CONDUCT.md) before contributing.

## Getting Started

### Prerequisites

Before you begin, ensure you have the following installed:

- **Ruby** (version specified in each gem's `.ruby-version`)
- **Git** for version control
- **[GitHub CLI](https://cli.github.com/)** (`gh`) for repository management
- **[jq](https://jqlang.github.io/jq/)** for JSON processing
- **[lefthook](https://github.com/evilmartians/lefthook)** for git hooks (optional but recommended)

Install the required tools:

```bash
brew install gh jq lefthook
```

### Initial Setup

1. **Clone the monorepo:**

   ```bash
   git clone git@github.com:tastybamboo/panda-monorepo.git panda
   cd panda
   ```

2. **Run the initialization script:**

   ```bash
   ./init
   ```

   This will automatically:
   - Discover and clone all panda-* gems you have access to
   - Set up git hooks
   - Install [`lefthook`](https://github.com/evilmartians/lefthook) where configured
   - Configure the `panda` CLI tool

3. **Add the Panda CLI to your PATH (optional):**

   ```bash
   echo 'export PATH="$HOME/Projects/panda/bin:$PATH"' >> ~/.zshrc
   source ~/.zshrc
   ```

## Development Workflow

### Working on Individual Gems

Each gem in the `gems/` directory is a separate Git repository with its own workflow:

```bash
# Navigate to the gem
cd gems/panda-core  # or panda-cms, panda-editor, etc.

# Create a feature branch
git checkout -b feature/your-feature-name

# Install dependencies
bundle install

# Make your changes
# ... edit files ...

# Run tests
bundle exec rspec

# Run code quality checks
bundle exec standardrb
bundle exec brakeman

# Commit your changes
git add .
git commit -m "feat: Add your feature description"

# Push to your fork (or branch)
git push origin feature/your-feature-name
```

### Creating Pull Requests

When you're ready to submit your changes:

1. **Ensure all tests pass:**

   ```bash
   bundle exec rspec
   ```

2. **Run linters and code quality tools:**

   ```bash
   bundle exec standardrb --fix
   bundle exec brakeman --quiet
   yamllint -c .yamllint .
   ```

3. **Push your branch and create a PR:**

   ```bash
   git push origin feature/your-feature-name
   gh pr create --title "Your PR Title" --body "Description of changes"
   ```

4. **Respond to review feedback** and make any requested changes.

## Contributing to Individual Gems

### panda-core

The core foundation gem providing shared functionality, authentication, and admin UI framework.

**Key areas:**

- Authentication system (OAuth integration)
- Admin UI components
- Base utilities and helpers
- CSS compilation system (Tailwind v4)

**Testing:**

```bash
cd gems/panda-core/spec/dummy
bundle exec rspec
```

### panda-cms

The content management system built on panda-core.

**Key areas:**

- Pages, posts, menus, and forms
- EditorJS integration
- Content rendering
- Admin CMS interface

**Testing:**

```bash
cd gems/panda-cms
bundle exec rspec
```

### panda-editor

The custom EditorJS implementation for content editing.

**Key areas:**

- EditorJS blocks
- JavaScript controllers
- Content rendering

### Other Gems

Each gem has its own `README.md` and development guidelines. Always refer to the gem-specific documentation for detailed contribution instructions.

## Contributing to Monorepo Infrastructure

The monorepo infrastructure includes:

- `init` script for bootstrapping
- `bin/` directory with CLI wrappers
- `scripts/` directory with the `panda` CLI implementation
- `.claude/` directory with AI development configurations
- `docs/` directory with documentation

### Making Changes to Infrastructure

1. **Test your changes locally:**

   ```bash
   # After modifying init or bin/ scripts
   ./init

   # Test the panda CLI
   panda --help
   panda gems list
   panda css compile
   ```

2. **Update documentation** if needed (README.md, docs/)

3. **Create a PR for the monorepo:**

   ```bash
   git checkout -b feature/infrastructure-improvement
   git add .
   git commit -m "feat: Improve initialization script"
   git push origin feature/infrastructure-improvement
   gh pr create
   ```

## Code Quality Standards

### Ruby Style

- Follow [StandardRB](https://github.com/standardrb/standard) style guide
- Run `bundle exec standardrb --fix` before committing
- No additional configuration needed (StandardRB is zero-config)

### Testing

- Write tests for all new features
- Maintain or improve code coverage
- Use RSpec for testing Rails engines
- System tests use Cuprite (headless Chrome)

### Git Commit Messages

Follow the [Conventional Commits](https://www.conventionalcommits.org/) specification:

- `feat:` for new features
- `fix:` for bug fixes
- `docs:` for documentation changes
- `refactor:` for code refactoring
- `test:` for adding or updating tests
- `chore:` for maintenance tasks

**Examples:**

```bash
git commit -m "feat: Add user authentication system"
git commit -m "fix: Resolve CSS compilation issue"
git commit -m "docs: Update installation instructions"
```

### YAML Files

- Always validate YAML files with `yamllint`:

  ```bash
  yamllint -c .yamllint .
  ```

## Pull Request Process

1. **Ensure your PR:**
   - Has a clear, descriptive title
   - Includes a summary of changes
   - References any related issues
   - Passes all CI checks
   - Has been tested locally

2. **PR Template:**

   ```markdown
   ## Summary
   Brief description of what this PR does

   ## Changes
   - List of changes made
   - Another change

   ## Testing
   - [ ] Tests pass locally
   - [ ] Linters pass
   - [ ] Manual testing completed

   ## Related Issues
   Fixes #123
   ```

3. **Review Process:**
   - PRs require at least one approval
   - Address review feedback promptly
   - Keep PRs focused and reasonably sized
   - CI must pass before merging

4. **After Approval:**
   - Maintainers will merge your PR
   - Your changes will be included in the next release

## Reporting Issues

### Bug Reports

When reporting bugs, please include:

- **Description:** Clear description of the issue
- **Steps to Reproduce:** Detailed steps to reproduce the bug
- **Expected Behavior:** What you expected to happen
- **Actual Behavior:** What actually happened
- **Environment:**
  - Ruby version
  - Rails version
  - Gem versions
  - Operating system
- **Screenshots/Logs:** If applicable

### Feature Requests

For feature requests, please include:

- **Description:** Clear description of the feature
- **Use Case:** Why this feature would be useful
- **Proposed Solution:** How you envision it working
- **Alternatives:** Any alternative solutions considered

### Security Issues

**Do not report security vulnerabilities publicly.** Instead, email security concerns to:

**james@otaina.co.uk**

## Development Tips

### Using Path References in Development

When developing multiple gems simultaneously:

```ruby
# In your test app's Gemfile (sites/your-app/)
gem "panda-core", path: "../../gems/panda-core"
gem "panda-cms", path: "../../gems/panda-cms"
```

**Important:** The monorepo's pre-push hook prevents accidentally pushing path references to GitHub.

### CSS Compilation

When modifying Tailwind classes:

```bash
# Compile CSS for all loaded modules
panda css compile

# Or from within a gem
bundle exec rake app:panda:compile_css
```

### Running Tests Across All Gems

```bash
# From the monorepo root
panda test all
```

## Getting Help

- **Documentation:** Check `docs/` directory for guides
- **Issues:** Browse existing [GitHub issues](https://github.com/tastybamboo/panda-monorepo/issues)
- **Discussions:** Join [GitHub Discussions](https://github.com/tastybamboo/panda-monorepo/discussions)
- **Email:** Contact james@otaina.co.uk

## License

By contributing to Panda Monorepo, you agree that your contributions will be licensed under the BSD 3-Clause License. See [LICENSE](LICENSE) for details.

Individual Panda gems are also BSD 3-Clause licensed. Check each gem's LICENSE file for specifics.

## Recognition

Contributors are recognized in:

- GitHub contribution graph
- Release notes and changelogs
- Project documentation

Thank you for contributing to Panda CMS! 🐼
