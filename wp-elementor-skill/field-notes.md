# Field Notes — Hard-Won Production Gotchas

> **When to read this file:** Alongside the topic files whenever building a custom widget, plugin,
> or wp.org-bound work — this is the "why it actually breaks" layer. Every rule exists because the
> opposite **shipped and broke something**; these are the bugs that pass `php -l` / `node --check`,
> survive code review, and only surface on a live page or in a wp.org submission.

Lessons distilled from shipping real WordPress/Elementor plugins (auth forms, an embedded
exam app).

---

## 1. Elementor widget lifecycle — the fatal & the silent

- **An untyped override of a typed parent method is a site-down fatal.** Elementor declares
  `has_widget_inner_wrapper(): bool`, `is_dynamic_content(): bool`, `get_categories(): array`,
  etc. Overriding one **without the matching return type** is a PHP fatal that white-screens
  **every page** (these run on `wp_enqueue_scripts`). `php -l` checks syntax only — it does
  **not** catch a signature mismatch. Always copy the parent's return type exactly, and
  instantiate the widget against a **typed `\Elementor\Widget_Base` stub** before shipping to
  catch it at class-declaration time.
- **`render()` runs on EVERY control change in the editor** — dozens of times per session. Any
  `add_filter()` / `add_action()` you set *inside* `render()` must be removed immediately after
  (add → render → remove, request-scoped). Leaked anonymous-closure filters stack up across
  re-renders, producing duplicated/mangled output and "undefined index" notices that crash the
  preview iframe.
- **`get_name()` is sticky — renaming it breaks every existing page.** The widget name is stored
  as `"widgetType"` inside each page's `_elementor_data` JSON. Change it and every placed
  instance renders as "widget not found." When refactoring/prefixing, **keep `get_name()`** and
  rename only the PHP class. If you truly must rename it, migrate the stored `_elementor_data`
  tokens **and** the `frontend/element_ready/<name>.default` hook names in JS together.
- **Pre-register asset handles before declaring them.** Handles returned from
  `get_style_depends()` / `get_script_depends()` must already be `wp_register_style/script`-ed
  (on `wp_enqueue_scripts`) or Elementor's dependency enqueue **silently no-ops** — your CSS/JS
  never loads and there's no error.
- **Registration plumbing is guard-heavy on purpose.** Register widget *instances* on
  `elementor/widgets/register`; register the category on
  `elementor/elements/categories_registered`. Gate `require_once` with
  `did_action('elementor/loaded')` **and** `class_exists('\Elementor\Widget_Base')`. Wrap
  optional group controls (e.g. `Group_Control_Box_Shadow`) in `class_exists()` for
  cross-version safety. (If preload/opcache/static-analysis chokes on `\Elementor\*` type hints
  in a hook callback, drop the hint and verify with `class_exists()` inside instead.)
- **Detect companion/sibling plugins at RUNTIME, never at file-load.** Plugins load in
  alphabetical order, so a constant/class another plugin defines may not exist yet when your file
  is parsed — a file-scope `defined()` check silently returns false for any plugin that sorts
  after yours. Check `defined()` / `class_exists()` from `plugins_loaded` (or later), inside the
  behaviour that needs it.
