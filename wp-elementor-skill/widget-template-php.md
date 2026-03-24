# Widget Pattern — Render a PHP Template File

> **When to use this file:** Load whenever building a widget whose markup lives in a
> separate `.php` template file rather than being written inline in `render()`.
> Common use cases: complex card layouts, CPT single views, reusable partials,
> WooCommerce-style overrideable templates, or any widget needing theme/child-theme
> template overrides.
>
> **API used:**
> - `get_template_part()` — WP core, `developer.wordpress.org/reference/functions/get_template_part/`
> - `locate_template()` — WP core, `developer.wordpress.org/reference/functions/locate_template/`
> - `load_template()` — WP core, `developer.wordpress.org/reference/functions/load_template/`

---

## Three resolution strategies

Choose the strategy that fits your plugin's architecture:

| Strategy | Template lookup order | Override support | Use when |
|---|---|---|---|
| **A — Plugin-only** | `plugin/templates/` only | ❌ No theme override | Templates are internal/non-public |
| **B — Theme-overrideable** | Child theme → Parent theme → Plugin fallback | ✅ Full override | Distributing a public plugin |
| **C — `get_template_part()`** | Active theme directory only | ✅ Theme-only | Widget lives inside a theme, not a plugin |

---

## Strategy A — Plugin-only template file

```php
protected function render(): void {
    $settings = $this->get_settings_for_display();

    $template = plugin_dir_path( __FILE__ ) . 'templates/widget-mywidget.php';

    // ✅ Verify the file exists before including — avoids fatal errors.
    if ( ! file_exists( $template ) ) {
        return;
    }

    // ✅ Pass $settings to the template via a local variable.
    // Do NOT use extract($settings) — pollutes scope with untrusted keys.
    // The template accesses data as $settings['key'] or $args['settings']['key'].
    $args = [
        'settings' => $settings,
        'widget'   => $this,
    ];

    // ✅ load_template() sets up WP globals ($wp_query, $post, etc.)
    // inside the included file — same environment as standard WP templates.
    // Source: developer.wordpress.org/reference/functions/load_template/
    //
    // Second param $load_once = false — allows same partial to be included
    // multiple times (e.g. inside a repeater loop). Use true for singletons.
    load_template( $template, false, $args );
}
```

**Template file** (`plugin/templates/widget-mywidget.php`):

```php
<?php
defined( 'ABSPATH' ) || exit;

/**
 * Available variables (passed via load_template $args):
 *
 * @var array  $args['settings']  All widget settings from get_settings_for_display().
 * @var object $args['widget']    The widget instance (\Elementor\Widget_Base).
 */
$settings = $args['settings'] ?? [];
$widget   = $args['widget']   ?? null;

if ( empty( $settings['title'] ) ) {
    return;
}
?>
<div class="myplugin-widget">
    <h2 class="myplugin-widget__title"><?php echo esc_html( $settings['title'] ); ?></h2>
    <?php if ( ! empty( $settings['description'] ) ) : ?>
        <div class="myplugin-widget__desc"><?php echo wp_kses_post( $settings['description'] ); ?></div>
    <?php endif; ?>
</div>
```

---

## Strategy B — Theme-overrideable template (plugin with public API)

This pattern lets child/parent themes override the widget's template by placing a file
at the same relative path inside their theme directory. Used by WooCommerce, Easy
Digital Downloads, and similar plugins.

```php
protected function render(): void {
    $settings = $this->get_settings_for_display();

    // ✅ locate_template() searches:
    //   1. Child theme directory
    //   2. Parent theme directory
    //   3. Returns '' if not found in either
    // Source: developer.wordpress.org/reference/functions/locate_template/
    $theme_template = locate_template( 'myplugin/widget-mywidget.php' );

    if ( $theme_template ) {
        // Theme has overridden the template — use it.
        $template = $theme_template;
    } else {
        // Fall back to the plugin's bundled template.
        $template = plugin_dir_path( __FILE__ ) . 'templates/widget-mywidget.php';
    }

    if ( ! file_exists( $template ) ) {
        return;
    }

    $args = [
        'settings' => $settings,
        'widget'   => $this,
    ];

    load_template( $template, false, $args );
}
```

**Applying a filter for full extensibility:**

```php
protected function render(): void {
    $settings = $this->get_settings_for_display();

    $default_template = plugin_dir_path( __FILE__ ) . 'templates/widget-mywidget.php';

    // ✅ Allow plugins/themes to completely replace the template path via filter.
    $template = apply_filters(
        'myplugin_widget_mywidget_template',
        locate_template( 'myplugin/widget-mywidget.php' ) ?: $default_template,
        $settings
    );

    if ( ! file_exists( $template ) ) {
        return;
    }

    load_template( $template, false, [ 'settings' => $settings, 'widget' => $this ] );
}
```

---

## Strategy C — `get_template_part()` (theme-resident widget)

Use this only when the widget lives inside a **theme** (not a plugin). `get_template_part()`
only searches the active theme and its parent — it has no plugin fallback.

