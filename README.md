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

### Core Files (9)

| File | What it covers |
|---|---|
| [`SKILL.md`](./SKILL.md) | **Router & Golden Rules.** Default assumptions, architecture decision tree, mandatory output format, widget boilerplate map, quick-reference index. Start here. |
| [`scaffolding.md`](./scaffolding.md) | **Plugin & Theme Structure.** Plugin bootstrap pattern, singleton class, child theme setup, Custom Post Types, taxonomy registration, AJAX handlers, widget registration reminder (`has_widget_inner_wrapper` + `is_dynamic_content`). |
| [`php-standards.md`](./php-standards.md) | **PHP Best Practices.** Sanitization, escaping, nonce verification, `WP_Error`, transient caching, `declare(strict_types=1)`, WP 6.8+ bcrypt/BLAKE2b password hashing, full `wp_set_password()` hook notes. |
| [`js-css-standards.md`](./js-css-standards.md) | **Frontend Standards.** ES6+ IIFE pattern, `elementor/frontend/init` event, scope guard, WP 6.3+ defer/async enqueue API, `wp_add_inline_script`, BEM CSS, scoped design tokens. |
| [`elementor-patterns.md`](./elementor-patterns.md) | **Elementor Integration.** V4 compatibility rules, full custom widget pattern, Dynamic Tag category reference table, Loop Grid query filter, Pro Form custom actions, Theme Builder custom conditions. |
| [`woocommerce.md`](./woocommerce.md) | **WooCommerce Integration.** HPOS compatibility declaration, Order API rules, Loop Grid for products, hook-based additions, template overrides. WC 10.7 sync-on-read change documented. |
| [`rest-api.md`](./rest-api.md) | **Custom REST Endpoints.** `register_rest_route`, JSON Schema draft-04 (explained), `rest_validate_request_arg`, `WP_Error` permission callbacks, REST nonce pattern + refresh, global post context setup. |
| [`offcanvas-ui.md`](./offcanvas-ui.md) | **Accessible Off-Canvas UI.** Full PHP/CSS/JS pattern — `inert` in HTML markup (not only JS), CSS class toggle, ARIA `dialog` + `aria-modal`, focus trap, iOS Safari scroll lock, `prefers-reduced-motion`. |
| [`performance.md`](./performance.md) | **Performance & Accessibility Checklists.** Frontend (LCP, lazy loading, WP 6.7+ auto-sizes, WP 6.9+ IE conditional comments removal, WP 6.8+ Speculative Loading), backend (WP_Query optimizations, WP 6.9+ salted cache keys, WP 7.0+ `wp_sync_storage` exclusion), WCAG 2.2 AA. |

### Widget Boilerplate Files (35)

Each file provides `register_controls()` and `render()` patterns verified against the corresponding Elementor native widget source code. Load the relevant boilerplate alongside `elementor-patterns.md` for any widget build — it ensures correct control IDs, defaults, selectors, and render logic.

