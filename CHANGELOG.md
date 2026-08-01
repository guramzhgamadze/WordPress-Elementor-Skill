# Changelog — WordPress & Elementor Pro Skill

Audit history and version-tracking for the skill. The router (`SKILL.md`) carries only the
**current** stack; this file holds the release-by-release detail so volatile facts live in one
place instead of being scattered across the sub-files.

---

## Current stack (keep in sync with `SKILL.md` §1)

| Component | Version | Released |
|---|---|---|
| WordPress | **7.0.2** ("Armstrong" 7.0: May 20, 2026) | July 17, 2026 (security) |
| PHP (recommended / minimum) | 8.3 / 7.4 | — |
| Elementor (free + Pro, independent versions) | **4.2.0** both | July 20, 2026 |
| WooCommerce | **10.9.4** (10.9.0: June 23, 2026) | July 7, 2026 |

**Sources:**
- wordpress.org/download/releases/ (7.0.2 · 7.0.1) · make.wordpress.org/core/2026/07/03/wordpress-7-1-release-party-schedule/ (7.1 → Aug 19, 2026)
- make.wordpress.org/core/2026/01/09/dropping-support-for-php-7-2-and-7-3/ (PHP 7.4 minimum)
- wordpress.org/plugins/elementor/ + api.wordpress.org plugin info (free 4.2.0, Jul 20) · elementor.com/pro/changelog/ (Pro 4.2.0, Jul 20)
- developer.woocommerce.com/releases/ · developer.woocommerce.com/2026/06/23/woocommerce-10-9/

---

## Audit rounds

### Round 31 — July 27, 2026 — editor panel tabs, dynamic-tag parents & the no-default colour rule
Three pieces of field-verified work, each from building a real plugin against a live Elementor
4.2.0 install rather than from documentation.

**Editor panel tabs** — sourced from Elementor 4.2.0's own shipped, unminified package
(`assets/js/packages/editor-elements-panel/editor-elements-panel.js`) and from building a working
panel tab end-to-end.
- **New `elementor-extending.md` §5 — "Add a TAB to the editor's Elements panel."** Elementor 4.x
  exposes an official `window.elementorV2.editorElementsPanel.injectTab({ id, label, component,
  position })`, which performs the `panel/elements/regionViews` filter, `addTab()` and the nav
  button in one call and renders a React component through a Portal into
  `#elementor-panel-elements-wrapper`. Script handle: `elementor-v2-editor-elements-panel`.
  Includes the PHP enqueue (`elementor/editor/before_enqueue_scripts`, capability-gated on the
  post in the editor URL) and a defensive registration pattern that polls briefly and then fails
  **quietly** — an exception here takes out the entire panel.
- **Explicit anti-pattern recorded:** do *not* buffer `elementor/editor/footer` and
  `preg_replace()` a tab button into Elementor's rendered markup (a major SEO plugin does this,
  matching `data-tab="global"` and `elementor-component-tab`, and ships a fallback for when the
  regex fails). It breaks silently on any class rename; `injectTab()` removes the need.
- **Two traps that pass every linter and only appear in a live editor**, now documented:
  (1) anything the panel's REST route calls **must live outside the `is_admin()` include block** —
  a REST request is not an admin request, so an admin-only rendering helper fatals with
  "undefined function" while the meta box using it works; (2) **the Elementor 4.x panel is LIGHT,
  not dark** — with the real 4.2 token values tabled, including that `--e-a-btn-bg-primary` is a
  pale pink (`#f3bafd`) expecting dark text, and that white-alpha tracks are invisible on it. Use
  neutral grey alpha and mid-tone semantic colours, and verify via computed styles.
- Router (`SKILL.md`) sub-file map and §4 pattern index updated; `elementor-extending.md` sections
  renumbered 5→6, 6→7, 7→8 with the internal `§7`→`§8` cross-reference corrected; the JS and PHP
  hooks quick-reference tables gained the panel-tab entries.

