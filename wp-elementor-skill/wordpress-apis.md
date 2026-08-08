# Common WordPress APIs — Options, Settings, Metadata, Capabilities, Cron, i18n

> **When to read this file:** Building an **admin settings page**, persisting plugin data,
> exposing meta to REST/Elementor, gating actions by capability, scheduling background work, or
> making a plugin translatable. These are the WordPress "Common APIs" the other files don't cover
> (REST → `rest-api.md`; transients/HTTP/sanitization → `php-standards.md`; CPT/AJAX →
> `scaffolding.md`). For exact, current function signatures, the canonical lookup is the **Code
> Reference**: developer.wordpress.org/reference/ — verify there before relying on memory.

---

## 1. Options API — storing plugin settings

```php
$opts = get_option( 'myplugin_options', [] );   // ALWAYS pass a default

// ✅ WP 6.6+: pass an explicit BOOLEAN $autoload — not the legacy 'yes'/'no' strings.
update_option( 'myplugin_options', $opts, true );       // true  → autoloaded on every page
update_option( 'myplugin_big_cache', $blob, false );    // false → NOT autoloaded (large/rare)
add_option(    'myplugin_flag', '1', '', false );        // 3rd arg ($deprecated) stays ''
delete_option( 'myplugin_options' );
```

- **Store one option as an array**, not a dozen scalar options — fewer `wp_options` rows and a
  single autoload entry. Sanitize the whole array on write (see the Settings API below).
- **`autoload` is a real performance lever.** Autoloaded options load on **every** request via the
  `alloptions` cache. Set `false` for anything large or rarely read (caches, logs, per-item blobs);
  bloated autoload is a common cause of slow sites. (WP 6.6 changed the default to "auto" so core
  can decide; be explicit anyway.) See `performance.md`.
- Site-wide on multisite: `get_site_option()` / `update_site_option()`.

---

## 2. Settings API — the review-safe admin settings page

The canonical pattern. `settings_fields()` emits the nonce, the **`sanitize_callback` is the single
trusted place to clean input**, and field renderers **escape on output** — together that satisfies
Plugin Check and wp.org review.

```php
// (a) Register the setting + sections + fields on admin_init.
add_action( 'admin_init', function () {

    register_setting( 'myplugin_group', 'myplugin_options', [
        'type'              => 'array',
        'sanitize_callback' => 'myplugin_sanitize_options',  // ← the only trusted cleaning point
        'default'           => [ 'api_key' => '', 'enabled' => false ],
        'show_in_rest'      => false,                          // true only with a defined schema
    ] );

    add_settings_section(
        'myplugin_main',
        esc_html__( 'Main Settings', 'myplugin' ),
        '__return_false',          // optional intro callback
        'myplugin'                 // page slug (matches do_settings_sections below)
    );

    add_settings_field(
        'myplugin_api_key',
        esc_html__( 'API Key', 'myplugin' ),
        'myplugin_field_api_key',
        'myplugin',
        'myplugin_main'
    );
} );

// (b) Sanitize callback — clean EVERY field; never trust $input.
function myplugin_sanitize_options( $input ): array {
    return [
        'api_key' => sanitize_text_field( $input['api_key'] ?? '' ),
        'enabled' => ! empty( $input['enabled'] ),
    ];
}

// (c) Field renderer — escape on output.
function myplugin_field_api_key(): void {
    $opts = get_option( 'myplugin_options', [] );
    printf(
        '<input type="text" name="myplugin_options[api_key]" value="%s" class="regular-text" autocomplete="off">',
        esc_attr( $opts['api_key'] ?? '' )
    );
}

// (d) The admin page — capability-gated, with the settings_fields() nonce.
add_action( 'admin_menu', function () {
    add_options_page(
        esc_html__( 'My Plugin', 'myplugin' ),   // <title>
        esc_html__( 'My Plugin', 'myplugin' ),   // menu label
        'manage_options',                         // capability
        'myplugin',                               // menu slug
        'myplugin_render_settings_page'
    );
} );

function myplugin_render_settings_page(): void {
    if ( ! current_user_can( 'manage_options' ) ) {
        return;
    }
    ?>
    <div class="wrap">
        <h1><?php echo esc_html( get_admin_page_title() ); ?></h1>
        <form action="options.php" method="post">
            <?php
            settings_fields( 'myplugin_group' );   // nonce + option_page + _wp_http_referer
            do_settings_sections( 'myplugin' );    // renders the sections + fields
            submit_button();
            ?>
        </form>
    </div>
    <?php
}
```