- **A new UI section inside a widget MUST ship its own Style controls (Golden Rule #6).** Adding
  markup with CSS-only and no controls means its heading/colours inherit the theme with no way
  to change them — exactly the "I can't style this" complaint. Every new section needs a matching
  control group (heading colour/typography/alignment, body text, bg/border, links), and any new
  button must reuse an already-controlled class or get its own controls.

---

## 2. `content_template()` escaping — the editor-XSS that wp.org rejects

> ⚠️ **Use `{{ }}` (escaped) for user settings — NOT `{{{ }}}`.** This corrects the instinct to
> reach for triple-brace "to avoid double-encoding."

- **`{{{ settings.* }}}` on a user-controlled value is an editor-context XSS** and **wp.org plugin
  review rejects it** (real rejection on `WordPress.Security.EscapeOutput`). `{{{ }}}` injects raw
  HTML into the Backbone preview; `{{ }}` runs it through `_.escape()`. Escaped output renders
  **correctly** in the browser — `O'Reilly` → `O&#x27;Reilly` displays as `O'Reilly` (the entity
  decodes on render). There is **no apostrophe corruption**; that was a myth.
- **Rule:** `{{ }}` for ALL interpolated settings — titles, labels, button text, plain
  `TEXT`/`TEXTAREA`. Reserve `{{{ }}}` **only** for Elementor-generated HTML that is not raw user
  input: `{{{ iconHTML.value }}}` from `elementor.helpers.renderIcon( ... )`, and processed
  media. Even inside HTML **attributes**, use `{{ }}` — a triple-brace in an attribute breaks the
  attribute.
- **Plugin Check flags the PHP `echo $this->method()` wrapper, not the brace.** A method that
  *returns* a built HTML string which is then `echo`'d is always flagged. Fix the **source**:
  make the method echo internally (literals + `esc_*`) and call it as a bare statement. A
  `phpcs:ignore … "escaped during construction"` here **hides** the real `{{{ }}}` issue —
  reviewers grep for exactly those justification phrases.

---

## 3. Output escaping & the wp.org "escape on output" review

- **Escape LATE, at the point of output — building a value "safely" earlier does not count.**
  Every `$`-variable, option, and generated value is escaped where it is echoed, with the
  context-appropriate function (`esc_html` / `esc_attr` / `esc_url` / `esc_textarea` /
  `wp_kses` / `wp_kses_post`, or the `esc_html__()` / `esc_html_e()` i18n variants).
  Source: developer.wordpress.org/apis/security/escaping/
- **Plugin Check flags `echo $var` / `echo $this->method()` / `echo func()`** — it does **not**
  flag string literals or `esc_*()` / `wp_kses*()` calls. Fix a flagged echo by wrapping the
  dynamic part, or by restructuring so the echo is literals + escapers only.
- **`phpcs:ignore` is only for genuine false positives with a true, verifiable reason** (e.g.
  "static inline SVG, no user input"). Never use it to silence a real finding — reviewers
  literally grep for the justification text. When in doubt, fix the source.
- **Justified, unavoidable false positives that DO get a tagged ignore:**
  - `NonPrefixedHooknameFound` on **core** hooks you fire/filter — `do_action('wp_login', …)`,
    `apply_filters('login_redirect', …)`, `login_enqueue_scripts`, and third-party cache hooks
    you must match by exact name.
  - `NonPrefixedConstantFound` on cache opt-out signals — `DONOTCACHEPAGE`, `DONOTCACHEOBJECT`,
    `DONOTMINIFY`, `DONOTROCKETOPTIMIZE` (each behind `! defined()`); they MUST match the exact
    names cache plugins check, so they can't be prefixed.
- **Inputs too:** `wp_unslash()` then sanitize on **every** superglobal read — including
  `$_SERVER`: `sanitize_text_field( wp_unslash( $_SERVER['REQUEST_METHOD'] ?? '' ) )`. Plugin
  Check raises `MissingUnslash` + `InputNotSanitized` otherwise.
- **Vendored libraries: don't edit them, disclose them.** A bundled MIT lib that trips dozens of
  `EscapeOutput`/`AlternativeFunctions` errors stays unmodified (so it keeps updating cleanly);
  exclude its directory from your Plugin Check run and note it in the review reply, *provided* its
  output is never echoed (e.g. all its exceptions are caught internally).

---

## 4. CSS architecture — the rules that bite inside Elementor

- **Never style on bare ARIA attributes.** `[aria-hidden="true"] { display:none }` (e.g. to hide
  a honeypot) also hides required-field asterisks and decorative SVGs. Give the element its own
  class (`.myplugin-hp`) and style that. Found only by inspecting **computed styles in a live
  DOM** — screenshots "looked fine."
- **CSS custom properties cascade parent → child only — never upward.** In Elementor the markup is
  often `.myplugin-wrap > .elementor-widget… .myplugin` and the **outer wrapper does not carry
  your root class**, so a rule on the wrapper that reads `var(--myplugin-card-bg)` resolves to
  nothing if the variable is defined only on the inner element. Declare every `--myplugin-*`
  variable on a selector that **includes the wrapper** (`.myplugin, .myplugin-wrap { … }`) or
  provide a `var(x, fallback)`. Reproduce by rendering an actual Elementor **widget**, not just a
  shortcode.
- **A `min-width` floor on a flex child is a footgun.** A flex child holding unbreakable user
  data (a long email, a URL) with `flex:1; min-width:180px` cannot shrink below 180px → it
  overflows and collides with the next column. Flex children that hold user data get
  **`min-width:0`** (so they can shrink) + **`overflow-wrap:anywhere; word-break:break-word`** (so
  the string wraps).
- **Body-appended UI escapes your scoped token container.** A modal/popup mounted via
  `document.body.appendChild` sits **outside** your `.myplugin-scope`, so its `var(--accent)` /
  `--fs-*` / fonts resolve to nothing and it renders unstyled. Give body-mounted UI its **own
  token block** (`.myplugin-modal-overlay { …same tokens… }`). Don't fix it by moving the node
  inside the scope — that subjects it to the scope's element resets. (Relevant to off-canvas
  panels, lightboxes, and any `position:fixed` overlay — see `offcanvas-ui.md`.)
