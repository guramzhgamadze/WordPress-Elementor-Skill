# Widget Pattern — Render a Saved Elementor Template

> **When to use this file:** Load whenever building a widget whose output is driven by a
> saved Elementor template (a post of type `elementor_library`). Common use cases:
> card grids, hero sections, modal popups, and any widget where the inner layout is
> designed visually in the Elementor editor rather than hard-coded in PHP.
>
> **API used:** `\Elementor\Plugin::$instance->frontend->get_builder_content_for_display()`
> Source: `elementor/includes/frontend.php` — Elementor GitHub (main branch)
> Code reference: `code.elementor.com/methods/elementor-frontend-get_builder_content_for_display/`

---

## How it works

`get_builder_content_for_display( int $post_id, bool $with_css = false ): string`

- Renders all Elementor elements saved in a post and returns the full HTML string.
- Automatically switches the global `$post` context to `$post_id` so Dynamic Tags
  (Post Title, Post Meta, etc.) pull data from the correct post.
- Restores the original `$post` context and edit-mode state afterwards.
- Returns `''` if the post does not exist or is not built with Elementor.
- **Recursion guard:** returns an error notice if `$post_id === get_the_ID()` (same
  template trying to render itself).

### `$with_css` parameter

| Value | Behaviour |
|---|---|
| `false` (default) | CSS is enqueued to the page's `<head>` via `wp_enqueue_style` — correct for standard frontend use. |
| `true` | CSS is inlined inside the returned HTML string — use for AJAX responses, email, or PDF export where `<head>` is unavailable. |

---

## register_controls() skeleton

```php
protected function register_controls(): void {

    $this->start_controls_section( 'section_template', [
        'label' => esc_html__( 'Template', 'myplugin' ),
        'tab'   => \Elementor\Controls_Manager::TAB_CONTENT,
    ] );

    // ✅ SELECT2 control populated with all published elementor_library posts.
    // 'select2' type with 'object_type' => 'elementor_library' is the
    // standard pattern used by Elementor Pro widgets internally.
    $this->add_control( 'template_id', [
        'label'       => esc_html__( 'Choose Template', 'myplugin' ),
        'type'        => \Elementor\Controls_Manager::SELECT2,
        'label_block' => true,
        'options'     => $this->get_elementor_templates(),
        'description' => esc_html__( 'Select a saved Elementor template from the library.', 'myplugin' ),
    ] );

    $this->end_controls_section();
}

/**
 * Returns an associative array of [ post_id => title ] for all published
 * posts in the `elementor_library` post type.
 *
 * ✅ Called only in register_controls() (admin/editor context) — safe to
 * run a get_posts() query here; it does NOT run on every frontend page load.
 *
 * @return array<string, string>
 */
private function get_elementor_templates(): array {
    $templates = get_posts( [
        'post_type'      => 'elementor_library',
        'post_status'    => 'publish',
        'posts_per_page' => -1,
        'orderby'        => 'title',
        'order'          => 'ASC',
        // ✅ no_found_rows: we never need pagination here
        'no_found_rows'  => true,
    ] );

    if ( empty( $templates ) ) {
        return [];
    }

    $options = [ '' => esc_html__( '— Select —', 'myplugin' ) ];
    foreach ( $templates as $tpl ) {
        // ✅ Cast to string — PHP array keys from WP_Post->ID are integers.
        // SELECT2 control compares option values as strings; an integer key
        // causes a type mismatch and the saved value never matches the option.
        $options[ (string) $tpl->ID ] = $tpl->post_title;
    }
    return $options;
}
```

---

## Required widget methods

```php
public function has_widget_inner_wrapper(): bool {
    return false;
}

// ✅ MUST return true — the rendered template may contain Dynamic Tags
// (post title, custom fields, etc.) which change per user/context.
// Caching a template render would freeze dynamic values.
// Source: developers.elementor.com/docs/widgets/widget-output-caching/
protected function is_dynamic_content(): bool {
    return true;
}
```

---

## render() skeleton