**Dynamic-tag parents & empty-value discipline** (`elementor-patterns.md`) — from a real
Dynamic Tags plugin:
- Documented the **two dynamic-tag parents** and that `get_content_type()` is `final` on both:
  `\Elementor\Core\DynamicTags\Tag` ('ui' — Elementor output-buffers `render()`) for
  TEXT/URL/COLOR/NUMBER, and `\Elementor\Core\DynamicTags\Data_Tag` ('plain' — returns
  `get_value()`) for **IMAGE/MEDIA/GALLERY**. An image tag written as `extends Tag` with a
  `get_value()` looks correct and **silently returns nothing** (the field appears empty, no error).
  The parents are incompatible, so shared field logic goes in a **trait**; `Tag::WRAPPED_TAG`
  defaults `false`.
- **Empty image/gallery fields must return an EMPTY value — never
  `Utils::get_placeholder_image_src()`.** Elementor's image widget bails on an empty `url`, so a
  placeholder paints a grey box on every unset slot (worst on repeated optional slots, where an
  `absint`-sanitised field stores `0`). Pro's `ACF_Image` is the reference: empty by default,
  opt-in `fallback` MEDIA control. **Carousels/sliders accept only the GALLERY category** — an
  IMAGE tag can't fill them; gallery tags extend `Data_Tag` and return a flat `[['id'=>int]]`
  list (`id` mandatory — verified against the Pro Gallery, free Gallery/Carousel and Pro
  `ACF_Gallery` consumers, not assumed).

**Colour-control defaults — the "no default you can't switch off" rule** (`SKILL.md` §5 +
`field-notes.md` §4):
- A COLOUR control with a `'default'` can never be turned off — Elementor emits it exactly like a
  user value and clearing the swatch restores it. **Core carries no colour defaults** (verified in
  `includes/widgets/heading.php`): resting look belongs in your stylesheet's `var(--token, …)`;
  **state** colours (hover/active/selected/current) are **empty** controls that write a **direct
  CSS property** (never a custom property with a fallback), and your CSS must leave that state
  colour-free — otherwise "empty" still paints the fallback.
- Five more live-editor field notes now in §4: theme form-control selectors
  (`input[type="search"]:focus`, 0-2-1) outrank bare classes — **element-qualify** interactive
  selectors; `container-type: inline-size` can inflate a wrapper to **~63,000px** at narrow
  viewports (prefer an `@media` breakpoint + `vw`-based `clamp()` over `cqi`); **guard each
  `register_taxonomy()` independently** of a `post_type_exists()` early-return; renaming a taxonomy
  is one `wp_term_taxonomy` column + `clean_taxonomy_cache()`; and `accent-color` on a range input
  is dead once a theme sets `appearance: none` — style the `::-webkit-*` / `::-moz-*` pseudos.

### Round 30 — July 22, 2026 — live-source currency sweep (WP 7.0.2 / Elementor 4.2.0 / WC 10.9)
All facts verified against wordpress.org, the wp.org plugins API, elementor.com/pro/changelog,
developer.woocommerce.com, and developers.elementor.com.
- **Corrected a Round 21 error:** "Elementor 4.2.0 (June 5, 2026)" was wrong — free and Pro
  were on 4.1.x then, and their version numbers are **independent**, not shared. Both actually
  reached **4.2.0 on July 20, 2026** (free: **Atomic Grid**; Pro: **Atomic Loop** + Atomic Forms
  enhancements). Fixed in `SKILL.md`, this file's stack table, README, and `docs/index.html`.
- **WordPress 7.0.2** (July 17, security; 7.0.1 July 9 = 31 fixes + PHP 8.5 compat) noted in the
  stack; **WP 7.1 scheduled for Aug 19, 2026** flagged as the next audit trigger.
- **WooCommerce 10.8 → 10.9.4**: new `woocommerce.md` note — Store API **defers draft-order
  creation** to near place-order time (code assuming an early draft order breaks), product
  editor beta in its **final deprecation window** (removed in WC 11.0), Abilities/MCP domain
  abilities for products/orders, experimental code-API + GraphQL, transactional email logging
  in core. No HPOS or minimum-requirement changes.
- **V4 stance re-verified today**: developers.elementor.com still documents only V3
  `Widget_Base`; no third-party Atomic extension docs. Skill keeps targeting V3; the V4
  awareness list now includes Atomic Grid and Atomic Loop. Plugin Check still 2.0.0 (unchanged).