- **Style controls must target the OUTERMOST wrapper the widget owns.** Targeting an inner element
  leaves the title/border/card outside the styled area. Pair with the `{{WRAPPER}}` selector
  rules in `js-css-standards.md`.
- **`{{WRAPPER}}` selectors can't cross-reference another control's value.** A combined visual
  effect (e.g. focus-glow spread *and* colour) needs **two** controls feeding one selector. Use
  `CHOOSE` + `selectors_dictionary` for non-numeric CSS toggles (not `SELECT`). URL controls
  return an **array** (`['url' => …, 'is_external' => …]`), not a string — read `['url']`.
- **A COLOUR control with a `default` can never be switched off.** Elementor emits a control's
  `default` exactly like a user-set value, so `'default' => '#6B4C9A'` paints that colour on every
  install and clearing the swatch just restores it. Users read that as "the widget forces a colour
  on me." **Elementor core carries no `default` on colour controls at all** — check
  `includes/widgets/heading.php`: `title_color` uses `'global' => ['default' => Global_Colors::COLOR_PRIMARY]`
  (a *reference* the user can clear) and `title_hover_color` is **completely empty — no default, no
  global**. Empty ⇒ nothing emitted ⇒ the stylesheet's resting value stands. So: **resting look
  lives in your CSS; state colours (hover / active / selected / current) are empty controls.** Two
  consequences that are easy to miss:
  - **State colours must write a direct CSS property, never a custom property.** `--my-btn-bg:{{VALUE}}`
    read back as `var(--my-btn-bg, red)` means the fallback paints whenever the control is empty —
    "default" becomes "red", not "off". `'{{WRAPPER}} .btn:hover' => 'background:{{VALUE}};'` emits
    *no rule at all* when empty, which is the only way default means off.
  - **Then your stylesheet must not declare that state either.** Leaving
    `.btn:hover { background: var(--my-btn-bg, rgba(255,255,255,.5)) }` in the CSS re-introduces the
    same problem from the other side. Keep hover/selected rules **colour-free** — carry the state
    with `font-weight` or an inset highlight so the control genuinely owns every colour.
- **Theme form-control selectors outrank your bare classes.** `input[type="search"]:focus` is
  **0-2-1** (element + attribute + pseudo-class); your `.my-search:focus` is **0-2-0** — the theme
  wins. Astra ships exactly that rule and repaints focused inputs its own blue, which then persists
  until blur and reads as a bug in *your* widget. Same trap on `input[type="range"]::-webkit-slider-thumb`
  (0-1-1 beats `.my-range::-webkit-slider-thumb` at 0-1-0). Fix by **element-qualifying every
  interactive-control selector** — `input[type="search"].my-search:focus` (0-3-1),
  `.my-widget button.my-chip:hover` — which still sits far below Elementor's generated
  `.elementor-{id} .elementor-element.elementor-element-{id} .sel` (0-4-0+), so the controls keep
  winning. Reach for `!important` only after this fails; it would also lock the user's controls out.
- **`container-type: inline-size` can inflate the widget wrapper to tens of thousands of pixels.**
  Inside Elementor's nested flex containers (`.e-con` → `.e-con-inner` → `.elementor-element`),
  Chromium computed the widget wrapper at **62,926px against 4,867px of real content** — ~58,000px
  of blank page below the widget — but **only at narrow viewports**; desktop was exact. The
  giveaway is a parent taller than its only child. Bisect by setting `container-type: normal`
  inline: if the height collapses, that is it. Nothing else fixed it — not `min-height: 0`,
  `display: flow-root`, `contain-intrinsic-block-size`, `align-self: stretch`, nor removing the
  flex context from the wrapper — and neither the `@container` rules nor the `cqi` units were
  involved (neutralising both changed nothing). Unless you genuinely need the widget to respond
  to *its own* width rather than the viewport's, prefer a plain `@media` query at Elementor's
  breakpoint (767px mobile / 1024px tablet) and a `vw`-based `clamp()` instead of `cqi`.
