# Widget Boilerplate — HTML

> **When to use this file:** Load whenever building a widget that outputs raw custom HTML/JS/CSS code.
> Verified against `elementor/includes/widgets/html.php` (Elementor 3.35+).

---

```php
protected function register_controls(): void {

    $this->start_controls_section( 'section_html', [
        'label' => esc_html__( 'HTML', 'myplugin' ),
        'tab'   => \Elementor\Controls_Manager::TAB_CONTENT,
    ] );

    $this->add_control( 'html', [
        'label'       => esc_html__( 'HTML Code', 'myplugin' ),
        'type'        => \Elementor\Controls_Manager::CODE,
        'language'    => 'html',
        'rows'        => 20,
        'default'     => '',
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

// ✅ Return false to enable output caching for static widgets (same HTML for all users).
// Return true if output varies per user, session, time, or random values.
// Source: developers.elementor.com/docs/widgets/widget-output-caching/
protected function is_dynamic_content(): bool {
    return false;
}

protected function render(): void {
    $settings = $this->get_settings_for_display();

    if ( empty( $settings['html'] ) ) {
        return;
    }

    // ✅ HTML widget outputs raw user HTML — this is intentional and matches
    // Elementor's native HTML widget behaviour. Only users with 'unfiltered_html'
    // capability can save arbitrary HTML via the CODE control in Elementor.
    // wp_kses_post() would strip scripts/iframes which defeats the widget's purpose.
    // The phpcs ignore is correct — document the intent clearly instead.
    echo $settings['html']; // phpcs:ignore WordPress.Security.EscapeOutput.OutputNotEscaped -- intentional: HTML widget mirrors native Elementor HTML widget; unfiltered_html cap enforced by Elementor editor
}
```

---



**content_template() skeleton:**

```php
protected function content_template(): void {
    ?>
    <# if ( settings.html ) { #>
        <div class="myplugin-html-wrapper">{{{ settings.html }}}</div>
    <# } #>
    <?php
}
```