| File | Widget type |
|---|---|
| [`widget-button.md`](./widget-button.md) | Button / CTA with type, icon, and link |
| [`widget-container.md`](./widget-container.md) | Container / layout wrapper / section / card shell |
| [`widget-image.md`](./widget-image.md) | Image with caption, link, lightbox — `Group_Control_Image_Size` |
| [`widget-heading.md`](./widget-heading.md) | Heading / title / HTML tag selector |
| [`widget-text-editor.md`](./widget-text-editor.md) | Rich text / WYSIWYG body content |
| [`widget-video.md`](./widget-video.md) | Video embed (YouTube, Vimeo, self-hosted) |
| [`widget-elementor-template.md`](./widget-elementor-template.md) | Render a saved Elementor template — concise |
| [`widget-template-elementor.md`](./widget-template-elementor.md) | Render a saved Elementor template — extended (SELECT2, Dynamic Tags, CSS timing) |
| [`widget-php-template.md`](./widget-php-template.md) | PHP template file widget — concise |
| [`widget-template-php.md`](./widget-template-php.md) | PHP template file widget — extended (Strategies A/B/C, ob_start, path traversal guard) |
| [`widget-divider.md`](./widget-divider.md) | Divider / horizontal rule with optional text or icon |
| [`widget-spacer.md`](./widget-spacer.md) | Spacer / vertical gap (single responsive SLIDER) |
| [`widget-icon.md`](./widget-icon.md) | Single standalone icon with optional link |
| [`widget-icon-box.md`](./widget-icon-box.md) | Icon + title + description box |
| [`widget-image-box.md`](./widget-image-box.md) | Image + title + description box |
| [`widget-image-gallery.md`](./widget-image-gallery.md) | Image grid / gallery — GALLERY control |
| [`widget-image-carousel.md`](./widget-image-carousel.md) | Image slider / carousel — Swiper (bundled with Elementor) |
| [`widget-icon-list.md`](./widget-icon-list.md) | Bullet list with icons per item — REPEATER |
| [`widget-counter.md`](./widget-counter.md) | Animated number counter with prefix/suffix |
| [`widget-progress.md`](./widget-progress.md) | Percentage progress bar |
| [`widget-testimonial.md`](./widget-testimonial.md) | Customer quote / testimonial |
| [`widget-tabs.md`](./widget-tabs.md) | Tabbed content panels — horizontal/vertical |
| [`widget-accordion.md`](./widget-accordion.md) | Accordion — one panel open at a time |
| [`widget-toggle.md`](./widget-toggle.md) | Toggle — multiple panels open simultaneously |
| [`widget-social-icons.md`](./widget-social-icons.md) | Social media icon links row — REPEATER |
| [`widget-alert.md`](./widget-alert.md) | Colored alert / notice box with optional dismiss |
| [`widget-audio.md`](./widget-audio.md) | Audio player (SoundCloud / self-hosted) |
| [`widget-shortcode.md`](./widget-shortcode.md) | WordPress shortcode output via `do_shortcode()` |
| [`widget-html.md`](./widget-html.md) | Raw custom HTML / JS / CSS embed |
| [`widget-menu-anchor.md`](./widget-menu-anchor.md) | Named anchor for in-page navigation |
| [`widget-sidebar.md`](./widget-sidebar.md) | WordPress registered sidebar output |
| [`widget-read-more.md`](./widget-read-more.md) | WordPress `<!--more-->` read-more tag |
| [`widget-google-maps.md`](./widget-google-maps.md) | Google Maps embed |
| [`widget-star-rating.md`](./widget-star-rating.md) | Decorative star rating display |
| [`widget-rating.md`](./widget-rating.md) | Schema-ready structured rating (fractional, schema markup) |

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
| **WordPress** | 6.9.4+ | WP 7.0 scheduled April 9, 2026. RC1 released March 24, 2026. PHP 7.2/7.3 dropped in 7.0; new minimum PHP 7.4. |
| **PHP** | 8.3+ | Officially recommended by wordpress.org. PHP 8.4 and 8.5 carry a "beta support" label. |
| **Elementor (free)** | 3.35.7+ | V3 `Widget_Base` API. V4 production-ready (Beta) in 3.35 but PHP extension API not yet public — all patterns use V3. |
| **Elementor Pro** | 3.35.1+ | Independent version number. Always check both free and Pro when diagnosing compatibility issues. |
| **WooCommerce** | 8.2+ | HPOS enabled by default for new stores. WC 10.7 (scheduled April 14, 2026) disables HPOS "sync on read" by default — see `woocommerce.md`. |

---

## How to Use Day-to-Day

Once installed as a skill in Claude:

1. **Ask naturally** — "Build me a custom Elementor widget that displays WooCommerce products with a custom query" or "What's the correct hook to sanitize a URL input in WordPress 6.9?"
2. **Claude reads the router** — `SKILL.md` points to the relevant sub-file(s). For widget tasks it loads the matching `widget-*.md` boilerplate alongside `elementor-patterns.md`.
3. **Get production-ready code** — Every response follows the mandatory output format: placement declared, dependencies listed, approach explained, complete code block, integration notes included.

---

## Keeping This Up to Date

This skill is actively maintained and audited against official sources after every significant release. Audit history:

- **Rounds 1–13** — Original monolithic `SKILL.md` (56 bugs fixed)
- **Round 14** — Split into 9-file core bundle; added 35 widget boilerplates; Elementor free updated 3.35.6 → 3.35.7
- **Round 15** — WordPress 6.9.3 → 6.9.4; WP 7.0 Beta 5 confirmed released; `wp_kces_post` typo fixed → `wp_kses_post`
- **Round 16** — Full verification pass; all facts confirmed clean against live sources (March 16, 2026)
- **Round 17** — Comprehensive audit (March 25, 2026): **Bug fix** — added `inert` attribute directly to off-canvas HTML panel (keyboard focus gap before JS load); **Bug fix** — added JSON Schema draft-04 explanatory comment to `rest-api.md`; synced SKILL.md with Beta 6 details, RC1 delay, `WP_ALLOW_COLLABORATION` constant, Client-side Media Processing revert; updated README to document all 44 files across both sections

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