```php
protected function render(): void {
    $settings    = $this->get_settings_for_display();
    $template_id = (int) $settings['template_id'];

    if ( empty( $template_id ) ) {
        // ✅ Show a placeholder only inside the editor, not on the frontend.
        if ( \Elementor\Plugin::$instance->editor && \Elementor\Plugin::$instance->editor->is_edit_mode() ) {
            echo '<div style="padding:20px;border:1px dashed #ccc;text-align:center;">'
                . esc_html__( 'Please select a template.', 'myplugin' )
                . '</div>';
        }
        return;
    }

    // ✅ Verify the post exists and is published before rendering.
    // get_builder_content_for_display() returns '' for unpublished posts but
    // does not distinguish "not found" from "empty template" — check first.
    $post = get_post( $template_id );
    if ( ! $post || 'publish' !== $post->post_status ) {
        return;
    }

    // ✅ $with_css = false — CSS is enqueued to <head> automatically.
    // Pass true only for AJAX / email / PDF contexts where <head> is unavailable.
    // Source: elementor/includes/frontend.php — get_builder_content_for_display()
    $content = \Elementor\Plugin::$instance->frontend->get_builder_content_for_display( $template_id, false );

    if ( empty( $content ) ) {
        return;
    }

    // ✅ wp_kses_post would strip Elementor's inline scripts and data attributes.
    // The content is generated by Elementor's own renderer — treat it as trusted.
    // phpcs:ignore WordPress.Security.EscapeOutput.OutputNotEscaped
    echo $content;
}
```

---

## content_template() skeleton

```php
protected function content_template(): void {
    ?>
    <#
    if ( ! settings.template_id ) {
    #>
        <div style="padding:20px;border:1px dashed #ccc;text-align:center;">
            <?php echo esc_html__( 'Please select a template.', 'myplugin' ); ?>
        </div>
    <#
    } else {
    #>
        <div style="padding:20px;border:1px dashed #b3d4e8;text-align:center;background:#f0f8ff;">
            <?php echo esc_html__( 'Template preview only available on the frontend.', 'myplugin' ); ?>
            <br><small><?php echo esc_html__( 'Template ID:', 'myplugin' ); ?> {{ settings.template_id }}</small>
        </div>
    <#
    }
    #>
    <?php
}
```

> **Why no live preview?** `get_builder_content_for_display()` is a PHP method —
> it cannot run in the JS `content_template()`. This is expected and correct;
> Elementor's own "Template" widget uses the same placeholder pattern.

---

## Critical gotchas

**1. Recursion — widget inside the template it renders**
`get_builder_content_for_display()` has a built-in recursion guard: if the widget lives
inside the same template it tries to render, it returns an error notice instead of
causing infinite recursion. No extra code needed, but communicate this constraint clearly
to template authors.

**2. Dynamic Tags context**
`get_builder_content_for_display()` does **NOT** call `switch_to_post()` or change the
global `$post`. Dynamic Tags inside the rendered template read meta from the **current
page's post** — the one in the global `$post` at the time of the call — NOT from the
template post itself. This is documented behaviour confirmed in Elementor GitHub issues
(github.com/elementor/elementor/issues/31600, github.com/elementor/elementor/issues/13751).

If the template contains Dynamic Tags that should read from a **different** post (e.g.
a CPT entry), set up the post context explicitly before calling the method:

```php
// Set up context so Dynamic Tags inside the template read the CURRENT page's post:
global $post;
$original_post = $post;
$post = get_post( get_the_ID() );
setup_postdata( $post );

$content = \Elementor\Plugin::$instance->frontend
    ->get_builder_content_for_display( $template_id, false );

wp_reset_postdata();
// Note: get_builder_content_for_display() calls restore_current_post() internally,
// so $post is restored by it. The manual setup above is only needed when you need
// Dynamic Tags to read the CURRENT page post, not the template's own post.
```

**3. CSS enqueue timing**
CSS for the rendered template is enqueued inside the function call. If you call
`get_builder_content_for_display()` after `wp_head` has already fired (e.g. inside
a shortcode or late hook), the styles are enqueued too late and will be missing from
the page. This is a known Elementor limitation. Workaround: use `$with_css = true`
to inline the CSS inside the returned HTML string.

**4. `is_dynamic_content()` must return `true`**
Elementor's output cache would freeze the rendered template HTML on first load. Any
Dynamic Tags or conditional content inside the template would always show the cached
version. Always return `true` when using `get_builder_content_for_display()`.

**5. Select2 options populated once at registration**
`get_elementor_templates()` runs only in `register_controls()` (admin/editor context).
If a new template is added, the widget panel must be refreshed to see it. This is the
standard Elementor pattern — no fix needed, but document it for editors.

---

> **Remove checklist:**
> - The `get_elementor_templates()` helper → replace with a hardcoded ID if the
>   template is fixed and not user-selectable.
> - The editor placeholder in `content_template()` → keep it; it's the expected UX.
> - The `post_status` check in `render()` → keep it; prevents rendering draft templates.
