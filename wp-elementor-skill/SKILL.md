---
name: wordpress-elementor-dev
description: >
  Expert WordPress and Elementor Pro development skill. Use this whenever the user
  asks about WordPress theme development, Elementor Pro customization, plugin architecture,
  custom post types, hooks/filters, Loop Grids, Dynamic Tags, Theme Builder templates,
  off-canvas UIs, custom CSS/JS injection, performance optimization, or any PHP/JS/CSS
  code targeting a WordPress or Elementor environment. Trigger even for general questions
  like "how do I add a custom field in Elementor", "what hook should I use for X in
  WordPress", "how do I override a WooCommerce template", or "how do I register a
  custom widget" — do not wait for the user to explicitly say "use the skill."
  Also trigger for ACF integration, REST API endpoints, child theme setup, WooCommerce
  customization, WPML/multilingual setups, and any debugging of WordPress/Elementor issues.
---

# WordPress & Elementor Pro — Skill Router

A complete procedural guide for producing production-grade WordPress and Elementor Pro
code. This file is the **router** — read it first, then load the relevant sub-file(s)
for the task at hand.

---

## Sub-file Map — Read the Right File for Each Task

| Task type | Read this file |
|---|---|
| Plugin scaffold, child theme, code placement, CPT, AJAX handler | **scaffolding.md** |
| PHP standards, sanitization, escaping, nonces, WP_Error, transients, password hashing | **php-standards.md** |
| JavaScript standards, enqueue API, defer/async, wp_add_inline_script | **js-css-standards.md** |
| CSS standards, BEM, design tokens, Elementor CSS selectors | **js-css-standards.md** |
| Elementor custom widget, Dynamic Tags, Loop Grid, Form actions, Theme Builder conditions | **elementor-patterns.md** |
| WooCommerce HPOS, order API, template overrides, Loop Grid for products | **woocommerce.md** |
| REST API endpoints, schema, permission callbacks | **rest-api.md** |
| Off-canvas UI, off-canvas accessibility, focus trap | **offcanvas-ui.md** |
| Performance checklists (frontend + backend), speculative loading, IE conditional comments | **performance.md** |
| Accessibility checklist, WCAG 2.2 AA, ARIA patterns | **performance.md** |

**Always read the relevant sub-file before writing code.** For tasks that span multiple
areas (e.g. a WooCommerce widget with custom REST endpoint), read all relevant sub-files.

---

## 0. Golden Rules (Never Violate)

These override everything in all sub-files:

1. **Native APIs first** — WordPress core hook before plugin; Elementor API before template override.
2. **Sanitize in, escape out** — Every input sanitized. Every output escaped. No exceptions.
3. **Prefix everything** — All functions, classes, constants, hooks, and CSS classes use a project-specific prefix.
4. **State your placement** — Every code response must declare exactly where the code lives.
5. **No over-clarifying** — Only ask a clarification question if the missing info would materially change the code output. Otherwise, state your assumption and proceed.

---

## 1. Default Assumptions

Quickly assess — **only ask if the answer would change the code**:

| Info needed | Ask only if... |
|---|---|
| Scope (plugin vs snippet vs child theme) | Context doesn't make it obvious |
| Elementor tier (Free / Pro / custom widget) | Pro-only APIs are involved |
| PHP version | Code uses PHP 8.3+ features like typed class constants, or 8.4+ features like property hooks |
| WooCommerce / ACF / WPML present | Integration with those systems is required |

**Default assumptions when not stated:** WordPress 6.9.4+ (WP 7.0 scheduled April 9, 2026),
PHP 8.3+ (officially recommended by wordpress.org/about/requirements/; 8.4 and 8.5 = beta
support label; 8.2 is fully compatible but no longer the recommended default),
Elementor (free) 3.35.7+ / Elementor Pro 3.35.1+ (V3 Widget_Base API).
**Note:** Elementor core and Elementor Pro are separate plugins with independent version numbers
(as of March 2026: core = 3.35.7, Pro = 3.35.1). Always check both when diagnosing
compatibility issues. No multisite, standard single-site install assumed.

