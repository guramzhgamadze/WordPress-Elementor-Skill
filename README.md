# WordPress & Elementor Pro Development Standards

A centralized knowledge base and procedural guide for writing production-grade WordPress and Elementor Pro code. Whether you are scaffolding a new plugin, building advanced Elementor widgets, creating secure REST endpoints, or optimizing frontend performance — these documents give you the modern practices, strict standards, and compatibility rules to keep your codebase clean, secure, and maintainable.

The goal is not just to make things work, but to build robust, accessible, and performant projects that scale.

---

## Installation (Adding as a Claude AI Skill)

This repository is designed to be loaded as a custom Skill in Claude, so it can generate perfectly standardized, production-ready code for you on demand.

1. **Download** — Clone or download this repository as a `.zip` file.
2. **Upload** — In your Claude dashboard, go to **Customize → Skills → Upload a skill**, then drag and drop the `.zip` file.
3. **Done** — Claude will now automatically consult these docs whenever you ask a WordPress or Elementor question.

---

## What's Inside

The documentation is split into focused, single-topic files. `SKILL.md` is the router — Claude reads it first, then loads only the sub-files relevant to your task. This keeps context lean and responses fast.

| File | What it covers |
|---|---|
| [`SKILL.md`](./wp-elementor-skill/SKILL.md) | **Router & Golden Rules.** Default assumptions, architecture decision tree, output format, widget boilerplate map, current stack (WP 7.0 / Elementor 4.x / WC 10.8). Start here. |
| [`scaffolding.md`](./wp-elementor-skill/scaffolding.md) | **Plugin & Theme Structure.** Plugin bootstrap pattern, singleton class, child theme setup, Custom Post Types, taxonomy registration, AJAX handlers, V4 widget registration reminder. |
| [`php-standards.md`](./wp-elementor-skill/php-standards.md) | **PHP Best Practices.** Sanitization, escaping, nonce verification, `WP_Error`, transient caching, `declare(strict_types=1)`, WP 6.8+ bcrypt/BLAKE2b password hashing. |
| [`js-css-standards.md`](./wp-elementor-skill/js-css-standards.md) | **Frontend Standards.** ES6+ IIFE pattern, `elementor/frontend/init` event, WP 6.3+ defer/async enqueue API, `wp_add_inline_script`, BEM CSS, scoped design tokens, Elementor 4.x selector rules. |
| [`elementor-patterns.md`](./wp-elementor-skill/elementor-patterns.md) | **Elementor 4.x Integration.** V4 stable rules, full custom widget pattern (`has_widget_inner_wrapper`, `is_dynamic_content`, `content_template`), mandatory no-hardcoded-visuals rule, Dynamic Tags, Loop Grid query filter, Pro Form custom actions, Theme Builder custom conditions. |
| [`elementor-extending.md`](./wp-elementor-skill/elementor-extending.md) | **Elementor extension points beyond widgets.** Custom **form fields** (`Field_Base`), **theme locations** (`register_location` + `elementor_theme_do_location`), injecting controls into / filtering **native** widgets, Finder & context-menu items, a consolidated PHP + JS **hooks** quick-reference, and a **deprecations** table (`get_id_int`, `_register_controls`, `$depended_scripts`, schemes→globals, etc.). |
| [`woocommerce.md`](./wp-elementor-skill/woocommerce.md) | **WooCommerce Integration.** HPOS compatibility declaration, Order API rules (`wc_get_order`, `get_meta`, `update_meta_data`), Loop Grid for products, hook-based additions, template overrides. WC 10.7 sync-on-read change with source citation. |
| [`rest-api.md`](./wp-elementor-skill/rest-api.md) | **Custom REST Endpoints.** `register_rest_route`, JSON Schema draft-04, `rest_validate_request_arg`, `WP_Error` permission callbacks, REST nonce pattern + refresh, global post context setup. |
| [`offcanvas-ui.md`](./wp-elementor-skill/offcanvas-ui.md) | **Accessible Off-Canvas UI.** Full PHP/CSS/JS pattern — `inert` attribute set in HTML markup, CSS class toggle, `myplugin-offcanvas__backdrop--hidden` in HTML, ARIA `dialog` + `aria-modal`, focus trap, iOS Safari scroll lock, `prefers-reduced-motion` support. |
| [`performance.md`](./wp-elementor-skill/performance.md) | **Performance & Accessibility Checklists.** Frontend (LCP, lazy loading, WP 6.7+ auto-sizes, WP 6.9+ IE conditional comments, WP 6.8+ Speculative Loading), backend (WP_Query optimizations, WP 6.9+ salted cache keys, WP 7.0 RTC query scoping), WCAG 2.2 AA. |
| [`field-notes.md`](./wp-elementor-skill/field-notes.md) | **Hard-won production gotchas** distilled from shipping real plugins. Widget-lifecycle fatals (untyped overrides, `render()` re-runs, sticky `get_name()`), `content_template()` escaping & the wp.org editor-XSS rejection, CSS-in-Elementor footguns (custom-prop cascade, flex `min-width`, body-mounted UI), `wp_mail` inline-styles, `$_COOKIE` vs `$_REQUEST`, secrets at rest, embedding standalone apps inline, and wp.org review/packaging. |
| [`wp-org-guidelines.md`](./wp-elementor-skill/wp-org-guidelines.md) | **wordpress.org submission reference.** The 18 official Detailed Plugin Guidelines (paraphrased, actionable), Plugin Check 2.0.0 check-categories + how to run it (admin UI / WP-CLI, static vs runtime, auto-scan on updates), the AI-assisted-but-human-decided review process, and required headers/`readme.txt` fields. Sourced from the official guidelines, the Plugin Check page, and the review-team blog. |
| [`debugging.md`](./wp-elementor-skill/debugging.md) | **Debugging & static-analysis workflow.** PHPCS+WPCS (install, `phpcs.xml.dist` ruleset, security sniffs), PHPStan (WordPress, typed-override fatals), Plugin Check; `WP_DEBUG`/`WP_DEBUG_LOG`/`SCRIPT_DEBUG` + Query Monitor + WSOD diagnosis; Elementor Safe Mode / Regenerate Files & Data / element-cache freezes / editor-preview crashes; browser console + computed-styles; and a symptom→cause→where-to-look table. |
| [`wordpress-apis.md`](./wp-elementor-skill/wordpress-apis.md) | **Common WordPress APIs** the rest of the skill doesn't cover. Options API (WP 6.6+ boolean `autoload`), the review-safe **Settings API** admin-page pattern (nonce + single sanitize callback + escaped fields), Metadata API (`register_post_meta` + `show_in_rest`), Roles & Capabilities (gate by cap, not role), **WP-Cron** (+ the "not real cron" caveat, `DISABLE_WP_CRON` + system cron, Action Scheduler), and **Internationalization** (text-domain = slug, the WP 6.7 `init`-timing rule, JS i18n, `make-pot`). |
| [`svn/`](./wp-elementor-skill/svn/) | **Subversion (SVN) sub-bundle** (`svn/svn.md` + `svn/references/`) — a self-contained SVN reference distilled from the official SVN Book (1.7). Covers the daily work cycle, branching/tagging/merging, properties (`svn:ignore`/`svn:externals`/locking), and repository administration — plus a **WordPress.org plugin/theme SVN** worked example (`trunk`/`tags`/`assets`, tag a release, `Stable tag` match). This is how you *deploy* to the directory that `wp-org-guidelines.md` documents the *rules* for. |