> ⚠️ **The `sanitize_callback` runs on EVERY `update_option()` for that option — including your
> own programmatic writes.** `register_setting()` installs a `sanitize_option_{$option}` filter
> that fires inside `update_option()` regardless of who calls it (whenever the registering code
> has loaded — any `is_admin()` request, including `admin-post.php`). Three rules from a real
> "activation silently reverted" bug:
> - The callback must be a **pure function of `$input`** — never "preserve the current stored
>   value" for a key, and beware side effects. A "preserving" sanitizer silently reverted the
>   plugin's own `update_option( …, true )` on every activation click.
> - **Runtime state does not belong in a registered settings array.** Flags your code toggles
>   (an "enabled" state, counters, timestamps) live in their **own unregistered option**; inject
>   them at read time if you want a single accessor.
> - **WP-CLI smoke tests miss this bug class**: under CLI, `is_admin()` is false → the settings
>   page never loads → `register_setting()` never ran → the filter is absent. To reproduce admin
>   conditions, `add_filter( 'sanitize_option_myplugin_options', 'myplugin_sanitize_options' )`
>   before testing option writes.

> **Secrets in settings:** never re-render an API secret into `value="…"` — show an empty field
> with a "leave blank to keep" sanitizer and store it encrypted (see `field-notes.md` §6).

---

## 3. Metadata API — `register_meta` (REST- and Elementor-aware)

```php
register_post_meta( 'myplugin_item', '_myplugin_subtitle', [
    'type'              => 'string',
    'single'            => true,
    'sanitize_callback' => 'sanitize_text_field',
    'auth_callback'     => function () { return current_user_can( 'edit_posts' ); },
    'show_in_rest'      => true,   // REST exposure + needed for Gutenberg / Elementor Dynamic Tags
] );

$subtitle = get_post_meta( $post_id, '_myplugin_subtitle', true );
update_post_meta( $post_id, '_myplugin_subtitle', sanitize_text_field( $subtitle ) );
```

- **Prefix meta keys.** A leading underscore (`_myplugin_*`) marks the meta "protected" — hidden
  from the default Custom Fields UI and not editable by users directly.
- **`show_in_rest` is required** for the value to drive an Elementor **Dynamic Tag** / Loop Grid or
  appear in the block editor. For arrays/objects pass a `show_in_rest` schema.
- `register_term_meta()` / `register_user_meta()` follow the same shape.

---

## 4. Roles & Capabilities — gate by capability, never by role

```php
// Authorisation check before any privileged action (nonces confirm ORIGIN, caps confirm PERMISSION):
if ( ! current_user_can( 'edit_post', $post_id ) ) {   // meta-cap with the object id where relevant
    wp_die( esc_html__( 'You are not allowed to do this.', 'myplugin' ) );
}

// ❌ Never test a ROLE: current_user_can( 'administrator' ) is wrong — roles aren't capabilities.
// ✅ Test a capability: 'manage_options', 'edit_posts', or a custom one.

// Custom capability — add on activation, remove on uninstall (NOT on every page load).
register_activation_hook( __FILE__, function () {
    foreach ( [ 'administrator', 'editor' ] as $role_name ) {
        get_role( $role_name )?->add_cap( 'myplugin_manage_items' );
    }
} );
// Then gate with: current_user_can( 'myplugin_manage_items' )
```

---

## 5. WP-Cron — scheduled tasks