> 🔜 **WordPress 7.0 (April 9, 2026):** PHP 7.2 and 7.3 will no longer be supported. The new
> minimum PHP version is 7.4.0. Sites still on PHP 7.2/7.3 will remain pinned to WP 6.9 and
> will not receive the 7.0 update. The minimum **recommended** version remains PHP 8.3.
> **Database:** No new minimum is enforced in WP 7.0 — the DB minimum was last bumped in
> WP 6.5 (MySQL 5.5.5+). The official `wordpress.org/about/requirements/` page now lists
> **MariaDB 10.6+ or MySQL 8.0+** as the primary recommended versions, and PHP 8.3+ as
> the recommended PHP version.
> Source: make.wordpress.org/core/2026/01/09/dropping-support-for-php-7-2-and-7-3/,
> wordpress.org/about/requirements/

> 🤖 **WordPress 7.0 — New Developer APIs (WP AI Client + Connectors UI + Abilities API):**
> All opt-in, no breaking changes. See quick summary below — these do not affect most
> plugin/Elementor work in WP 7.0.
>
> - **WP AI Client:** `wp_ai_client_prompt($prompt)->generate_text()` — provider-agnostic
>   PHP + JS AI API. Use `function_exists('wp_ai_client_prompt')` to guard.
> - **Connectors UI:** Settings → Connectors admin page for managing AI provider credentials.
>   Hook: `connections-wp-admin-init`. APIs still experimental until WP 7.0 stable ships.
> - **Abilities API:** PHP side in WP 6.9 (`wp_register_ability()`); JS counterpart in WP 7.0.
>   `'meta' => ['show_in_rest' => true]` to expose via REST.
> - **Iframed Editor (PUNTED to WP 7.1):** No action required for WP 7.0. Prepare for 7.1
>   with `"apiVersion": 3` in block.json.
> - **Real-Time Collaboration (RTC):** WP 7.0 introduces simultaneous multi-author block
>   editing. Technically: HTTP polling sync provider (not WebRTC), CRDT update data stored
>   persistently in `wp_post_meta` on a new internal post type **`wp_sync_storage`**. Updates
>   are batched and periodically compacted. Default enable decision finalised around RC2.
>   **Plugin impact:** If your plugin queries `wp_post_meta` by post type or iterates all
>   posts/meta, add `'post_type__not_in' => ['wp_sync_storage']` (or equivalent exclusion)
>   to avoid unexpected hits on internal sync data. Hosts can swap the transport provider
>   via a `wp-config` constant.
>   Source: developer.wordpress.org/news/2026/03/whats-new-for-developers-march-2026/
> - **WP 7.0 Beta cycle:** Beta 4 released **March 10, 2026** (emergency security fast-follow,
>   same day as WordPress 6.9.2 and 6.9.3). **WordPress 6.9.4** was then released March 11, 2026
>   after the Security Team found not all fixes in 6.9.2 were fully applied — 6.9.4 is the
>   current stable release. Beta 5 **released** March 12, 2026. RC1 scheduled March 19, 2026.
>   Final release: April 9, 2026.
>   Source: wordpress.org/news/2026/03/wordpress-6-9-3-and-7-0-beta-4/,
>   wordpress.org/news/2026/03/wordpress-7-0-beta-5/,
>   wordpress.org/news/2026/03/wordpress-6-9-4-release/,
>   make.wordpress.org/core/2026/02/12/wordpress-7-0-release-party-schedule/

> 📌 **PHP support labels (WP 6.9):**
> - PHP 8.0–8.3 = fully compatible. PHP 7.4 = fully compatible.
> - **PHP 8.3 = officially recommended** (raised from beta in July 2025).
> - PHP 8.4 = beta support (WP 6.7+). PHP 8.5 = beta support (WP 6.9+).
> - "Beta support" = actively working toward full compatibility; possible deprecation notices.
> Source: make.wordpress.org/core/handbook/references/php-compatibility-and-wordpress-versions/

> ⚠️ **Elementor V4 note:** V4 production-ready (Beta status) in Elementor 3.35 (February 2, 2026).
> New V4 features in 3.35 include **Components** (reusable layout blocks with global sync +
> per-instance content overrides) and **Inline Editing** (edit Atomic Heading/Paragraph text
> directly on canvas). V4 PHP extension APIs (custom Atomic Elements / Components) are
> **not yet publicly documented** — do not use in third-party plugins. All skill code targets
> V3 `Widget_Base` API. V3 and V4 coexist; official 4.0 release is upcoming (V4 will become
> default for new sites on 4.0 — no forced migration for existing V3 widgets).

---

## 2. Architecture Decision Tree

Run through this mentally before writing a single line:

