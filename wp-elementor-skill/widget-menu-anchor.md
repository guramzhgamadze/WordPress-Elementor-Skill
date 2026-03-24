# Widget Boilerplate — Menu Anchor

> **When to use this file:** Load whenever building a widget that creates a named anchor for in-page navigation.
> Verified against `elementor/includes/widgets/menu-anchor.php` (Elementor 3.35+).

---

```php
protected function register_controls(): void {

    $this->start_controls_section( 'section_anchor', [
        'label' => esc_html__( 'Menu Anchor', 'myplugin' ),
        'tab'   => \Elementor\Controls_Manager::TAB_CONTENT,
    ] );

    $this->add_control( 'anchor', [
        'label'       => esc_html__( 'Menu Anchor ID', 'myplugin' ),
        'type'        => \Elementor\Controls_Manager::TEXT,
        'placeholder' => esc_html__( 'For Example: About', 'myplugin' ),
        'description' => esc_html__( 'This ID will be the CSS ID you will have to use without the # sign.', 'myplugin' ),
        'label_block' => true,
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
    $id = $this->get_settings_for_display()['anchor'];
    if ( ! empty( $id ) ) {
        echo '<div id="' . esc_attr( sanitize_html_class( $id ) ) . '"></div>';
    }
}
```

---



**content_template() skeleton:**

```php
protected function content_template(): void {
    ?>
    <# if ( settings.anchor ) {
        // ✅ Strip characters that are invalid in HTML id values (keep a-z A-Z 0-9 _ -)
        // Mirrors PHP-side sanitize_html_class() to keep editor preview consistent with frontend.
        var safeId = settings.anchor.replace( /[^a-zA-Z0-9_-]/g, '' );
    #>
        <div id="{{ safeId }}" class="myplugin-menu-anchor"></div>
    <# } #>
    <?php
}
```