```php
// Schedule on activation, guarding against a duplicate event.
register_activation_hook( __FILE__, function () {
    if ( ! wp_next_scheduled( 'myplugin_daily_task' ) ) {
        wp_schedule_event( time(), 'daily', 'myplugin_daily_task' );
    }
} );
add_action( 'myplugin_daily_task', 'myplugin_run_daily' );

// ✅ ALWAYS clear on deactivation — an orphaned event keeps firing forever.
register_deactivation_hook( __FILE__, function () {
    wp_clear_scheduled_hook( 'myplugin_daily_task' );
} );

// Custom interval (built-ins: hourly, twicedaily, daily, weekly).
add_filter( 'cron_schedules', function ( array $s ): array {
    $s['myplugin_5min'] = [ 'interval' => 5 * MINUTE_IN_SECONDS, 'display' => esc_html__( 'Every 5 Minutes', 'myplugin' ) ];
    return $s;
} );

// One-off job with an argument:
wp_schedule_single_event( time() + 10 * MINUTE_IN_SECONDS, 'myplugin_one_off', [ $item_id ] );
```

> ⚠️ **WP-Cron is NOT a real system cron — it is triggered by site traffic.** On a low-traffic
> site a "daily" task fires whenever the next visitor arrives, not at a fixed time. A *missed*
> event is queued and runs on the next load (not abandoned). For **reliable timing**, disable the
> traffic trigger and run a real server cron:
> ```php
> // wp-config.php
> define( 'DISABLE_WP_CRON', true );
> ```
> ```cron
> # crontab — every 5 minutes, headless:
> */5 * * * * cd /path/to/wp && wp cron event run --due-now > /dev/null 2>&1
> ```
> For **heavy, high-volume, or must-complete** background work (imports, bulk emails, queues),
> prefer **Action Scheduler** (the battle-tested queue bundled with WooCommerce) over raw WP-Cron —
> it persists jobs to the DB, retries on failure, and processes in batches.

---

## 6. Internationalization (i18n) — required for the directory

```php
// Header: "Text Domain: myplugin" — the text domain MUST EQUAL the plugin slug (Golden Rule #7).

// wp.org-HOSTED plugin: add NOTHING. Do not call load_plugin_textdomain().
// Translations arrive as language packs and load themselves. See the trap below.

// PRIVATE / self-hosted plugin only — this is the ONLY case where you need it:
add_action( 'init', function () {
    load_plugin_textdomain( 'myplugin', false, dirname( plugin_basename( __FILE__ ) ) . '/languages' );
} );
```

