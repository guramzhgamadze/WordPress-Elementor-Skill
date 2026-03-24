# Widget Pattern — Render an Elementor Template

> **When to use this file:** Load whenever building a widget that renders a saved
> Elementor template (page, section, or reusable block) inside its `render()`.
> This is the **default / recommended approach** when a widget needs a user-designed
> inner layout. Use `widget-php-template.md` only when the markup is static PHP.
>
> **API source:** `\Elementor\Plugin::$instance->frontend->get_builder_content_for_display()`
> Source: `elementor/includes/frontend.php` — Elementor GitHub main branch.
> Note: `get_builder_content()` is the internal method; always call the public
> `get_builder_content_for_display()` wrapper instead.

---

## How It Works

The user picks a saved Elementor template (post type `elementor_library`) by ID
via a SELECT control in your widget. Your `render()` calls
`get_builder_content_for_display( $template_id, $with_css )` which:

1. Verifies the post exists and is built with Elementor.
2. Renders all Elementor elements in that template through the normal frontend pipeline.
3. Returns the full rendered HTML string — echo it directly.

> ⚠️ **`$with_css = true` vs `false`:**
> Both values always produce CSS — the difference is WHERE it is output.
> Source: `elementor/includes/frontend.php` (Elementor GitHub main branch).
> - `false` (default) — CSS is enqueued to the page `<head>` via `wp_enqueue_style`.
>   Correct for standard frontend use on any Elementor-built page.
> - `true` — CSS is printed inline as a `<style>` block directly above the HTML in the
>   returned string. Use for AJAX responses, shortcodes loaded after `wp_head`, or any
>   context where `<head>` is unavailable or has already fired.
>   Note: Elementor automatically forces `$with_css = true` on AJAX and Customizer requests.

---

## register_controls() skeleton

```php
protected function register_controls(): void {

    // ── CONTENT ───────────────────────────────────────────────────────────────
    $this->start_controls_section( 'section_template', [
        'label' => esc_html__( 'Template', 'myplugin' ),
        'tab'   => \Elementor\Controls_Manager::TAB_CONTENT,
    ] );

    // ✅ SELECT2 control — searchable dropdown for large lists of templates.
    // Using plain SELECT is fine for small lists but SELECT2 is the correct
    // control for elementor_library which can grow to hundreds of entries.
    // Cast each option key to string — WP_Post->ID is int; SELECT2 compares as strings.
    $this->add_control( 'template_id', [
        'label'       => esc_html__( 'Choose Template', 'myplugin' ),
        'type'        => \Elementor\Controls_Manager::SELECT2,
        'options'     => $this->get_template_options(),
        'default'     => '',
        'label_block' => true,
        // ✅ is_dynamic_content() must return true when template_id is used,
        // because the rendered output varies per template and must not be cached.
        // See is_dynamic_content() override in the render skeleton below.
    ] );

    $this->end_controls_section();

    // ── STYLE ─────────────────────────────────────────────────────────────────
    $this->start_controls_section( 'section_wrapper_style', [
        'label' => esc_html__( 'Wrapper', 'myplugin' ),
        'tab'   => \Elementor\Controls_Manager::TAB_STYLE,
    ] );

    $this->add_responsive_control( 'min_height', [
        'label'      => esc_html__( 'Min Height', 'myplugin' ),
        'type'       => \Elementor\Controls_Manager::SLIDER,
        'size_units' => [ 'px', 'vh', 'em', 'rem' ],
        'range'      => [ 'px' => [ 'min' => 0, 'max' => 1000 ] ],
        'selectors'  => [
            '{{WRAPPER}} .myplugin-template-wrapper' => 'min-height: {{SIZE}}{{UNIT}};',
        ],
    ] );

    $this->end_controls_section();
}
```

---

## Required helper method

```php
/**
 * Build SELECT options from published Elementor templates.
 *
 * Queries the elementor_library CPT directly — avoids loading the full
 * Template Library API which is Pro-only for certain source types.
 *
 * @return array<string, string> Option key = post ID (string), value = post title.
 */
private function get_template_options(): array {
    // ✅ Empty string key '' matches the SELECT2 default value of ''.
    // Using '0' as the placeholder key would be treated as a valid post ID.
    $options = [ '' => esc_html__( '— Select Template —', 'myplugin' ) ];

    $templates = get_posts( [
        'post_type'      => 'elementor_library',
        'post_status'    => 'publish',
        'posts_per_page' => -1,
        'orderby'        => 'title',
        'order'          => 'ASC',
        // ✅ no_found_rows: true — skip COUNT(*) since we don't paginate here
        'no_found_rows'  => true,
    ] );

    foreach ( $templates as $template ) {
        $options[ (string) $template->ID ] = $template->post_title;
    }

    return $options;
}
```

---

## has_widget_inner_wrapper() + is_dynamic_content() + render() skeleton