---

## Core Philosophy

Six principles that override everything else:

1. **Native APIs first** — WordPress core hook before plugin logic; Elementor API before template override. Don't reinvent the wheel.
2. **Sanitize in, escape out** — Every input sanitized on receipt. Every output escaped at the point of rendering. No exceptions, ever.
3. **Prefix everything** — All functions, classes, constants, hooks, and CSS classes get a project-specific prefix. The WordPress ecosystem is crowded.
4. **State your placement** — Every code response declares exactly which file it belongs in and why. Plan before you code.
5. **Never hardcode visual settings in widgets** — Every visual property (colors, fonts, sizes, spacing, backgrounds, borders, shadows) must be an Elementor control. See `SKILL.md §5`.
6. **Name for the directory from day one** — Slug must not start with a trademark you don't own (`"X for WooCommerce"`, not `"WooCommerce X"`); text domain equals the slug; the wp.org slug and a widget's `get_name()` are permanent. Decide the public name, slug, text domain, and code prefix together, once. See `wp-org-guidelines.md`.

---

## Environment Assumptions

All patterns in this repository target a modern production stack. Unless a specific guide notes otherwise:

| Component | Version | Notes |
|---|---|---|
| **WordPress** | **7.0+** | "Armstrong", released May 20, 2026. Minimum PHP raised to 7.4 (7.2/7.3 dropped). RTC stored in a dedicated core table. |
| **PHP** | 8.3+ recommended | 7.4 is the WP 7.0 minimum. PHP 8.4 and 8.5 carry a "beta support" label — possible deprecation notices. |
| **Elementor (free + Pro)** | **4.2.0** | Current as of June 5, 2026. 4.0.0 (Mar 30, 2026) made the Atomic Editor stable and default for new installs. **V3 `Widget_Base` remains fully supported — all skill code targets V3.** |
| **WooCommerce** | **10.8+** | HPOS enabled by default for new stores since 8.2. WC 10.7 (April 14, 2026) disabled HPOS "sync on read" by default — see `woocommerce.md`. |

