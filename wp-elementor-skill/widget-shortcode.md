# Widget Boilerplate — Shortcode

> **When to use this file:** Load whenever building a widget that outputs a WordPress shortcode.
> Verified against `elementor/includes/widgets/shortcode.php` (Elementor 3.35+).

---

```php
protected function register_controls(): void {

    $this->start_controls_section( 'shortcode_section', [
        'label' => esc_html__( 'Shortcode', 'myplugin' ),
        'tab'   => \Elementor\Controls_Manager::TAB_CONTENT,
    ] );

    $this->add_control( 'shortcode', [
        'label'       => esc_html__( 'Enter your shortcode', 'myplugin' ),
        'type'        => \Elementor\Controls_Manager::TEXTAREA,
        'default'     => '',
        'placeholder' => '[shortcode]',
        'dynamic'     => [ 'active' => true ],
    ] );

    $this->end_controls_section();
}
```

**render() skeleton:**

```php

// ✅ REQUIRED on every widget — removes the redundant inner wrapper div.
// Return true ONLY if your render() output physically requires the inner wrapper.
// Source: developers.elementor.com/docs/widgets/widget-inner-wrapper/
public function has_widget_inner_wrapper(): bool {
    return false;
}

// ✅ MUST be true — do_shortcode() may execute shortcodes that output user-specific
// content (WooCommerce cart totals, membership gates, nonces, logged-in user data, etc.).
// Returning false would allow Elementor's output cache to serve one user's shortcode
// output to a different user. Always return true for shortcode widgets.
// Source: developers.elementor.com/docs/widgets/widget-output-caching/
protected function is_dynamic_content(): bool {
    return true;
}

protected function render(): void {
    $settings  = $this->get_settings_for_display();
    $shortcode = do_shortcode( shortcode_unautop( $settings['shortcode'] ) );
    // ✅ wp_kses_post() is intentionally used here — it strips <script> tags and other
    // unsafe markup from shortcode output. Most shortcodes should enqueue scripts via
    // wp_enqueue_script(), not inline them. If a specific shortcode genuinely requires
    // unfiltered output, use the HTML widget instead (which requires unfiltered_html cap).
    echo '<div class="myplugin-shortcode">' . wp_kses_post( $shortcode ) . '</div>'; // phpcs:ignore WordPress.Security.EscapeOutput.OutputNotEscaped -- shortcode output is kses-filtered above
}
```

---



**content_template() skeleton:**

```php
protected function content_template(): void {
    ?>
    <# if ( settings.shortcode ) { #>
        <div class="myplugin-shortcode">
            <p class="myplugin-shortcode-placeholder">
                <?php echo esc_html__( '[Shortcode placeholder — renders on frontend]', 'myplugin' ); ?>
                <code>{{{ settings.shortcode }}}</code>
            </p>
        </div>
    <# } #>
    <?php
}
```