```php
public function has_widget_inner_wrapper(): bool {
    return false;
}

// ✅ MUST return true — rendered output depends entirely on which template
// the user selects. Caching the output would freeze one template's HTML
// permanently, even after the user changes the selection.
// Source: developers.elementor.com/docs/widgets/widget-output-caching/
protected function is_dynamic_content(): bool {
    return true;
}

protected function render(): void {
    $settings    = $this->get_settings_for_display();
    $template_id = (int) ( $settings['template_id'] ?? 0 );

    if ( $template_id < 1 ) {
        // ✅ Show a placeholder in the editor so the user knows to pick a template
        if ( \Elementor\Plugin::$instance->editor
             && \Elementor\Plugin::$instance->editor->is_edit_mode() ) {
            echo '<div style="padding:2rem;text-align:center;background:#f0f0f0;color:#888;">'
                 . esc_html__( 'Please select an Elementor template.', 'myplugin' )
                 . '</div>';
        }
        return;
    }

    // ✅ Guard: do NOT render if template_id equals the current post ID.
    // Elementor itself outputs this exact error string when recursion is detected.
    // Source: elementor/includes/frontend.php — get_builder_content_for_display()
    if ( $template_id === get_the_ID() ) {
        echo '<div class="elementor-alert elementor-alert-danger">'
             . esc_html__( 'Invalid Data: The Template ID cannot be the same as the currently edited template. Please choose a different one.', 'elementor' )
             . '</div>';
        return;
    }

    // ✅ Verify the post exists and is published before passing to Elementor.
    // get_builder_content_for_display() silently returns '' for missing posts.
    $template_post = get_post( $template_id );
    if ( ! $template_post || 'publish' !== $template_post->post_status ) {
        if ( \Elementor\Plugin::$instance->editor
             && \Elementor\Plugin::$instance->editor->is_edit_mode() ) {
            echo '<div style="padding:1rem;background:#fff3cd;color:#856404;">'
                 . esc_html__( 'Selected template is not published or does not exist.', 'myplugin' )
                 . '</div>';
        }
        return;
    }

    echo '<div class="myplugin-template-wrapper">';

    // ✅ get_builder_content_for_display() is the public API.
    // $with_css = true — always inline the template CSS to ensure styles load
    // even when this widget is the only Elementor element on the page.
    // Source: elementor/includes/frontend.php (Elementor GitHub main branch)
    // phpcs:ignore WordPress.Security.EscapeOutput.OutputNotEscaped
    echo \Elementor\Plugin::$instance->frontend->get_builder_content_for_display(
        $template_id,
        true  // $with_css
    );

    echo '</div>';
}
```

---

## content_template() skeleton

```php
protected function content_template(): void {
    ?>
    <#
    var templateId = parseInt( settings.template_id, 10 );
    #>
    <# if ( ! templateId ) { #>
        <div style="padding:2rem;text-align:center;background:#f0f0f0;color:#888;">
            <?php echo esc_html__( 'Please select an Elementor template.', 'myplugin' ); ?>
        </div>
    <# } else { #>
        <div class="myplugin-template-wrapper myplugin-template-placeholder"
             style="padding:2rem;text-align:center;background:#e8f4fd;color:#0c5460;">
            <span class="eicon-elementor" style="font-size:2rem;display:block;margin-bottom:.5rem;"></span>
            <?php echo esc_html__( 'Elementor Template ID:', 'myplugin' ); ?>
            <strong>{{ templateId }}</strong>
            <br>
            <small><?php echo esc_html__( 'Renders on frontend only.', 'myplugin' ); ?></small>
        </div>
    <# } #>
    <?php
}
```

> **Why the placeholder?** `get_builder_content_for_display()` is a PHP method —
> it cannot run inside the JS `content_template()`. The editor preview always shows
> a placeholder. The actual rendered template is only visible on the frontend.

---

## Known limitations & gotchas

> **Dynamic content in nested templates:**
> If the selected template contains Dynamic Tags (Post Title, ACF fields, etc.),
> `get_builder_content_for_display()` renders them in the context of the current
> page's `$post` — not the template post itself. This is expected behaviour.
> Source: github.com/elementor/elementor/issues/31600

> **Recursive template guard:**
> Never allow a template to embed itself. The `$template_id === get_the_ID()` guard
> above mirrors Elementor's own internal check in `frontend.php`. Without it,
> recursive rendering will produce a fatal error or infinite loop.

> **Output caching:**
> Because `is_dynamic_content()` returns `true`, Elementor will never cache this
> widget's output. This is correct — the rendered HTML changes depending on the
> template the user selects and its own dynamic content.

> **Page CSS vs inline CSS:**
> `$with_css = true` adds a `<style>` block inline. On high-traffic pages, consider
> whether this causes duplicate style output if many instances exist. An alternative
> is to enqueue the template's CSS file separately using
> `\Elementor\Core\Files\CSS\Post` — but `$with_css = true` is the simplest safe default.

---

> **Remove checklist:**
> - `min_height` responsive control → remove for simple full-bleed embeds
> - Editor placeholder styles → adjust colours to match project design system
> - `get_template_options()` query → add `'meta_query'` to filter by template type if needed