- **Guard independent registrations independently.** A CPT-registration function that bails with
  `if ( post_type_exists( $slug ) ) { return; }` — so a page-builder plugin like CPT UI can stay
  the owner — will also skip every `register_taxonomy()` call after it. On every site where the
  other plugin owns the post type the taxonomy is then never registered at all, and each
  `wp_get_object_terms()` returns `WP_Error( 'invalid_taxonomy' )`. Devastating right after a
  migration that moved terms onto that taxonomy. Guard each registration with its own
  `post_type_exists()` / `taxonomy_exists()` check.
- **Renaming a taxonomy is one column.** `UPDATE wp_term_taxonomy SET taxonomy = 'new' WHERE
  taxonomy = 'old'` is the whole migration: `wp_term_relationships` points at `term_taxonomy_id`,
  which does not change, so every object keeps exactly the terms it had. Follow with
  `clean_taxonomy_cache()` for both names (term caches key on the taxonomy name, so the old one
  lingers as a phantom) and a rewrite flush if the archive slug changed.
- **`accent-color` on a range input is dead the moment a theme sets `appearance: none`.** It only
  tints *native* track/thumb rendering, so a control that writes `accent-color` is inert and the
  slider renders as a bare rectangle. Style the parts explicitly and element-qualified:
  `::-webkit-slider-runnable-track` + `::-webkit-slider-thumb` (needs its own
  `-webkit-appearance:none` and a `margin-top` to re-centre it on a thin track) and
  `::-moz-range-track` / `::-moz-range-progress` / `::-moz-range-thumb`. WebKit has **no**
  `::-moz-range-progress` equivalent — draw the filled portion as a background sized from a
  `--pct` custom property the script updates on `input`. Keep the input itself at a 44px hit
  height with a transparent background and let the visible track be thin.

---

## 5. Embedding a standalone app as an Elementor widget

- **Render inline — never in a cross-document `<iframe>`.** Elementor's style controls inject CSS
  onto the **parent** page; that CSS **cannot cross into a cross-document iframe**, so
  "controls on the toolbar style the app" is impossible with an iframe. Iframe embedding is a
  dead end for anything you want Elementor-styleable.
- **Inline means the theme's and Elementor's CSS bleed IN, not just out.** Elementor's frontend
  rules (`.elementor img { height:auto }`) and the active theme **out-specify** your single-class
  selectors → giant logos, broken layout. Boost specificity by **doubling your root class**
  (`.myapp.myapp .thing`, specificity (0,2,…) beats `.elementor .thing`), add defensive resets,
  and scope variable-setting (theme) controls to `{{WRAPPER}} .myapp-scope`.
- **Per-element control selectors must out-specify your boosted base.** If base rules are
  `.myapp.myapp .x` (0,3,0), a normal `{{WRAPPER}} .x` control selector (0,2,0) **loses** and the
  control does nothing — target `{{WRAPPER}} .myapp.myapp .x`. Engine-rendered **inline**
  `style="color:…"` can't be overridden by any control — don't expose controls for those values.
- **`container-type: inline-size` + a non-stretch flex parent collapses to a sliver.** A
  size-containment element has zero intrinsic width; an `align-items:center` parent shrink-wraps
  it to nothing. Default widget wrappers to `width:100%` and centre via a Max-width control, not
  the container's alignment. Use container units (`cqw`) instead of `vw` so the widget responds to
  its own width, not the viewport.
