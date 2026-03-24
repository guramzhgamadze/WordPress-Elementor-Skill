# Widget Pattern — Render a PHP Template File

> **When to use this file:** Load whenever building a widget whose markup lives in
> a separate PHP template file rather than inline in `render()`. Use this pattern
> when your widget's HTML is complex, needs to be overridable by child themes, or
> must be reused outside Elementor (e.g. shortcodes, REST responses).
>
> **Default choice:** If the inner layout needs to be visually designed by the user
> in Elementor, use `widget-elementor-template.md` instead. This file is for
> code-defined PHP markup only.

---

## When PHP template files make sense

| Situation | Recommended approach |
|---|---|
| User designs the inner layout visually | `widget-elementor-template.md` ✅ |
| Markup is fixed / developer-defined | **This file** ✅ |
| Template must be overridable by child themes | **This file** ✅ |
| Reusing the same markup in shortcodes or REST | **This file** ✅ |
| Complex conditional markup that would clutter `render()` | **This file** ✅ |

---

## File structure

```
my-plugin/
├── my-plugin.php
├── includes/
│   └── class-myplugin-widget.php
└── templates/
    └── widget-myplugin.php       ← the PHP template file
```

---

## Template resolution — overridable by child themes

```php
/**
 * Locate the widget template file.
 *
 * Resolution order (first found wins):
 *   1. Child theme:   {child-theme}/myplugin/widget-myplugin.php
 *   2. Parent theme:  {parent-theme}/myplugin/widget-myplugin.php
 *   3. Plugin:        {plugin}/templates/widget-myplugin.php
 *
 * This mirrors the WooCommerce template override convention.
 * Source: developer.wordpress.org/reference/functions/locate_template/
 *
 * @param string $template_name Filename relative to the plugin templates/ directory.
 * @return string Absolute path to the resolved template file.
 */
private function locate_template( string $template_name ): string {
    // ✅ locate_template() searches child theme first, then parent theme.
    // Pass $load = false to get the path only — we include it ourselves.
    $theme_file = locate_template( [
        'myplugin/' . $template_name,
    ] );

    if ( $theme_file ) {
        return $theme_file;
    }

    // Fallback to the plugin's own templates/ directory
    return plugin_dir_path( MYPLUGIN_FILE ) . 'templates/' . $template_name;
}
```

---

## register_controls() skeleton

```php
protected function register_controls(): void {

    // ── CONTENT ───────────────────────────────────────────────────────────────
    $this->start_controls_section( 'section_content', [
        'label' => esc_html__( 'Content', 'myplugin' ),
        'tab'   => \Elementor\Controls_Manager::TAB_CONTENT,
    ] );

    $this->add_control( 'title', [
        'label'       => esc_html__( 'Title', 'myplugin' ),
        'type'        => \Elementor\Controls_Manager::TEXT,
        'default'     => esc_html__( 'My Widget', 'myplugin' ),
        'label_block' => true,
        'dynamic'     => [ 'active' => true ],
    ] );

    $this->add_control( 'description', [
        'label'   => esc_html__( 'Description', 'myplugin' ),
        'type'    => \Elementor\Controls_Manager::TEXTAREA,
        'dynamic' => [ 'active' => true ],
    ] );

    $this->end_controls_section();
}
```

---

## has_widget_inner_wrapper() + is_dynamic_content() + render() skeleton

```php
public function has_widget_inner_wrapper(): bool {
    return false;
}

// ✅ Return false if the template output is the same for all users (enables caching).
// Return true if the template contains is_user_logged_in(), get_current_user_id(),
// or other per-user/per-request logic.
// Source: developers.elementor.com/docs/widgets/widget-output-caching/
protected function is_dynamic_content(): bool {
    return false;
}

protected function render(): void {
    $settings = $this->get_settings_for_display();

    // ✅ Resolve the template path — child theme overrides plugin default.
    $template = $this->locate_template( 'widget-myplugin.php' );

    if ( ! file_exists( $template ) ) {
        if ( \Elementor\Plugin::$instance->editor
             && \Elementor\Plugin::$instance->editor->is_edit_mode() ) {
            echo '<div style="padding:1rem;background:#f8d7da;color:#721c24;">'
                 . esc_html__( 'Template file not found: ', 'myplugin' )
                 . esc_html( basename( $template ) )
                 . '</div>';
        }
        return;
    }

    // ✅ Pass settings via $args to load_template() — avoids extract() and uses
    // the official WP API which sets up all globals ($post, $wp_query, $id, etc.)
    // inside the included file. Never use bare include/require — they skip WP global setup.
    // Source: developer.wordpress.org/reference/functions/load_template/
    //
    // Second param $load_once = false — each widget instance needs its own render pass.
    // $args is available inside the template as $args['settings'], $args['widget'].
    load_template( $template, false, [
        'settings' => $settings,
        'widget'   => $this,
    ] );
}
```

---

## PHP template file skeleton (`templates/widget-myplugin.php`)

```php
<?php
/**
 * Widget template: My Plugin Widget
 *
 * Available variables (passed via load_template $args):
 *   $args['settings'] (array)  — widget settings from get_settings_for_display()
 *   $args['widget']   (object) — the Widget_Base instance
 *
 * Override: copy to {theme}/myplugin/widget-myplugin.php
 */

defined( 'ABSPATH' ) || exit;

// ✅ Access via $args — load_template() does NOT extract $args into variables.
$settings = $args['settings'] ?? [];

$title       = $settings['title'] ?? '';
$description = $settings['description'] ?? '';

if ( empty( $title ) && empty( $description ) ) {
    return;
}
?>
<div class="myplugin-widget">
    <?php if ( $title ) : ?>
        <h3 class="myplugin-widget__title">
            <?php echo wp_kses_post( $title ); ?>
        </h3>
    <?php endif; ?>

    <?php if ( $description ) : ?>
        <div class="myplugin-widget__description">
            <?php echo wp_kses_post( $description ); ?>
        </div>
    <?php endif; ?>
</div>
```

> ⚠️ **Escaping in template files:**
> Always escape at the point of output inside the template file.
> `get_settings_for_display()` does NOT auto-escape TEXT or TEXTAREA values.
> Use `esc_html()` for plain text, `wp_kses_post()` for rich text that may
> contain links or formatting, `esc_url()` for URLs.

---

## content_template() skeleton

```php
protected function content_template(): void {
    ?>
    <#
    var title       = settings.title       || '';
    var description = settings.description || '';
    if ( ! title && ! description ) { return; }
    #>
    <div class="myplugin-widget">
        <# if ( title ) { #>
            <h3 class="myplugin-widget__title">{{{ title }}}</h3>
        <# } #>
        <# if ( description ) { #>
            <div class="myplugin-widget__description">{{{ description }}}</div>
        <# } #>
    </div>
    <?php
}
```

> ✅ `content_template()` mirrors the PHP template's structure exactly so the
> editor preview matches the frontend output. Keep both in sync whenever the
> template markup changes.

---

## Making templates overridable — developer documentation comment

Add a comment to the top of your plugin's main file so third-party developers
know where to place overrides:

```php
/**
 * Template override:
 * To override widget templates, copy the file from:
 *   my-plugin/templates/widget-myplugin.php
 * to:
 *   {your-theme}/myplugin/widget-myplugin.php
 * Child theme overrides take precedence over parent theme overrides.
 */
```

---

> **Remove checklist:**
> - `locate_template()` helper → remove if template is not overridable by themes
> - `extract()` call → replace with explicit variable passing if preferred
> - `description` control → remove if widget has only one content field
> - Override documentation comment → update with your actual plugin/template paths
