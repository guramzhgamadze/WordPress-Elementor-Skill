# Debugging — WordPress & Elementor

> **When to read this file:** Diagnosing a bug, a white screen, a failed wp.org check, a CSS/JS
> change that "didn't come through," or an editor preview that broke. Work the layers in order:
> **static analysis first** (cheapest — catches it before it runs), then **runtime logging**,
> then **Elementor-specific** state, then **browser** (JS/CSS). The symptom→cause table at the
> bottom maps real failures to where to look.

> ⚠️ **`php -l` / `node --check` prove SYNTAX ONLY.** They do **not** catch a typed-signature
> mismatch (a fatal — see `field-notes.md` §1), a missing/ineffective control, a leaked filter,
> escaping issues, or any runtime behaviour. Passing lint means nothing about correctness.

---

## 1. Static analysis (catch it before it runs)

Three complementary tools — each catches a different class of bug:

| Tool | Catches | Does NOT catch |
|---|---|---|
| **PHPCS + WPCS** | Coding standards, **security** (`WordPress.Security.*`: unescaped output, unsanitized input, missing nonces), naming/prefix, i18n | Type errors, logic bugs, runtime behaviour |
| **PHPStan** (WordPress) | **Type & logic** bugs — wrong arg types, impossible conditions, dead code, null misuse, undefined methods | Coding-style / escaping policy |
| **Plugin Check 2.0.0** | The **review-grade superset** — bundles PHPCS+WPCS plus repo/guideline/perf/a11y/i18n checks (what wp.org reviewers run) | Guarantees nothing about approval (human review is still mandatory) |

### PHPCS + WPCS — WordPress Coding Standards

PHPCS is the engine; **WPCS** (WordPress Coding Standards) is the ruleset that emits the
`WordPress.Security.*`, `WordPress.NamingConventions.*`, `WordPress.WP.I18n`, etc. codes.

```bash
# Install per-project (WPCS 3.x pulls PHPCSUtils + PHPCSExtra automatically;
# the composer-installer auto-registers installed_paths so phpcs finds WPCS):
composer require --dev \
  wp-coding-standards/wpcs:"^3.1" \
  dealerdirect/phpcodesniffer-composer-installer:"^1.0"

# Run (uses phpcs.xml.dist below); phpcbf auto-fixes what is mechanically fixable:
./vendor/bin/phpcs
./vendor/bin/phpcbf
```

Drive it with a committed **`phpcs.xml.dist`** ruleset — this is where you set your text domain,
prefixes, and minimum WP version so the security/naming/i18n sniffs actually apply:

```xml
<?xml version="1.0"?>
<ruleset name="MyPlugin">
    <description>WPCS ruleset for my plugin.</description>

    <file>.</file>
    <exclude-pattern>*/vendor/*</exclude-pattern>
    <exclude-pattern>*/node_modules/*</exclude-pattern>
    <exclude-pattern>*/includes/lib/*</exclude-pattern>   <!-- vendored libs: don't lint, disclose -->

    <arg value="ps"/>                          <!-- p = progress, s = show sniff codes -->
    <arg name="extensions" value="php"/>
    <arg name="parallel" value="8"/>

    <rule ref="WordPress"/>                     <!-- WordPress-Core + Extra + Docs + security -->

    <config name="minimum_wp_version" value="7.0"/>

    <!-- i18n: every text-domain must match the slug (else WordPress.WP.I18n flags it) -->
    <rule ref="WordPress.WP.I18n">
        <properties>
            <property name="text_domain" type="array">
                <element value="myplugin"/>
            </property>
        </properties>
    </rule>

    <!-- Prefix sniff: declare your prefixes or every global symbol is flagged -->
    <rule ref="WordPress.NamingConventions.PrefixAllGlobals">
        <properties>
            <property name="prefixes" type="array">
                <element value="myplugin"/>
                <element value="MyPlugin"/>
                <element value="MYPLUGIN"/>
            </property>
        </properties>
    </rule>
</ruleset>
```

> ⚠️ **PHPCS `--standard=WordPress` surfaces MORE than Plugin Check does.** PC runs a curated
> **review** ruleset; the full `WordPress` standard adds `WordPress-Extra` style nags PC's review
> ruleset doesn't enforce. So: use **Plugin Check** to answer *"will this pass review?"*, and
> **PHPCS/WPCS** for general code-quality and the fast, file-level security sniffs. WPCS is also
> **bundled inside Plugin Check** (`plugin-check/vendor/…`) — that bundled copy is what emits PC's
> `WordPress.Security.*` codes. To run phpcs against just the security sniffs quickly:
> `phpcs --standard=WordPress --sniffs=WordPress.Security.EscapeOutput,WordPress.Security.NonceVerification,WordPress.Security.ValidatedSanitizedInput path/`.