```
Does a WordPress core hook (add_action/add_filter) solve it?
  YES → Use the hook. No plugin needed. Place in child theme functions.php
        or Elementor Custom Code.
  NO  → Does Elementor's PHP/JS API solve it?
          YES → Extend via Elementor hooks, Dynamic Tags, or Widget_Base.
                Register via elementor/widgets/register or elementor/dynamic_tags/register.
          NO  → Is this logic reusable across themes or sites?
                  YES → Scaffold a dedicated plugin (see scaffolding.md).
                  NO  → Child theme functions.php or Elementor Custom Code block.

Is a WooCommerce override needed?
  → Use Elementor Loop Grid + custom query filter BEFORE touching template files.
  → Only override woocommerce/ templates as an absolute last resort.

Is this a REST API endpoint?
  → Always register via register_rest_route() inside a plugin, never in functions.php.
```

---

## 15. Mandatory Output Format

**Every single code response must follow this structure — no exceptions:**

```
📍 PLACEMENT
Exact file path or Elementor hook location.
e.g. /wp-content/plugins/myplugin/includes/class-myplugin-hooks.php
     Elementor → Site Settings → Custom Code → wp_footer

⚙️ REQUIRES
WordPress X.X+ | PHP X.X+ | Elementor Pro X.X+ | ACF X.X+ | WooCommerce X.X+
(list only what the code actually depends on)

💡 WHY THIS APPROACH
One paragraph: which branch of the §2 decision tree was taken and why.

📋 CODE
Complete, commented, deployment-ready code block — no truncation, no omissions.

🔧 INTEGRATION NOTES (include when relevant)
Any manual steps required: flush rewrite rules, set Query ID in Elementor editor,
activate plugin, clear Elementor cache, etc.
```

---

## 16. Quick Reference — Pattern Index

| Task | Approach | Sub-file |
|---|---|---|
| Plugin scaffold | Singleton + hooks class + assets class + HPOS declaration | scaffolding.md |
| Child theme setup | `wp_enqueue_style` parent + child in functions.php | scaffolding.md |
| Custom Post Type + Taxonomy | `register_post_type()` + `register_taxonomy()` in plugin | scaffolding.md |
| Secure AJAX handler | `wp_ajax_` hooks + nonce verify + `wp_send_json_*` | scaffolding.md |
| PHP sanitization / escaping | `wp_unslash()` + `sanitize_*` + `esc_*` patterns | php-standards.md |
| Transient caching | `get_transient` / `set_transient` | php-standards.md |
| External API call + WP_Error | `wp_remote_get()` + `WP_Error` pattern | php-standards.md |
| WP 6.8 password hashing | `wp_check_password()` + `wp_password_needs_rehash()` | php-standards.md |
| WP 6.8 app password / key hashing | `wp_fast_hash()` + `wp_verify_fast_hash()` (BLAKE2b) | php-standards.md |
| JS standards + enqueue defer/async | IIFE + WP 6.3+ enqueue API | js-css-standards.md |
| PHP → JS data passing | `wp_add_inline_script()` with `wp_json_encode()` | js-css-standards.md |
| CSS BEM + design tokens | Scoped tokens, 8pt spacing, fluid type | js-css-standards.md |
| Custom Elementor widget | `Widget_Base` + all required methods | elementor-patterns.md |
| Elementor Dynamic Tag | `Tag` class + `elementor/dynamic_tags/register` | elementor-patterns.md |
| Elementor Loop Grid query | `elementor/query/` filter | elementor-patterns.md |
| ACF field in Elementor | Dynamic Tag extending `\Elementor\Core\DynamicTags\Tag` | elementor-patterns.md |
| Elementor Pro Form action | `Action_Base` + field iteration | elementor-patterns.md |
| Theme Builder custom condition | `Condition_Base` + `elementor/theme/register_conditions` | elementor-patterns.md |
| WooCommerce HPOS compatibility | `FeaturesUtil::declare_compatibility()` + `wc_get_order()` | woocommerce.md |
| WooCommerce loop | Loop Grid + `elementor/query/` filter | woocommerce.md |
| Custom REST endpoint | `register_rest_route()` + schema callback in plugin | rest-api.md |
| Off-canvas filter panel | BEM CSS + ARIA JS + Custom Code block | offcanvas-ui.md |
| Performance checklist | Frontend + backend checklists | performance.md |
| Accessibility checklist | WCAG 2.2 AA patterns | performance.md |
| Elementor dependency check | `Requires Plugins: elementor` header + `did_action` fallback | scaffolding.md |