- **A blanket `button {}` reset breaks native buttons and lets the theme bleed through.** Keep
  `text-align:center; line-height:normal` in the reset and harden specific buttons against theme
  rules (e.g. Astra's `button { background; color; padding }`).

---

## 6. PHP & security gotchas

- **Read cookies from `$_COOKIE`, not `$_REQUEST`.** PHP's default `request_order = "GP"` means
  `$_REQUEST` holds GET+POST only — **never cookies**. A cookie read through a `$_REQUEST`-based
  helper always comes back empty (e.g. "new device" every login). Read `$_COOKIE['name']`
  directly, then `wp_unslash()` + validate.
- **Secrets are encrypted at rest and never re-rendered in admin HTML.** Don't store an API
  client secret as a plaintext option and echo it back into `<input value="…">` — DB-level tools
  read it in full. Encrypt at rest (e.g. AES-256-GCM keyed from wp-config salts), render an
  **empty** field with a "leave blank to keep" sanitizer, and never assume a third-party
  "redaction" layer knows your option names — verify empirically.
- **Rate-limit on `REMOTE_ADDR` by default.** Forwarded headers (`X-Forwarded-For`, etc.) are
  trivially spoofable; trusting them by default lets an attacker rotate fake IPs past the limiter.
  Make forwarded-header trust **opt-in** via a filter, for sites genuinely behind a known proxy.
- **Policy decisions belong on the core filter, not one handler.** A subscriber/redirect rule
  enforced only inside your own login handler is bypassed by `wp-login.php` and third-party login
  flows. Put it on the core filter (`login_redirect`, high priority) so it applies everywhere.
- **Don't `wp_kses()` inline SVG — it lowercases `viewBox` and breaks the icon.** `wp_kses_hair()`
  lowercases every attribute name, but SVG attributes are **case-sensitive**
  (`viewBox` → `viewbox` is ignored → the icon clips). For static developer SVG, inline it as a
  **string literal** (no variable) so Plugin Check sees safe literal output; for SVG that must
  come from a function in `render()`, a justified `// phpcs:ignore … -- static inline SVG, no
  user input` is acceptable.
- **`sanitize_text_field()` always returns a string** (even `''` for array/object input). A
  following `is_string()` guard is dead code (PHPStan `function.alreadyNarrowedType`) — drop it.
- **Security headers need a send-once guard — and three hooks.** Other plugins send the same
  headers (X-Frame-Options, nosniff…); check `headers_list()` before sending so you never
  duplicate one already queued. Send on `send_headers` (front end) **plus** `admin_init` and
  `login_init` — `send_headers` alone never covers wp-admin or the login screen. Exception: a
  **strict** `Cache-Control: no-store, private` must overwrite whatever weaker value a
  cache/theme queued earlier — don't send-once-guard that one.
- **Never build a regex request-filter "WAF"** that pattern-matches SQLi/XSS/XXE payloads —
  bypass-prone, false-positive-prone, breaks sites, and wp.org reviewers reject it. A platform
  plugin defends **transport + configuration**; it cannot patch other code's bugs — document
  what's out of scope instead of shipping snake oil.
- **CSP on WordPress: only the nonce-free directives.** `object-src 'none'`, `base-uri 'self'`
  and `frame-ancestors` are deliverable; a real `script-src` policy is impractical on WordPress
  (inline scripts everywhere) — don't attempt it. Assemble the entire policy into **ONE**
  `Content-Security-Policy` header; never emit two.
- **Web-cache deception has a buildable defense:** send `Cache-Control: no-store, private` on
  (a) logged-in responses, (b) dynamic responses served under a static-looking extension
  (`/account.css`), and (c) REST auth responses — plus define the cache-plugin bypass constants
  (`DONOTCACHEPAGE` etc.). A broken logged-in cache bypass really does serve guest-cached HTML to
  authenticated users in production.
- **Secret redaction lives in ONE shared helper used by every output path.** Redacting an option
  name in one tool is pointless while another exit (a DB reader, an options dump, an export) can
  read the same value — route every endpoint through the same `is_secret_name()` / redaction
  functions.
- **One-time notices: transient flash + PRG, never a URL flag.** A "saved!" notice read from
  `$_GET` survives every refresh. Set a short **user-scoped transient**, redirect to a clean URL
  (Post/Redirect/Get), then read-and-`delete_transient()` on display — it shows once, never
  again.

---

## 7. Transactional email (`wp_mail`)

- **Email bodies are inline styles ONLY.** `wp_kses_post()` does not allow `<style>`/`<head>`, so
  it strips those **tags** but **keeps their text content**; `wpautop()` then wraps the leftover
  CSS in `<p>` and the mail client renders it as **visible body text**. WordPress's mailer also
  treats the body as an HTML **fragment** (`<html>`/`<head>`/`<style>` are not honoured, and email
  clients ignore `<style>` blocks anyway). **Strip `<style>/<head>/<script>/<link>` blocks
  (content included) and `<html>/<body>` wrappers BEFORE `wp_kses_post()`,** then style every
  element with `style="…"` attributes. Set `Content-Type: text/html` only on the custom-body path.

---

## 8. Theme Builder & editor context

- **Theme Builder targets a REAL page, not a virtual post.** Plugins that serve content from a
  virtual post (`ID = -1`) never match Elementor Theme Builder conditions (Singular > Page) — no
  header/footer/template applies and you get a bare unstyled page. Create **real WP pages** for
  each action and let Elementor target them by ID; keep virtual rewrite pages only as the
  non-Elementor fallback. Create/adopt those pages idempotently on activation (see §9).
- **Detect editor/preview context** when your code rewrites URLs or changes behaviour, so it
  stands aside in the builder. Check: `REST_REQUEST` + an `/elementor/` route, `?action=elementor`,
  `?elementor-preview`, `elementor_*` AJAX actions, and
  `\Elementor\Plugin::$instance->preview->is_preview_mode()`. Enqueue editor-only CSS on
  `elementor/editor/after_enqueue_styles`.
- **Refreshing `--var` defaults won't show on already-customised widgets, and Elementor caches
  per-page CSS.** A new default only appears where the user hasn't set an Elementor value, and not
  until **Elementor → Tools → Regenerate Files & Data** + a hard refresh. Tell the user this
  instead of assuming the change is broken.
- **The document a widget lives in is NOT the post being viewed — never use
  `get_queried_object_id()` to identify your own widget.** On a Theme Builder template the
  settings are in the template's `_elementor_data`; the queried object is whichever post the
  template is currently rendering (and on an archive it isn't a post at all). Any code that looks
  the widget up by the queried ID — an AJAX endpoint, a REST route, a cache key — finds nothing
  and fails on **every** theme template while working perfectly on ordinary pages. Ask Elementor
  instead, inside `render()`:
  ```php
  $doc = \Elementor\Plugin::$instance->documents->get_current();
  $id  = $doc ? (int) $doc->get_main_id() : (int) get_the_ID();
  ```
  `Frontend::get_builder_content()` calls `switch_to_document()` before rendering elements for
  exactly this reason. Use `get_main_id()`, not `get_id()`, so revisions and autosaves resolve to
  the parent.

---

## 9. Data safety — activation & uninstall

- **Uninstall must not delete the user's content.** Only delete pages/posts your plugin created
  (flagged with your own `_myplugin_auto_created` meta), that are **unedited** (no
  `_elementor_edit_mode`), and **empty**. Deleting "by stored ID" without provenance checks wipes
  pages the user later customised.
- **Activation is adopt-or-create, idempotently.** Never duplicate an existing plugin page, never
  overwrite user edits. Re-running activation (or an update) must converge, not multiply.
- **Rewrite-rule changes need a permalink flush** — a new action slug 404s until rules regenerate.
  Flush on activation, and on first load after an update; don't rely on the user visiting
  Settings → Permalinks.
- **Managed marker blocks (`# BEGIN MyPlugin` … `# END MyPlugin`) need line-anchored strip
  regexes.** If one marker name is a string **prefix** of another ("MyPlugin" vs "MyPlugin
  Hardening"), an unanchored regex matches the wrong block and swallows content between them.
  Anchor the marker lines (`[ \t]*\R` after BEGIN, `[ \t]*(?:\R|$)` after END) — and remove
  EVERY block you wrote on deactivation, not just the first.

---

## 10. Build, versioning & wp.org distribution

- **The distributable zip is an allowlist, not the repo.** Copy ONLY runtime dirs/files
  (`includes/ admin/ assets/ languages/` + `readme.txt uninstall.php <main>.php`) into a clean
  staging folder named exactly as the slug, then compress. Everything else (`.git/ .github/
  CLAUDE.md README.md *.html graphify-out/ wporg-assets/ Screenshots/ dotfiles`) must stay out, or
  Plugin Check flags stray/hidden/bad-name files. Verify the staged tree has no dotfiles before
  zipping.
- **A version bump touches every surface in lockstep:** main-file header `Version:` + the
  `MYPLUGIN_VERSION` define; `readme.txt` `Stable tag` + `== Changelog ==` + `== Upgrade Notice ==`
  (each notice < 300 chars); `README.md` changelog; and any landing `index.html` download
  button/title. After bumping, **grep all files for the old version** — a stuck `index.html` is
  easy to miss.
- **Only bump the version when front-end assets change.** A server-only fix (data/answer-key,
  back-end logic) needs no bump — bumping forces every visitor to re-download all assets for
  nothing. Conversely, CSS/JS changes are **invisible without** a version bump (it is the
  enqueue cache-buster).
- **Every stylesheet edit must hit BOTH `.css` and `.min.css`.** Production enqueues the minified
  file (the unminified one only under `SCRIPT_DEBUG`), so a change applied only to `.css` ships
  invisible. After editing, grep **both** files for the new selector, then bump the version
  constant.
- **Verify the zip's internal path separators are FORWARD slashes.** Windows PowerShell 5.1's
  `Compress-Archive` writes **backslash** entry names (`slug\main.php`); Linux treats `\` as an
  ordinary filename character, so WordPress extracts flat files with no plugin folder →
  **"plugin does not exist"** on upload. Pack with a .NET 5+ tool (pwsh 7+) and, after packing,
  assert an entry named `slug/main-file.php` (forward slash) exists. On a cloud-synced folder
  (OneDrive/Dropbox) a just-written zip can briefly vanish mid-sync — re-create and re-verify if
  it disappears.
- **Screenshots, banner, and icon are wp.org SVN `/assets/` files — NOT part of the plugin zip.**
  Name screenshots `screenshot-1.png`, `screenshot-2.png`, … each with a matching caption in
  `readme.txt == Screenshots ==`. Banner (`banner-772x250` / `banner-1544x500`) and icon
  (`icon-128x128` / `icon-256x256`) are listing graphics, also assets-only. Re-check listing
  graphics after any **rename** — an old-brand banner survives unnoticed across releases.
- **Run Plugin Check (the wp.org reviewer's own tool) before every resubmission**, excluding any
  vendored-lib directory, until **your** code is 0 findings.
- **The directory runs on SVN, not git.** The release itself — checkout, copy the built tree into
  `trunk/`, `svn cp trunk tags/X.Y.Z`, commit, and manage `/assets/` — is a Subversion workflow.
  Full commands + a WordPress.org worked example are in the **`svn/`** sub-bundle (`svn/svn.md`).

---

## 11. Verification reality

- **`php -l` / `node --check` prove syntax only.** They do **not** catch typed-signature
  mismatches (a fatal — see §1), missing/ineffective controls, leaked filters, or any runtime
  behaviour. Stub-test against a **typed `\Elementor\Widget_Base`** (instantiate + run
  `register_controls()` via reflection + exercise `render()` for each user state) before shipping.
- **You usually cannot see visual bugs from static checks.** Logo size, button placement, a
  sliver-collapse, a colliding flex column, a `var()` that resolved to nothing — all pass lint and
  "look fine" in code. Verify on a **live render** (the WP MCP bridge / a browser-driven preview),
  inspecting **computed styles**, not just screenshots.
- **Never assert "my query matches the archive" against `$wp_query->posts`.** That is the
  archive's *first page*, so a widget asked for 100 per page legitimately returns more and the
  assertion fails on correct code — which then sends you refactoring a query that was fine.
  Compare against `found_posts`, or re-run the archive's own `query_vars` unpaged.
- **Style-tab controls do not exist outside the editor unless you ask for them.** Elementor skips
  registering them for performance, so a harness that inspects `get_controls()` from WP-CLI reports
  **every Typography / Colour / Border / Box-Shadow / alignment control as missing** — a wall of
  confident false failures against code that is completely fine. Call
  `\Elementor\Core\Frontend\Performance::set_use_style_controls( true )` before `get_controls()`
  (this is exactly what `Core\Files\CSS\Base::parse_content()` does before rendering CSS).
- **Group controls do not carry their selector on the control you would guess.** The
  `_typography_typography` and `_box_shadow_box_shadow_type` entries are the *type* switches and
  have no `selector`; the real one lives on `_box_shadow`, `_font_family`, `_font_size` and so on.
  Assert against a sub-control that actually has a selector, or the assertion is meaningless.
- **When a check fails, suspect the check before the artifact.** Mis-scoped regex, byte-slicing
  multibyte UTF-8 (`head -c`/`tail -c` cuts a Georgian/emoji char mid-byte → looks like
  corruption), or a stub that flags on the wrong condition produce confident **false alarms**.
  Verify the harness (paths, encodings, regex), then the file.

---

## 12. Re-rendering a widget out of band (AJAX / REST) and in loops

- **A background request has no page.** `get_the_ID()`, `$wp_query`, `is_singular()` and the
  permalink of "the current page" are all absent inside a REST or admin-ajax handler. Any query
  option built on them — *related to the current post*, *exclude the current post*, *current
  query* — silently produces a **different result set** on page 2 than on page 1, so the visitor
  gets repeats or gaps rather than an error. Pass the displayed post's ID explicitly, validate it
  (`get_post()` + published + not password-protected), `setup_postdata()` around the render, and
  restore afterwards.
- **Don't try to rebuild an archive's main query in a background request.** Query vars sent by the
  client are the caller defining their own query, which is the thing your endpoint exists to
  prevent; `WP::parse_request()` a second time fires `parse_request` again and other plugins
  hooked there will act on it. For a *current query* source, leave pagination as **real page
  links** — that is what Elementor's own Archive Posts widget does — and say so in a control
  description.
- **Links rendered inside an AJAX response are built from the AJAX request.** `add_query_arg()`
  and `remove_query_arg()` with no URL argument read `$_SERVER['REQUEST_URI']`, which during the
  call is `/wp-json/...`. Middle-clicking such a link hands the visitor raw JSON. Send the page's
  URL along and validate it with **`wp_validate_redirect( $url, '' )`** — core's own "is this URL
  ours" check — rather than trusting it or inventing a regex.
- **"No results" and "past the end" are different answers.** Rendering the empty state for any
  page with no posts means an append-mode request one page past the end drops *"no posts found"*
  underneath a grid full of results. Gate the empty state on page 1.
- **A Load More button must stop existing, not just stop working.** Guard on
  `$paged >= $max_num_pages` and render nothing — a button that survives its last page reads as a
  stray line of unstyled text under the grid, and pressing it fetches an empty page.

### Dynamic-tag values that render as CSS cannot be looped

- **Elementor deliberately keeps them out of the cached stylesheet.** A control that is both
  dynamic and CSS-producing — a container **background image** bound to the featured image is the
  usual case — is recorded in the CSS file's `dynamic_elements_ids` meta and *excluded* from
  `elementor-post-<id>.css`. It is supplied separately by
  `\Elementor\Core\DynamicTags\Dynamic_CSS`, resolved against whatever post is current.
- **That mechanism is once-per-request and single-selector, so it cannot serve a loop.**
  `Core\Files\CSS\Base::enqueue()` records the file handle in a **static** `$printed` array and
  returns early ever after; and the selector it emits
  (`.elementor-<template> .elementor-element-<id>`) is identical for every card, so even emitted
  N times the last post would win for all N. The symptom is exact and misleading: **the image
  shows in the editor and is simply absent on the front end**, because the editor regenerates CSS
  against the current post on every render.
- **Fix by scoping per item.** Give each card wrapper a per-post class, subclass `Dynamic_CSS`,
  override `get_element_unique_selector()` to prefix that class (it is what fills `{{WRAPPER}}`,
  so it covers every control), construct it with `new` (not `create()` — the files manager caches
  by class+args and would hand every card the first card's CSS), and `echo` the result inline.
  Elementor Pro's loop grid does the same thing with `.e-loop-item-<post_id>`.
- **Check `dynamic_elements_ids` is non-empty before doing any of this** — almost no template
  styles anything dynamically, and that check is what stops every card paying for a second CSS
  parse.
- **An image *widget* with a dynamic tag has none of these problems**, because it renders an
  `<img>` inline per post. If a user only needs a per-post picture, that is the cheap answer.

### Swiper inside an optimised site

- **Swiper's autoplay refuses to start while the carousel measures zero, and never retries.**
  `run()` opens with `if ( ! swiper.size ) { autoplay.running = false; autoplay.paused = false;
  return; }`. Nothing calls it again, so autoplay is dead for the life of the page. The
  fingerprint is `running === false` **and** `paused === false` — no other path clears both.
- **This is why "it works in the editor and is frozen on the live site" happens.** LiteSpeed /
  WP Rocket / Cloudflare APO defer or async the stylesheet that gives the container its width, so
  the measurement at init is 0; the editor has no such optimisation. It is a **race**, so it
  reproduces intermittently — do not conclude it is fixed because one reload autoplayed.
- **Re-arm rather than re-init.** Watch the root with a `ResizeObserver` (plus a short interval
  fallback, since a late stylesheet can change the width without a resize entry for that element);
  once `swiper.size` is non-zero, `swiper.update()` then `swiper.autoplay.start()`. Record any
  visitor-initiated pause on the instance first, so recovery never restarts what someone stopped.
- **Elementor's carousels ship no visible play/pause button.** That is a WCAG 2.2.2 failure, not a
  pattern to copy. Keep the control reachable by keyboard and hidden until focused — that gives
  the same clean look without removing the mechanism.