### PHPStan — type & logic bugs PHPCS can't see

```bash
composer require --dev phpstan/phpstan szepeviktor/phpstan-wordpress
```
```neon
# phpstan.neon
includes:
    - vendor/szepeviktor/phpstan-wordpress/extension.neon
parameters:
    level: 5                      # 5 is a sane WordPress baseline; raise gradually
    paths:
        - includes
    # For Elementor/WooCommerce types, add their stubs so PHPStan stops reporting
    # "unknown class \Elementor\Widget_Base":
    # scanFiles:
    #     - vendor/php-stubs/woocommerce-stubs/woocommerce-stubs.php
```
```bash
./vendor/bin/phpstan analyse
# Lock in unavoidable false positives (e.g. dynamic Elementor internals) as a baseline:
./vendor/bin/phpstan analyse --generate-baseline
```
PHPStan catches things lint and PHPCS miss — e.g. an **untyped override of a typed Elementor
method** (the white-screen fatal), a dead `is_string()` guard after `sanitize_text_field()`
(`function.alreadyNarrowedType`), or a wrong return type. **Pair it with a typed
`\Elementor\Widget_Base` stub** that instantiates the widget so signature mismatches fatal at
class-declaration time during testing, not on the live site (see `field-notes.md` §1, §11).

### Plugin Check 2.0.0 — the reviewer's tool

Full categories and usage are in **`wp-org-guidelines.md`**. For debugging:
```bash
wp plugin check <your-plugin-slug> --format=json --exclude-directories=includes/lib
```
Static checks run by default; **runtime** checks need `--require .../cli.php`. Run it until your
own code is **0 findings** before every submission *and* every update (PC auto-scans updates since
Oct 2025). JSON output is grouped under `FILE:` headers — parse JSON, not CSV (commas in messages
break naive splitting).

---

## 2. Runtime WordPress debugging

Turn on debugging in **`wp-config.php`** (a dev/staging site — never `WP_DEBUG_DISPLAY` on prod):

```php
define( 'WP_DEBUG',         true );   // enable error reporting
define( 'WP_DEBUG_LOG',     true );   // write to wp-content/debug.log (or a path string)
define( 'WP_DEBUG_DISPLAY', false );  // keep errors OUT of the page; read the log instead
define( 'SCRIPT_DEBUG',     true );   // load unminified core/Elementor CSS & JS
define( 'SAVEQUERIES',      true );   // record DB queries (Query Monitor / debugging only)
@ini_set( 'display_errors', 0 );
```

- **`wp-content/debug.log`** is the first place to look for a white screen or a 500 — the fatal's
  message, file, and line are logged there.
- **Query Monitor** (free plugin) is the single most useful runtime tool: PHP errors, slow/duplicate
  queries, hooks fired, enqueued scripts/styles, REST calls, and which template/callback ran.
- **Your own logging**, guarded so it never leaks on production:
  ```php
  if ( defined( 'WP_DEBUG' ) && WP_DEBUG ) {
      error_log( 'myplugin: ' . wp_json_encode( $debug_data ) );
  }
  ```
  (Raw error text can contain API keys / user data — never log unguarded. See `php-standards.md`.)

**White-screen-of-death (WSOD):** it's a PHP **fatal**. Enable `WP_DEBUG_LOG`, reload, read the
last lines of `debug.log` — you get the exact file:line. The classic Elementor cause is an
**untyped override** of `has_widget_inner_wrapper()` / `is_dynamic_content()` / `get_categories()`
(runs on `wp_enqueue_scripts`, so it kills *every* page). `php -l` cannot catch it — PHPStan + a
typed stub can.

---

## 3. Elementor-specific debugging

- **Safe Mode** (Elementor → Tools → General → *Enable Safe Mode*) loads the editor with only
  Elementor active — isolates whether a bug is yours or a theme/third-party conflict.
- **Regenerate Files & Data** (Elementor → Tools → General) — **the fix when a CSS change "didn't
  come through."** Elementor caches **per-page CSS**; new `--var` defaults or selector changes
  won't appear until you regenerate + hard-refresh. Tell the user this instead of assuming it's
  broken.
