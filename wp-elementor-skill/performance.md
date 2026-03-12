# Performance & Accessibility Checklists

## Frontend Performance

- [ ] Scripts use `strategy: defer` via WP 6.3+ API — never block parsing.
      **Exception: Elementor widget scripts must NOT use `defer`** — use `[ 'in_footer' => true ]`
      with `elementor-frontend` declared as a dependency instead (see elementor-patterns.md Step 2)
- [ ] Critical above-fold CSS inlined; rest loaded asynchronously
- [ ] Version strings on all enqueued assets (`MYPLUGIN_VERSION`) for cache busting
- [ ] LCP image has `fetchpriority="high"` and is NOT lazy-loaded
- [ ] All below-fold images have `loading="lazy"` + explicit `width`/`height` (prevents CLS).
      **WP 6.7+:** `auto-sizes` is prepended to the `sizes` attribute of lazy-loaded images by
      default — the browser uses the rendered layout width to pick the correct srcset source.
      Can be disabled via the `wp_img_tag_add_auto_sizes` filter (added in 6.7.1).
- [ ] WebP format served; AVIF where browser support allows
- [ ] `wp_add_inline_script()` used for PHP→JS data passing (not `wp_localize_script()`);
      `wp_json_encode()` return value checked before passing — a `false` return outputs broken JS
- [ ] **WP 6.9+ IE conditional comments removed** — `wp_script_add_data( 'handle', 'conditional', 'IE' )`
      and `wp_style_add_data( 'handle', 'conditional', 'IE' )` now trigger a deprecation notice
      if `WP_DEBUG` is true, and **the asset is entirely ignored** — it will not appear on the
      page at all. Remove any IE-conditional asset registration from plugins and child themes.
      Note: bundled files that were only ever loaded conditionally for IE remain in core as
      **empty stub files** — do not depend on them for any actual styles or scripts.
      Source: make.wordpress.org/core/2025/11/19/legacy-internet-explorer-code-removed/
- [ ] **WP 6.8+ Speculative Loading** uses `prefetch` with `conservative` eagerness by default
      (Speculation Rules API) — only for logged-out users and only when pretty permalinks are
      enabled. The `wp_speculation_rules_configuration` filter receives
      `array( 'mode' => 'auto', 'eagerness' => 'auto' )` by default (WordPress Core resolves
      `'auto'` to `prefetch` + `conservative`; do not test for `'conservative'` directly — check
      for `'auto'` or the resolved output instead).
      If your plugin uses **one-time nonces**, **session state writes**, or **cart/checkout
      side-effects** that trigger on page load, exclude those URLs via the
      `wp_speculation_rules_href_exclude_paths` filter.
      Add the `no-prefetch` CSS class to a link *or its container element* to exclude that link
      (and all descendant links) from Core's prefetch speculation rule — WP Core's selector is
      `.no-prefetch, .no-prefetch a`; use `no-prerender` for the prerender rule
      (selector: `.no-prerender, .no-prerender a`).
      Test coverage with DevTools → Application → Speculation Rules.

---

## Backend / Database Performance

- [ ] `no_found_rows => true` on WP_Query when pagination is absent
- [ ] `update_post_meta_cache => false`, `update_term_meta_cache => false` when metadata not needed
- [ ] Expensive queries or remote calls wrapped in `get_transient` / `set_transient`
- [ ] Transient keys are ≤ 172 characters — dynamic keys use `md5()` to prevent silent cache misses
- [ ] No `SELECT *` — use `$wpdb->get_results` with explicit column names
- [ ] No queries inside loops — pre-fetch with a single WP_Query, then loop results
- [ ] **WP 6.9+ WP_Query cache key change** — WordPress 6.9 changed how cache keys are generated
      for queries performed through `WP_Query` (and other query classes: `WP_Term_Query`,
      `WP_Comment_Query`, `WP_User_Query`, etc.). Cache keys now store alongside the
      last-changed timestamp and validate it on retrieval, instead of embedding it in the key.
      This is compatible with all existing persistent object cache drop-ins.
      However: if your plugin directly reads or writes to cache groups such as `post-queries`,
      `term-queries`, `comment-queries`, `user-queries`, or checks specific keys, review against
      the four new functions introduced in WP 6.9: `wp_cache_get_salted()`,
      `wp_cache_set_salted()`, `wp_cache_get_multiple_salted()`, `wp_cache_set_multiple_salted()`.
      **The order of items in the array of salts must be consistent** or cache hits will be missed.
      Source: make.wordpress.org/core/2025/11/17/consistent-cache-keys-for-query-groups-in-wordpress-6-9/

---

## Accessibility Checklist (WCAG 2.2 AA)

- [ ] Interactive elements use `<button>` or `<a>` — never `<div onClick>` or `<span onClick>`
- [ ] All `<img>` have descriptive `alt`; decorative images use `alt=""`
- [ ] Color contrast ≥ 4.5:1 for body text; ≥ 3:1 for large text and UI components
- [ ] `:focus-visible` styles present and visible on every interactive element
- [ ] Off-canvas / modal panels use `role="dialog"`, `aria-modal="true"`, `aria-labelledby`,
      Escape key close, focus trap, **CSS class toggle for visibility**, and **`inert` attribute
      toggled by JS** to block keyboard Tab into hidden panels — never `aria-hidden` on a
      `role="dialog"` element (ARIA spec conflict); `visibility:hidden` alone does not block
      keyboard focus (see offcanvas-ui.md for the complete pattern)
- [ ] Dynamic content changes announced via `aria-live="polite"` (non-critical) or `"assertive"` (errors)
- [ ] Form labels explicitly associated via `for`/`id` pairing or `aria-labelledby`
- [ ] Full keyboard navigation without mouse — Tab, Shift+Tab, Enter, Escape all functional