---

## How to Use Day-to-Day

Once installed as a skill in Claude:

1. **Ask naturally** — "Build me a custom Elementor widget that displays WooCommerce products with a custom query" or "What's the correct hook to sanitize a URL input in WordPress 6.9?"
2. **Claude reads the docs** — The router in `SKILL.md` points to the relevant sub-file. Claude reads only what it needs for your task.
3. **Get production-ready code** — Every response follows the mandatory output format: placement declared, dependencies listed, approach explained, complete code block, integration notes included.

---

## Keeping This Up to Date

This skill is audited against official live sources after every significant release.
The full round-by-round history lives in [`CHANGELOG.md`](./CHANGELOG.md).

**Latest — Round 21 (July 11, 2026):** Post-release currency sweep. WordPress **7.0 "Armstrong"**
shipped May 20, 2026; Elementor is at **4.2.0** (June 5, 2026); WooCommerce **10.8** (May 26, 2026).
This round removed the obsolete WP 7.0 beta/RC/delay narration from the router, settled the RTC
storage and WooCommerce 10.7 sections into shipped fact, consolidated two pairs of duplicate
widget-template files into one each, and moved release history into `CHANGELOG.md` so version
facts live in a single place.

**Next scheduled audit:** after the next WordPress release (7.0.x / 7.1) or a major Elementor release.

---

## Official Sources

All facts in this skill are verified against official documentation only:

- [wordpress.org/news](https://wordpress.org/news/) — Release announcements
- [make.wordpress.org/core](https://make.wordpress.org/core/) — Core developer notes
- [developer.wordpress.org](https://developer.wordpress.org/) — WordPress developer reference
- [developers.elementor.com](https://developers.elementor.com/) — Elementor developer docs
- [elementor.com/pro/changelog](https://elementor.com/pro/changelog/) — Elementor Pro changelog
- [github.com/elementor/elementor](https://github.com/elementor/elementor) — Elementor free changelog
- [wordpress.org/plugins/elementor](https://wordpress.org/plugins/elementor/) — Elementor on WordPress.org
- [developer.woocommerce.com](https://developer.woocommerce.com/) — WooCommerce developer blog

---

## License

GPL-2.0-or-later — consistent with the WordPress ecosystem.