- **System Info** (Elementor → System Info) — versions, server config, active experiments; paste it
  when diagnosing compatibility.
- **Element cache freezes** = the "An error occurred" form / dead-redirect class of bug: a widget
  without `is_dynamic_content(): true` gets its HTML cached, freezing per-request nonces and
  `?redirect_to=`. Symptom looks like a logic bug; fix is the cache flag + clear Elementor cache.
- **Editor preview blank / crashing** = a JS error in `content_template()` — a leaked
  `add_filter()` from `render()` (it re-runs on every control change), an "undefined index," or a
  triple-braced value. Open the **browser console while in the editor**; check that
  `content_template()` mirrors `render()`.
- **`?elementor-preview` / Safe Mode** in the URL force-load the editor preview context — useful
  for reproducing editor-only behaviour from your URL-rewrite/context-detection code.

---

## 4. Frontend / JS / CSS debugging

- **Browser console** — JS errors, and your own `console.error()` from AJAX handlers. An Elementor
  JS handler that "never runs" on an AJAX-loaded widget (popup, loop item) is almost always
  bound with `window.addEventListener('elementor/frontend/init', …)` (silently missed on
  Elementor < 3.5) or shipped with `strategy:'defer'` — use `jQuery(window).on(...)` and no defer
  (see `js-css-standards.md`).
- **Network tab** — failed REST/AJAX calls, 403 from a stale/expired nonce (refresh-and-retry
  pattern in `rest-api.md`), wrong response shape.
- **Inspect *computed* styles, not the stylesheet** — this is the only way to catch the bugs that
  "look fine" in code: a `var(--x)` that resolved to **nothing** (custom properties cascade
  parent→child only — see `field-notes.md` §4), a control selector that **lost on specificity** to
  theme/Elementor CSS, or a flex child overflowing because of a `min-width` floor. Screenshots
  alone miss all of these.
- **`SCRIPT_DEBUG`** serves Elementor's unminified frontend JS so stack traces are readable.

---

## 5. Symptom → likely cause → where to look

| Symptom | Likely cause | First check |
|---|---|---|
| **White screen on every page** | PHP fatal — often an untyped override of a typed Elementor method, or a parse error in an included file | `wp-content/debug.log` (exact file:line); then PHPStan + typed stub |
| **Form shows "An error occurred" / redirect is dead** | Elementor element-cache froze a per-request nonce / `redirect_to` | Set `is_dynamic_content(): true`; clear Elementor cache |
| **CSS change "didn't come through"** | Per-page CSS cache, or a `var()` resolving to nothing | **Regenerate Files & Data** + hard refresh; inspect **computed** styles for the empty var |
| **Editor preview blank / broken** | `content_template()` JS error (triple-brace, leaked filter, undefined index) | Browser console **in the editor**; diff `content_template()` vs `render()` |
| **Widget renders as "widget not found" after an update** | `get_name()` was changed (stored in `_elementor_data`) | Restore the old name, or migrate the stored JSON tokens (`field-notes.md` §1) |
| **JS handler never fires on a popup / loop item** | Bound init with `addEventListener` (not jQuery) or used `strategy:defer` | `jQuery(window).on('elementor/frontend/init', …)`, no defer |
| **Giant logo / broken layout when embedding an app** | Theme/Elementor CSS out-specifies single-class selectors | Boost base specificity (doubled class); inspect computed styles (`field-notes.md` §5) |
| **wp.org review / Plugin Check flags `EscapeOutput`** | `echo $var` / `echo $this->method()` not escaped at output | Escape at the point of output; make methods echo literals + `esc_*` (`field-notes.md` §3) |
| **Slow admin/front page** | N+1 queries / unbounded `WP_Query` | Query Monitor → Queries by component; add `no_found_rows`, scope `post_type` (`performance.md`) |

---

## 6. Verification ≠ debugging

A green lint/test run is not proof of correctness — see `field-notes.md` §11. Two reminders that
prevent wasted hours:
- **You usually can't see visual bugs from static checks.** Verify on a **live render** (the WP
  MCP bridge / a browser-driven preview), inspecting computed styles — not screenshots alone.
- **When a check fails, suspect the check before the artifact.** Mis-scoped regex or byte-slicing
  multibyte UTF-8 (`head -c`/`tail -c` cutting a character mid-byte) produces confident false
  alarms. Verify the harness (paths, encodings, regex), then the file.
