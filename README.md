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
| [`SKILL.md`](./SKILL.md) | **Router & Golden Rules.** Default assumptions, architecture decision tree, output format, quick-reference index. Start here. |
| [`scaffolding.md`](./scaffolding.md) | **Plugin & Theme Structure.** Plugin bootstrap pattern, singleton class, child theme setup, Custom Post Types, taxonomy registration, AJAX handlers. |
| [`php-standards.md`](./php-standards.md) | **PHP Best Practices.** Sanitization, escaping, nonce verification, `WP_Error`, transient caching, `declare(strict_types=1)`, WP 6.8+ bcrypt/BLAKE2b password hashing. |
| [`js-css-standards.md`](./js-css-standards.md) | **Frontend Standards.** ES6+ IIFE pattern, `elementor/frontend/init` event, WP 6.3+ defer/async enqueue API, `wp_add_inline_script`, BEM CSS, scoped design tokens. |
| [`elementor-patterns.md`](./elementor-patterns.md) | **Elementor Pro Integration.** V4 compatibility rules, full custom widget pattern (`has_widget_inner_wrapper`, `is_dynamic_content`, `content_template`), Dynamic Tags, Loop Grid query filter, Pro Form custom actions, Theme Builder custom conditions. |
| [`woocommerce.md`](./woocommerce.md) | **WooCommerce Integration.** HPOS compatibility declaration, Order API rules (`wc_get_order`, `get_meta`, `update_meta_data`), Loop Grid for products, hook-based additions, template overrides. Includes WC 10.7 sync-on-read change. |
| [`rest-api.md`](./rest-api.md) | **Custom REST Endpoints.** `register_rest_route`, schema callbacks, `rest_validate_request_arg`, proper permission callbacks with `WP_Error`, global post context setup. |
| [`offcanvas-ui.md`](./offcanvas-ui.md) | **Accessible Off-Canvas UI.** Full PHP/CSS/JS pattern — CSS class toggle, `inert` attribute, ARIA `dialog` + `aria-modal`, focus trap, iOS Safari scroll lock, `prefers-reduced-motion` support. |
| [`performance.md`](./performance.md) | **Performance & Accessibility Checklists.** Frontend checklist (LCP, lazy loading, WP 6.7+ auto-sizes, WP 6.9+ IE conditional comments, WP 6.8+ Speculative Loading), backend checklist (WP_Query optimizations, WP 6.9+ salted cache keys), WCAG 2.2 AA accessibility checklist. |

---

## Core Philosophy

Four principles that override everything else:

1. **Native APIs first** — WordPress core hook before plugin logic; Elementor API before template override. Don't reinvent the wheel.
2. **Sanitize in, escape out** — Every input sanitized on receipt. Every output escaped at the point of rendering. No exceptions, ever.
3. **Prefix everything** — All functions, classes, constants, hooks, and CSS classes get a project-specific prefix. The WordPress ecosystem is crowded.
4. **State your placement** — Every code response declares exactly which file it belongs in and why. Plan before you code.

---

## Environment Assumptions

All patterns in this repository target a modern production stack. Unless a specific guide notes otherwise:

| Component | Version | Notes |
|---|---|---|
| **WordPress** | 6.9.4+ | WP 7.0 scheduled April 9, 2026. PHP 7.2/7.3 support dropped in 7.0; new minimum is PHP 7.4. |
| **PHP** | 8.3+ | Officially recommended by wordpress.org. PHP 8.4 and 8.5 carry a "beta support" label — possible deprecation notices. |
| **Elementor (free)** | 3.35.7+ | V3 `Widget_Base` API. V4 production-ready in 3.35 (February 2026) but its PHP extension API is not yet publicly documented — all patterns here use V3. |
| **Elementor Pro** | 3.35.1+ | Independent version number from Elementor free. Always check both when diagnosing compatibility issues. |
| **WooCommerce** | 8.2+ | HPOS enabled by default for new stores. WC 10.7 (April 14, 2026) disables HPOS "sync on read" by default — see `woocommerce.md`. |

---

## How to Use Day-to-Day

Once installed as a skill in Claude:

1. **Ask naturally** — "Build me a custom Elementor widget that displays WooCommerce products with a custom query" or "What's the correct hook to sanitize a URL input in WordPress 6.9?"
2. **Claude reads the docs** — The router in `SKILL.md` points to the relevant sub-file. Claude reads only what it needs for your task.
3. **Get production-ready code** — Every response follows the mandatory output format: placement declared, dependencies listed, approach explained, complete code block, integration notes included.

---

## Keeping This Up to Date

This skill is actively maintained and audited against official sources after every significant release. Audit history:

- **Rounds 1–13** — Original monolithic `SKILL.md` (56 bugs fixed)
- **Round 14** — Split into 9-file bundle; Elementor free updated 3.35.6 → 3.35.7
- **Round 15** — WordPress 6.9.3 → 6.9.4; WP 7.0 Beta 5 confirmed released; `wp_kces_post` typo fixed → `wp_kses_post`
- **Round 16** — Full verification pass; all facts confirmed clean against live sources (March 16, 2026)

**Next scheduled audit:** April 9–14, 2026 — after WordPress 7.0 stable and WooCommerce 10.7 both ship.

---

## Official Sources

All facts in this skill are verified against official documentation only:

- [wordpress.org/news](https://wordpress.org/news/) — Release announcements
- [make.wordpress.org/core](https://make.wordpress.org/core/) — Core developer notes
- [developer.wordpress.org](https://developer.wordpress.org/) — WordPress developer reference
- [developers.elementor.com](https://developers.elementor.com/) — Elementor developer docs
- [elementor.com/pro/changelog](https://elementor.com/pro/changelog/) — Elementor Pro changelog
- [github.com/elementor/elementor](https://github.com/elementor/elementor) — Elementor free changelog
- [developer.woocommerce.com](https://developer.woocommerce.com/) — WooCommerce developer blog

---

## License

GPL-2.0-or-later — consistent with the WordPress ecosystem.