### Round 29 — July 22, 2026 — second CLAUDE.md mining pass (post-review lessons)
Re-mined the four project CLAUDE.md files that changed since the July 11 pass
(zen-mcp-bridge — a full wp.org review cycle, zen-site-security, zen-login-authentication,
Biology exam). Added only lessons the skill didn't already carry; no new files, no renumbering.
- **`debugging.md`**: new "`$wpdb` sniffs & suppression mechanics" subsection (block-form
  `phpcs:disable` for DB code, built-SQL `PreparedSQL.NotPrepared`, ≥ 1 placeholder rule, name
  EVERY firing sniff, `ExceptionNotEscaped`, core-private functions, the
  `Requires at least` compatibility hard-gate) + an OWASP-ZAP four-bucket triage table
  (real / false-positive / intentional / environmental).
- **`wp-org-guidelines.md`**: review escalation (AUTOPREREVIEW bot → human, fix the whole class,
  same-thread concise replies, one complete update per round) + new **"Hard lines from real 2026
  reviews"** section — no user-authored SQL ever (whitelist identifiers, parameterize values),
  no `AUTH_KEY`/salt reuse, `== External services ==` required even for inbound services,
  protocol-endpoint nonce exceptions are explained not faked, `wp_print_styles()` for standalone
  HTML pages. Sharpened `Tested up to` (must equal current WP major).
- **`wordpress-apis.md`** §2: the sanitize-callback trap — it fires on EVERY `update_option()`
  including programmatic writes; pure function of `$input`; runtime state lives in an
  unregistered option; WP-CLI (`is_admin()` false) can't reproduce it without adding the filter.
- **`php-standards.md`**: transients under a persistent object cache — SQL `DELETE` on
  `wp_options` silently no-ops; use a generation counter / `delete_transient()` registry.
- **`field-notes.md`**: §1 sibling-plugin detection at runtime (alphabetical load order); §6
  send-once header guard + the three header hooks, never-build-a-regex-WAF, practical CSP scope,
  web-cache-deception `no-store` defense, shared secret-redaction helper, transient-flash + PRG
  notices; §9 line-anchored managed-marker regexes; §10 `.css`/`.min.css` parity and
  zip forward-slash verification + cloud-sync caveat.
- **`SKILL.md`**: Abilities API bullet now carries the registration-timing gotcha
  (`wp_abilities_api_init` / `wp_abilities_api_categories_init`, required pre-registered
  category, real `Requires at least: 6.9` floor).

### Round 28 — July 11, 2026 — bundled a standalone SVN skill
- Reviewed a user-authored **Subversion (SVN)** skill (a `.skill` zip: `SKILL.md` + 4
  `references/` files, ~1,205 lines, distilled from the official SVN Book 1.7). Checked every
  claim against SVN behaviour — **accurate throughout, no fixes needed** (verified: `--reintegrate`
  auto since 1.8, `^/` since 1.6, peg revisions, the `enable-auto-props` gotcha, the three
  meanings of "lock", error codes, `svnadmin`/`svnlook` local-path-not-URL, BDB-only `recover`,
  and the WordPress.org `trunk`/`tags`/`assets` + `Stable tag` specifics that match our own files).
