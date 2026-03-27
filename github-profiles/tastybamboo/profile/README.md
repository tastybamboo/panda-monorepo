![Panda CMS](https://github.com/pandacms/.github/blob/main/images/panda-transparent-small.png?raw=true)

# Panda CMS – Better websites, on Rails. 🐼

A modern, modular content management system built for Ruby on Rails. In production since March 2024.

**[Website](https://tastybamboo.net)** · **[Pro Features](https://tastybamboo.net/pro.html)** · **[Managed Hosting](https://tastybamboo.net/hosting.html)**

![Gem Version](https://img.shields.io/gem/v/panda-cms) ![Build Status](https://img.shields.io/github/actions/workflow/status/tastybamboo/panda-cms/ci.yml)
![GitHub Last Commit](https://img.shields.io/github/last-commit/tastybamboo/panda-cms) [![Ruby Code Style](https://img.shields.io/badge/code_style-standard-brightgreen.svg)](https://github.com/standardrb/standard)

## The Ecosystem

| Gem | Description | Status |
|-----|-------------|--------|
| [panda-core](https://github.com/tastybamboo/panda-core) | Shared foundation: OAuth auth, user management, configuration | Open Source |
| [panda-cms](https://github.com/tastybamboo/panda-cms) | Core CMS: pages, posts, templates, menus, forms, SEO | Open Source |
| [panda-editor](https://github.com/tastybamboo/panda-editor) | EditorJS-based block editor with footnotes, HTML/Markdown import | Open Source |
| [panda-cms-pro](https://github.com/tastybamboo/panda-cms-pro) | Collections, versioning, RBAC, REST API, content sync | Pro (£120/yr) |
| [panda-helpdesk](https://github.com/tastybamboo/panda-helpdesk) | Ticketing, departments, customer portal, SLA tracking | Pro (included) |
| [sent_emails](https://github.com/tastybamboo/sent_emails) | Email capture, delivery tracking, admin UI, resend | Open Source |

## Quick Start

```bash
bundle add panda-cms
rails generate panda:cms:install
rails db:migrate
rails server
```

Visit `/admin` to start managing content.

## Free vs Pro

The free tier includes pages, posts, templates, menus, forms, SEO, file management, and a modern block editor — features that competing CMS platforms charge hundreds of dollars for.

**Pro** adds collections, versioning, RBAC, REST API, content sync, website users, and a full helpdesk with ticketing — all for £120/yr per production site (inc. VAT).

## About

Panda CMS is lovingly maintained by [Otaina Limited](https://www.otaina.co.uk). Licensed under the [BSD-3-Clause License](https://opensource.org/licenses/bsd-3-clause).

Pandas fall over and eat a lot. So do we. 🐼