```php
protected function render(): void {
    $settings = $this->get_settings_for_display();

    // ✅ WP 5.5+ $args parameter passes variables to the template.
    // Inside the template, variables are available as $args['key'].
    // Source: developer.wordpress.org/reference/functions/get_template_part/
    get_template_part(
        'template-parts/widgets/mywidget',  // slug: looks for template-parts/widgets/mywidget.php
        null,                               // name: if set, also looks for mywidget-{name}.php
        [
            'settings' => $settings,
            'widget'   => $this,
        ]
    );
}
```

**Template file** (`theme/template-parts/widgets/mywidget.php`):

```php
<?php
defined( 'ABSPATH' ) || exit;

$settings = $args['settings'] ?? [];
$widget   = $args['widget']   ?? null;
?>
<div class="myplugin-widget">
    <h2><?php echo esc_html( $settings['title'] ?? '' ); ?></h2>
</div>
```

---

## Required widget methods

```php
public function has_widget_inner_wrapper(): bool {
    return false;
}

// ✅ Return false if the template file renders static content identical
// for all users — enables Elementor output caching.
// Return true if the template uses get_the_ID(), is_user_logged_in(),
// current_user_can(), or any per-user/per-session logic.
// Source: developers.elementor.com/docs/widgets/widget-output-caching/
protected function is_dynamic_content(): bool {
    return false; // change to true if template uses dynamic/user-specific data
}
```

---

## content_template() skeleton

PHP template files cannot run in the JS `content_template()`. Use a representative
placeholder that mirrors the visual structure:

```php
protected function content_template(): void {
    ?>
    <#
    if ( ! settings.title ) { return; }
    #>
    <div class="myplugin-widget">
        <h2 class="myplugin-widget__title">{{{ settings.title }}}</h2>
        <# if ( settings.description ) { #>
            <div class="myplugin-widget__desc">{{{ settings.description }}}</div>
        <# } #>
    </div>
    <?php
}
```

> **Note:** If the PHP template file contains complex logic that cannot be replicated
> in JS, it is acceptable for `content_template()` to output a simplified preview.
> Elementor's own Post Content widget uses this approach.

---

## Outputting the rendered template as a string (for hooks/filters)

When you need the template output as a string rather than echoing it directly —
for example to pass it to a filter or cache it:

```php
private function get_template_html( array $settings ): string {
    ob_start();

    $template = plugin_dir_path( __FILE__ ) . 'templates/widget-mywidget.php';

    if ( file_exists( $template ) ) {
        load_template( $template, false, [ 'settings' => $settings, 'widget' => $this ] );
    }

    return ob_get_clean() ?: '';
}

protected function render(): void {
    $settings = $this->get_settings_for_display();
    $html     = $this->get_template_html( $settings );

    if ( empty( $html ) ) {
        return;
    }

    // ✅ The HTML is generated by our own template — mark with phpcs ignore.
    // phpcs:ignore WordPress.Security.EscapeOutput.OutputNotEscaped
    echo $html;
}
```

---

## Critical gotchas

**1. Never use `include` or `require` directly**
`load_template()` sets up all WordPress globals (`$post`, `$wp_query`, `$id`, etc.)
inside the included file, exactly as WordPress does for standard templates. A bare
`include` skips this setup, causing WP template tags to fail or return wrong data.

**2. `$args` key — not extracted**
`load_template()` does NOT extract `$args` into variables. Inside the template,
all passed values live in `$args['key']`, not `$key` directly. Document this
in every template file header.

**3. `$load_once = false` for repeated partials**
If the same template is used in a loop (e.g. a posts grid widget rendering each card),
pass `false` as the second parameter to `load_template()`. The default `true` uses
`require_once`, so only the first card would render.

**4. Path traversal — never pass user input as template name**
`locate_template()` does not sanitise template names. Never interpolate control values
directly into template paths. Use a fixed map instead:

```php
// ❌ NEVER do this — path traversal attack vector
$template = locate_template( $settings['template_name'] . '.php' );

// ✅ Use a fixed allowlist map
$allowed = [
    'style-a' => 'myplugin/widget-mywidget-a.php',
    'style-b' => 'myplugin/widget-mywidget-b.php',
];
$tpl_slug = $allowed[ $settings['template_style'] ] ?? $allowed['style-a'];
$template  = locate_template( $tpl_slug ) ?: plugin_dir_path( __FILE__ ) . 'templates/' . basename( $tpl_slug );
```

**5. Child theme wins over parent theme**
`locate_template()` checks the child theme first. If a child theme user adds
`myplugin/widget-mywidget.php` to their theme, it takes precedence over both
the parent theme and plugin fallback. Document this in your plugin's README.

---

> **Remove checklist:**
> - Strategy B filter (`myplugin_widget_mywidget_template`) → remove for internal plugins.
> - `ob_start()` wrapper → only needed when returning HTML as a string.
> - `$args['widget']` → remove if the template does not need widget methods.