> 🚨 **The bundled-`.mo` trap — you cannot ship translations inside a wp.org-hosted plugin.**
> This looks like it should work and silently doesn't, so read the mechanism once:
>
> 1. WordPress resolves a plugin's translations through `WP_Textdomain_Registry`
>    (`wp-includes/class-wp-textdomain-registry.php`). Its `get_path_from_lang_dir()` scans
>    **`WP_LANG_DIR/plugins/`** — i.e. `wp-content/languages/plugins/` — and **nothing else.**
>    It never looks inside your plugin's own `/languages/` folder.
> 2. So a `.mo` you ship in `myplugin/languages/` is **never loaded**. Your `.pot` is fine there
>    (it is a template, not a catalogue), but the compiled files are dead weight.
> 3. The only thing that changes this is `load_plugin_textdomain()`, which registers a *custom
>    fallback path* on the registry — and **Plugin Check flags that call as discouraged**:
>    *"load_plugin_textdomain() has been discouraged since WordPress version 4.6. When your plugin
>    is hosted on WordPress.org, you no longer need to manually include this function call."*
> 4. Net effect for a hosted plugin: bundling is either **dead weight** (no call) or a **review
>    warning** (with the call). Neither is what you want.
>
> **The supported route:** ship only the `.pot`; translations live on
> [translate.wordpress.org](https://translate.wordpress.org/) and are delivered as **language
> packs**, which install to `WP_LANG_DIR/plugins/` — exactly where the loader already looks.
> Note the precedence is the right way round: a language pack always wins, and a custom path is
> only a fallback, so an author-supplied catalogue can never shadow the community translation.

### Getting a locale actually translated (the parts that bite)

- **90% or nothing.** wordpress.org generates the first language pack only once **≥90%** of that
  locale's strings are approved as **Current**. A "just the user-facing strings" catalogue
  typically lands near 35–40% and will **never** produce a pack. Translate the admin/settings/
  control-label strings too, or the work ships to nobody.
- **Anyone can import; only editors can approve.** Any wp.org user can use *Import Translations*
  on a plugin/theme project (the link appears at the bottom of a translation-set page **when
  logged in**). Imported strings land as **Waiting**. To set them Current you need **PTE** for
  that locale — request it on the front page of `make.wordpress.org/polyglots/` using the
  `#locale_code` tag (e.g. `#de_DE`) so the locale's GTEs are notified.
- **Build the catalogue from the GlotPress export, not your local `.pot`.** Download the locale's
  PO from the project and fill *that* in. Two reasons: the `msgid`s then match the project
  exactly, and the export carries the locale's real **`Plural-Forms`**. These disagree — a
  locally generated POT gave `nplurals=2` for Georgian while GlotPress uses `nplurals=1`.
- **Validate placeholders before importing.** A translation that loses or renames a `%s` / `%d` /
  `%1$s`, or drops an HTML tag, breaks output at runtime and passes every PHP lint. Diff the
  placeholder and tag sets between `msgid` and `msgstr` programmatically.
- **`.l10n.php` is the fast path.** WP 6.5+ prefers `myplugin-{locale}.l10n.php` over `.mo`;
  the registry checks for both. Generate with `wp i18n make-php`.
- **Interim use while you wait for a pack:** drop the compiled files into the site's own
  `wp-content/languages/plugins/`. That is the language-pack location, so the loader finds them
  with no plugin code and no Plugin Check warning, and a real pack later overwrites them.
- **Machine translation is not acceptable unreviewed.** Polyglots states MT *"without human
  review…will NOT be considered acceptable"*. Have a native speaker read it before approving.

> ⚠️ **WP 6.7+ — don't translate before `init`.** Calling `__()` / `_e()` / `esc_html__()` for your
> text domain **earlier than the `init` action** now triggers a `_doing_it_wrong` notice:
> *"Translation loading for the `myplugin` domain was triggered too early."* Don't translate at
> file-load, `plugins_loaded`, or in a class constructor that runs early. `register_post_type`
> labels, widget control labels, settings fields, etc. all run on/after `init` — those are fine.
> Debug the offender: hook `doing_it_wrong_run` and `debug_print_backtrace()` when the function is
> `_load_textdomain_just_in_time`.
> Source: make.wordpress.org/core/2024/10/21/i18n-improvements-6-7/

```php
// Functions — always use the ESCAPING variants at output:
esc_html__( 'Save', 'myplugin' );          esc_html_e( 'Save', 'myplugin' );
esc_attr__( 'Close', 'myplugin' );          esc_attr_e( 'Close', 'myplugin' );

// Plurals + placeholders (add a translator comment for context):
printf(
    /* translators: %s: number of items */
    esc_html( _n( '%s item', '%s items', $count, 'myplugin' ) ),
    esc_html( number_format_i18n( $count ) )
);
_x( 'Post', 'noun', 'myplugin' );   // disambiguate same word, different meaning

// JavaScript strings (wp.i18n / block editor):
wp_set_script_translations( 'myplugin-js', 'myplugin', plugin_dir_path( __FILE__ ) . 'languages' );
```

- **Never** interpolate a variable into a translation string (`__( "Hi $name" )`) — use a
  placeholder (`sprintf( __( 'Hi %s', 'myplugin' ), $name )`) so the string is extractable.
- Generate the template: **`wp i18n make-pot . languages/myplugin.pot`** (WP-CLI).
- Plugin Check's **Internationalization** category flags a wrong/missing text domain and
  variable-in-string usage.