- **Integrated as a self-contained sub-bundle at `svn/`** (preserved 1:1 — `svn/svn.md` +
  `svn/references/`). Wired in via the router (sub-file map + 2 Quick-Reference rows) and
  cross-linked from `wp-org-guidelines.md` and `field-notes.md` §10 ("the directory runs on SVN;
  here's how to actually deploy"). First subfolder in the otherwise-flat skill — a deliberate
  choice to keep the donated skill intact and loadable on demand.
- README + `docs/index.html` updated: 49 → **54** files (14 core + 35 widget + a 5-file SVN
  sub-bundle).

### Round 27 — July 11, 2026 — Elementor extension points (from developers.elementor.com)
- Mapped developers.elementor.com against the skill. Widgets/Controls/Dynamic Tags/Form Actions/
  Theme Conditions were already covered deeply; the documented **components** the skill lacked are
  now in a new **`elementor-extending.md`** (14th core file):
  - **Custom Form Fields** (`Field_Base` + `elementor_pro/forms/fields/register`, render, the
    Pro 3.28 method-based deps, and validation on the form lifecycle hooks).
  - **Theme Locations** (`elementor/theme/register_locations` + `register_location` +
    `elementor_theme_do_location()` with fallback) — the sibling of Theme Conditions.
  - **Extending NATIVE widgets** — inject a control via
    `elementor/element/{el}/{section}/before_section_end`, and filter output via
    `elementor/widget/render_content` (a common need the skill didn't cover).
  - **Finder** and **Context Menu** items (niche; Finder registrar flagged as version-variable).
  - A consolidated **PHP + JS hooks quick-reference** and a **deprecations** table (`get_id_int`,
    `_register_controls`/`_content_template`, `$depended_scripts`, schemes→globals,
    `widgets_registered`→`register`, `.elementor-widget-container`, addEventListener→jQuery).
- Wired into the router (sub-file map + 4 Quick-Reference rows), README, and `docs/index.html`
  (13 → **14** core; 48 → **49** total).
- **Both flagged items since VERIFIED** (web access restored): (1) the **`VISUAL_CHOICE`** control
  is real — `\Elementor\Controls_Manager::VISUAL_CHOICE`, image-based options (`title` + `image`),
  `columns`/`default`/`label_block`; added a concrete §7 example to `elementor-extending.md` and
  removed the hedge. (2) **No published third-party PHP API for custom Atomic Elements** exists as
  of mid-2026 (no `Atomic_Widget_Base`; docs still point to `Widget_Base`; atomic elements are
  documented only as a *data structure*) — the V4 note is now stated definitively, and the
  "keep targeting V3 `Widget_Base`" stance is confirmed correct.
  Sources: developers.elementor.com/docs/editor-controls/control-visual-choice/ ·
  developers.elementor.com/docs/data-structure/atomic-elements/

### Round 26 — July 11, 2026 — Common WordPress APIs (from developer.wordpress.org)
- Mapped developer.wordpress.org against the skill and filled the real gaps with a new
  **`wordpress-apis.md`** (13th core file) covering the Common APIs the skill lacked:
  - **Options API** — with the **WP 6.6** change: pass an explicit **boolean `autoload`** (not
    `'yes'`/`'no'`); `false` for large/rarely-used options (a real performance lever).
  - **Settings API** — the full review-safe admin-settings-page pattern (`register_setting` +
    `sanitize_callback` as the single trusted cleaning point + `settings_fields()` nonce +
    escaped field renderers + capability-gated page).
  - **Metadata API** — `register_post_meta` with `show_in_rest` (needed for Elementor Dynamic
    Tags / Loop) + sanitize/auth callbacks.
  - **Roles & Capabilities** — gate by capability, never by role; add custom caps on activation.
  - **WP-Cron** — scheduling + clear-on-deactivation, custom intervals, and the key caveats:
    it's **traffic-triggered (not real cron)** → `DISABLE_WP_CRON` + a system cron, and use
    **Action Scheduler** for heavy/reliable jobs.
  - **Internationalization** — text-domain = slug, the escaping i18n variants, and the **WP 6.7**
    rule: don't call `__()` before the `init` action (`_doing_it_wrong` "triggered too early");
    JS i18n via `wp_set_script_translations`; `wp i18n make-pot`.
- Verified the two recent changes live (WP 6.6 autoload, WP 6.7 translation timing). Wired into
  the router (sub-file map + 5 Quick-Reference rows), README, and `docs/index.html`
  (12 → **13** core; 47 → **48** total).

### Round 25 — July 11, 2026 — debugging & static-analysis guide
- New **`debugging.md`** (12th core file) — closes the gap between SKILL.md's advertised
  "debugging of WordPress/Elementor issues" and what the skill actually delivered. Covers:
  - **Static analysis** (lead section): **PHPCS + WPCS** (install, a committed `phpcs.xml.dist`
    ruleset with text-domain/prefix/min-WP config, security sniffs, `phpcbf`), **PHPStan**
    (WordPress extension, level, baseline, catching typed-override fatals), and **Plugin Check**
    (the review-grade superset that bundles PHPCS+WPCS) — with a table of what each tool does and
    doesn't catch, and the nuance that `--standard=WordPress` surfaces more than PC's review ruleset.
  - **Runtime**: `WP_DEBUG`/`WP_DEBUG_LOG`/`WP_DEBUG_DISPLAY`/`SCRIPT_DEBUG`/`SAVEQUERIES`,
    `debug.log`, Query Monitor, guarded `error_log()`, and WSOD (fatal) diagnosis.
  - **Elementor-specific**: Safe Mode, Regenerate Files & Data, element-cache freezes, editor-preview
    crashes, System Info.
  - **Frontend**: browser console / Network / **computed-styles** inspection for the `var()`/specificity
    class of bugs.
  - A **symptom → likely cause → first check** table seeded from the real `CLAUDE.md` cases.
- Wired into the router + README + `docs/index.html` (11 → **12** core; 46 → **47** total).

### Round 24 — July 11, 2026 — official wordpress.org submission reference
- Analyzed three canonical sources — the **Detailed Plugin Guidelines**, the **Plugin Check**
  plugin page, and the **make.wordpress.org/plugins** review-team blog — and distilled them into a
  new **`wp-org-guidelines.md`** (11th core file):
  - The **18 Directory Guidelines** in paraphrased, actionable form (kept numbered, since
    reviewers cite them by number) — GPL licensing, no obfuscation, no trialware, SaaS rules,
    no phone-home without opt-in consent, no remote executable code, no required credits/dashboard
    hijacking, ≤5 tags / no readme spam, use WP-bundled libraries, increment versions, trademark
    slug rules, etc.
  - **Plugin Check 2.0.0** — the six check categories (Plugin Repo Requirements, Security,
    Performance, Accessibility, Code Standards/PHPCS, Internationalization), admin-UI + WP-CLI
    usage, static-vs-runtime (`--require cli.php`), and the fact that it auto-scans **every update**
    since Oct 2025.
  - The **review process reality** — AI-assisted but human-decided, a days-to-~week queue, and the
    finding that **author responsiveness strongly correlates with approval** (silence is the most
    common rejection path).
  - Required headers / `readme.txt` fields, cross-referenced to `field-notes.md` §10.
- Wired into the router (sub-file map) and updated README + `docs/index.html` counts (10 → **11**
  core files; 45 → **46** total). Ties into existing skill rules (bundled-library use, `Requires`
  headers, the standing "run Plugin Check before resubmission" rule).
- **Added Golden Rule #7 — "Name for the directory from day one"** (SKILL.md §0): slug must not
  start with a trademark you don't own (Guideline 17), text domain must equal the slug, and the
  wp.org slug + a widget's `get_name()` are permanent. Distinct from Rule #3 (internal code
  prefixing) — this governs the public identity. Updated the README "principles" list (5 → 6) and
  the docs "golden rules" count (6 → 7); existing "Golden Rule #6" references are unchanged.

### Round 23 — July 11, 2026 — field notes from real plugins (+ two corrections)
- Mined the `CLAUDE.md` mistake-logs from two shipped Elementor plugins (a frontend-auth suite
  and an embedded exam app) and distilled the generalizable lessons into a new
  **`field-notes.md`** (10th core file) — widget-lifecycle fatals, `content_template()` escaping,
  CSS-in-Elementor footguns, transactional email, PHP/security gotchas, embedding standalone apps,
  data-safety, and wp.org review/packaging.
- **Correction 1 — `content_template()` escaping:** changed the guidance from `{{{ }}}` to
  **`{{ }}` (escaped) for user settings**. `{{{ }}}` on a user value is an editor-context XSS that
  wp.org review rejects; the old "avoids apostrophe corruption" rationale was a myth (the browser
  decodes the escaped entity). `{{{ }}}` is now reserved for Elementor-generated HTML
  (`renderIcon`, attribute strings). Updated `elementor-patterns.md`.
- **Correction 2 — `elementor/frontend/init` binding:** now leads with the **jQuery** binding
  (`jQuery(window).on('elementor/frontend/init', …)`). Elementor fires it through jQuery; native
  `addEventListener` silently misses it on Elementor < 3.5 (3.5+ added dual-dispatch, so native
  also works on modern versions — but jQuery is the version-agnostic, documented-safe default).
  Updated `js-css-standards.md`. Source: developers.elementor.com/native-js-events-in-elementor/.
- Wired `field-notes.md` into the router (sub-file map + "skim it for any widget/plugin/wp.org
  work") and updated README + `docs/index.html` counts (9 → **10** core files; 44 → **45** total).
- Note: the 35 widget boilerplates still use `{{{ settings.* }}}` for text in their
  `content_template()` examples — a follow-up sweep to `{{ }}` is recommended for wp.org-bound use
  (tracked, not yet applied).

### Round 22 — July 11, 2026 — widget coverage expansion
- Checked Elementor 4.0–4.2 for newly added widgets. The new elements are all **V4 Atomic
  Elements** (Div Block, Flexbox Container, Atomic Heading/Paragraph/Image/Button/Video/SVG,
  Atomic Tabs, and Atomic Forms + composable fields — Radio/Select/Date/Time/File Upload added
  in Pro 4.1.0). These remain out of scope for third-party building (V4 extension docs still
  finalizing); added an **awareness note** enumerating them in `SKILL.md` §1.
- Closed pre-existing coverage gaps in classic free native widgets:
  - **New `widget-text-path.md`** — text along a curved/custom SVG path (`<textPath>` + unique
    per-instance path id; preset shapes by default, with an SVG-sanitization note for custom uploads).
  - **New `widget-nested.md`** — guide for **Nested Tabs / Nested Accordion** built on
    `Widget_Nested_Base` (container panels via `print_child()`, repeater↔child sync, ARIA + keyboard
    handler, and the `<details>`/`<summary>` route for Accordion). Honestly flags the semi-internal
    parts of the nested-elements editor API.
  - **Off-Canvas:** added a "native widget vs custom-code pattern" decision note to `offcanvas-ui.md`.
- Updated the router map (sub-file map + Quick Reference) and `docs/index.html` counts
  (33 → **35** widget files; 42 → **44** total).

### Round 21 — July 11, 2026 — post-release currency sweep + restructure
- **WordPress 7.0 "Armstrong" shipped May 20, 2026** (delayed from the original April 9 target
  while the Real-Time Collaboration storage layer was redesigned into a **dedicated core database
  table**, replacing the rejected `wp_post_meta` / `wp_sync_storage` approach).
- **Elementor 4.2.0** (June 5, 2026) is current; **WooCommerce 10.8** (May 26, 2026) is current.
  *(Round 30 correction: the Elementor claim was wrong — on June 5 free/Pro were on 4.1.x;
  both actually reached 4.2.0 on **July 20, 2026**, and free & Pro version numbers are
  independent, not shared.)*
- Replaced ~115 lines of WP 7.0 beta/RC/delay narration in `SKILL.md` §1 with a compact,
  durable "current stack" table + a short "what changed in WP 7.0 for devs" summary.
- Settled forward-looking sections into shipped fact: `woocommerce.md` (WC 10.7 sync-on-read),
  `performance.md` (RTC query scoping), `scaffolding.md` (WP 7.0 PHP minimum).
- Bumped stale Elementor version labels in `elementor-patterns.md` (V4 preamble, Dynamic Tag
  category reference).
- **Consolidated duplicate widget-template files:** merged `widget-template-elementor.md` into
  `widget-elementor-template.md`, and `widget-template-php.md` into `widget-php-template.md`
  (deleted the two mirror-named twins; updated the router map). Fixed a self-contradiction in
  the old Elementor-template twin about whether `get_builder_content_for_display()` switches the
  global `$post` (it does not).
- Clarified `get_render_attribute_string()` vs `print_render_attribute_string()` usage in
  `SKILL.md` §5.
- Refreshed `README.md` and `docs/index.html`; moved release history into this file.

**Per-widget audit (all 33 widget boilerplates reviewed line-by-line):**
- **`widget-toggle.md` — fixed real bug:** the expand/collapse icon guard checked
  `$tab['selected_icon']` (a repeater key) but `selected_icon` is a widget-level control, so
  the guard was always false and the icon never rendered. Now reads `$settings['selected_icon']`
  in both `render()` and `content_template()` (the same fix `widget-accordion.md` already had).
- **`widget-icon-list.md` — fixed false claim:** a comment stated
  `add_inline_editing_attributes()` is "deprecated since Elementor 3.x." It is not, and it is
  recommended in `SKILL.md` §5 — corrected to an accurate, optional-usage note.
- **`widget-social-icons.md` — fixed accessibility defect:** icon-only links had no accessible
  name (WCAG 2.4.4 / 4.1.2). Added an `.myplugin-screen-only` label derived from the icon class.
- **`widget-tabs.md`:** added the missing ARIA APG keyboard-interaction note (arrow keys +
  roving tabindex for `role="tab"`), matching the notes in accordion/toggle.
- **`widget-image.md`:** added the `phpcs:ignore` comment its sibling files already carry on the
  Elementor-generated `get_attachment_image_html()` echo (lint consistency).
- Bulk-updated the stale "(Elementor 3.35+)" verified-against labels in all 33 widget files to
  "(Elementor 3.35+ / V3 Widget_Base API, current through 4.2)."
- **Verified clean:** every widget implements `has_widget_inner_wrapper()`,
  `is_dynamic_content()`, and `content_template()`; output is escaped (`esc_html`/`esc_attr`/
  `esc_url`/`wp_kses_post`, with documented `phpcs:ignore` only for trusted oEmbed/`do_shortcode`/
  raw-HTML cases); no `.elementor-widget-container` targeting; Swiper/Counter scripts correctly
  avoid the `strategy:defer` + `elementor/frontend/init` trap.
- **`widget-alert.md` — made type colors functional:** the type now drives a
  `.myplugin-alert--{type}` modifier class on the alert element, backed by a shipped CSS block
  with default info/success/warning/danger schemes; added Background/Text color controls so the
  defaults stay user-overridable (Golden Rule #6). Previously the type wrote a class to the
  widget wrapper and silently relied on Elementor's native alert CSS.
- **`widget-social-icons.md` — wired up `item_icon_color`:** added a per-item Custom Color
  picker (shown when "Custom" is selected) using a `{{CURRENT_ITEM}}` selector, and added the
  `elementor-repeater-item-{id}` class to each link so the token resolves. Relabeled the
  unimplemented "Official Color" option to the honest "Default".
- **`widget-testimonial.md` — wired up the `link` control:** the name is now wrapped in an `<a>`
  when a URL is set (in both `render()` and `content_template()`), matching heading/icon-box.

### Round 20 — April 2, 2026 — Elementor 4.0.0 release
- Elementor 4.0.0 released March 30, 2026 (free + Pro simultaneously). Updated version numbers
  from 3.35.x → 4.0.0 in `SKILL.md` and `elementor-patterns.md`. Atomic Editor became Stable and
  default for new installs; V4 PHP extension API stable but third-party docs still finalizing —
  skill continues targeting V3 `Widget_Base`.

### Round 19
- Live-source audit: updated Elementor 4.0 Beta status across 4 files; confirmed V3 `Widget_Base`
  fully supported; updated V4 rules preamble.

### Round 18
- Fixed backdrop overlay visible before JS load (`offcanvas-ui.md`); added pre-stable caveat to
  WP 7.0 AI Client API names; added `woocommerce_hpos_enable_sync_on_read` source citation;
  46-file full scan.

### Round 17
- Fixed missing `inert` attribute on off-canvas HTML panel; added JSON Schema draft-04 comment to
  `rest-api.md`; synced WP 7.0 Beta 6 / RC1 details.

### Round 16 — March 16, 2026
- Full verification pass; all facts confirmed clean against live sources.

### Round 15
- WordPress 6.9.3 → 6.9.4; WP 7.0 Beta 5 confirmed released; `wp_kces_post` typo fixed →
  `wp_kses_post`.

### Round 14
- Split the original monolithic `SKILL.md` into a 9-file bundle; Elementor free 3.35.6 → 3.35.7.

### Rounds 1–13
- Original monolithic `SKILL.md` (56 bugs fixed).
