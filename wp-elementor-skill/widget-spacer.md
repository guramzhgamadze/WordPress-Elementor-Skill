# Widget Boilerplate — Spacer

> **When to use this file:** Load whenever building a widget that adds vertical space between elements.
> Verified against `elementor/includes/widgets/spacer.php` (Elementor 3.35+).

---

```php
protected function register_controls(): void {

    $this->start_controls_section( 'section_spacer', [
        'label' => esc_html__( 'Spacer', 'myplugin' ),
        'tab'   => \Elementor\Controls_Manager::TAB_CONTENT,
    ] );

    $this->add_responsive_control( 'space', [
        'label'      => esc_html__( 'Space', 'myplugin' ),
        'type'       => \Elementor\Controls_Manager::SLIDER,
        'default'    => [ 'size' => 50, 'unit' => 'px' ],
        'size_units' => [ 'px', 'em', 'rem', 'vh', '%', 'custom' ],
        'range'      => [ 'px' => [ 'min' => 0, 'max' => 500 ] ],
        'selectors'  => [
            '{{WRAPPER}} .myplugin-spacer' => 'height: {{SIZE}}{{UNIT}};',
        ],
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
    echo '<div class="myplugin-spacer"></div>';
}

protected function content_template(): void {
    ?>
    <div class="myplugin-spacer" style="height: {{ settings.space.size }}{{ settings.space.unit }};"></div>
    <?php
}
```
