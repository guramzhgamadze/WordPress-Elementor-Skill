# Widget Boilerplate — Alert

> **When to use this file:** Load whenever building a colored notice/alert box widget.
> Verified against `elementor/includes/widgets/alert.php` (Elementor 3.35+ / V3 Widget_Base API, current through 4.2).

---

```php
protected function register_controls(): void {

    $this->start_controls_section( 'section_alert', [
        'label' => esc_html__( 'Alert', 'myplugin' ),
        'tab'   => \Elementor\Controls_Manager::TAB_CONTENT,
    ] );

    $this->add_control( 'alert_type', [
        'label'        => esc_html__( 'Type', 'myplugin' ),
        'type'         => \Elementor\Controls_Manager::SELECT,
        'default'      => 'info',
        'options'      => [
            'info'    => esc_html__( 'Info',    'myplugin' ),
            'success' => esc_html__( 'Success', 'myplugin' ),
            'warning' => esc_html__( 'Warning', 'myplugin' ),
            'danger'  => esc_html__( 'Danger',  'myplugin' ),
        ],
        // ✅ The selected type drives a `.myplugin-alert--{type}` modifier class on the alert
        // element in render() (see below) + the default CSS at the bottom of this file.
        // Do NOT use prefix_class => 'elementor-alert-': that writes the class to the widget
        // WRAPPER and depends on Elementor's native alert CSS being loaded, which is NOT
        // guaranteed for a custom widget — the colors would silently never appear.
    ] );

    $this->add_control( 'alert_title', [
        'label'       => esc_html__( 'Title & Description', 'myplugin' ),
        'type'        => \Elementor\Controls_Manager::TEXT,
        'placeholder' => esc_html__( 'Your Title', 'myplugin' ),
        'default'     => esc_html__( 'This is an Alert', 'myplugin' ),
        'label_block' => true,
        'dynamic'     => [ 'active' => true ],
    ] );

    $this->add_control( 'alert_description', [
        'label'       => '',
        'type'        => \Elementor\Controls_Manager::TEXTAREA,
        'default'     => esc_html__( 'I am a description. Click the edit button to change this text.', 'myplugin' ),
        'placeholder' => esc_html__( 'Your Description', 'myplugin' ),
        'dynamic'     => [ 'active' => true ],
    ] );

    $this->add_control( 'show_dismiss', [
        'label'   => esc_html__( 'Dismiss Button', 'myplugin' ),
        'type'    => \Elementor\Controls_Manager::SELECT,
        'default' => 'hide',
        'options' => [
            'show' => esc_html__( 'Show', 'myplugin' ),
            'hide' => esc_html__( 'Hide', 'myplugin' ),
        ],
    ] );

    $this->end_controls_section();

    // ── STYLE ─────────────────────────────────────────────────
    $this->start_controls_section( 'section_type_style', [
        'label' => esc_html__( 'Alert Box', 'myplugin' ),
        'tab'   => \Elementor\Controls_Manager::TAB_STYLE,
    ] );

    // ✅ Background + Text color controls OVERRIDE the per-type CSS defaults — so visuals
    // remain fully user-controllable (SKILL.md §0 Golden Rule #6). Leave empty to keep the
    // type preset; set a value to override it.
    $this->add_control( 'background_color', [
        'label'     => esc_html__( 'Background Color', 'myplugin' ),
        'type'      => \Elementor\Controls_Manager::COLOR,
        'selectors' => [ '{{WRAPPER}} .myplugin-alert' => 'background-color: {{VALUE}};' ],
    ] );

    $this->add_control( 'alert_text_color', [
        'label'     => esc_html__( 'Text Color', 'myplugin' ),
        'type'      => \Elementor\Controls_Manager::COLOR,
        'selectors' => [ '{{WRAPPER}} .myplugin-alert' => 'color: {{VALUE}};' ],
    ] );

    $this->add_group_control(
        \Elementor\Group_Control_Border::get_type(),
        [ 'name' => 'alert_border', 'selector' => '{{WRAPPER}} .myplugin-alert' ]
    );

    $this->add_responsive_control( 'border_radius', [
        'label'      => esc_html__( 'Border Radius', 'myplugin' ),
        'type'       => \Elementor\Controls_Manager::DIMENSIONS,
        'size_units' => [ 'px', '%', 'em', 'rem' ],
        'selectors'  => [
            '{{WRAPPER}} .myplugin-alert' => 'border-radius: {{TOP}}{{UNIT}} {{RIGHT}}{{UNIT}} {{BOTTOM}}{{UNIT}} {{LEFT}}{{UNIT}};',
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
    $settings = $this->get_settings_for_display();
    // ✅ Type modifier class on the alert element itself — this is what makes the
    // info/success/warning/danger colors actually apply (see CSS block below).
    $type = ! empty( $settings['alert_type'] ) ? $settings['alert_type'] : 'info';
    ?>
    <div class="myplugin-alert myplugin-alert--<?php echo esc_attr( $type ); ?>" role="alert">
        <?php if ( 'show' === $settings['show_dismiss'] ) : ?>
            <button type="button" class="myplugin-alert-dismiss" aria-label="<?php esc_attr_e( 'Close', 'myplugin' ); ?>">
                &times;
            </button>
        <?php endif; ?>
        <?php if ( ! empty( $settings['alert_title'] ) ) : ?>
            <span class="myplugin-alert-title"><?php echo esc_html( $settings['alert_title'] ); ?></span>
        <?php endif; ?>
        <?php if ( ! empty( $settings['alert_description'] ) ) : ?>
            <span class="myplugin-alert-description"><?php echo wp_kses_post( $settings['alert_description'] ); ?></span>
        <?php endif; ?>
    </div>
    <?php
}
```

---



**content_template() skeleton:**

```php
protected function content_template(): void {
    ?>
    <# if ( settings.alert_title || settings.alert_description ) {
        var type = settings.alert_type || 'info';
    #>
    <div class="myplugin-alert myplugin-alert--{{ type }}" role="alert">
        <# if ( 'show' === settings.show_dismiss ) { #>
            <button type="button" class="myplugin-alert-dismiss">&times;</button>
        <# } #>
        <# if ( settings.alert_title ) { #>
            <span class="myplugin-alert-title">{{ settings.alert_title }}</span>
        <# } #>
        <# if ( settings.alert_description ) { #>
            <span class="myplugin-alert-description">{{ settings.alert_description }}</span>
        <# } #>
    </div>
    <# } #>
    <?php
}
```

---

**Default styles** (`assets/css/myplugin-alert.css` — register it and declare via
`get_style_depends()`; see `elementor-patterns.md` Step 2). These are sensible **defaults**;
the Style-tab Background / Text / Border controls override them, so visuals stay user-controlled.

```css
/* Baseline layout + per-type default colors (mirrors how Elementor's own alert ships presets). */
.myplugin-alert {
    position: relative;
    padding: 15px;
    border-left: 5px solid transparent;
}
.myplugin-alert--info    { background: #d9edf7; border-color: #5bc0de; color: #31708f; }
.myplugin-alert--success { background: #dff0d8; border-color: #5cb85c; color: #3c763d; }
.myplugin-alert--warning { background: #fcf8e3; border-color: #f0ad4e; color: #8a6d3b; }
.myplugin-alert--danger  { background: #f2dede; border-color: #d9534f; color: #a94442; }
.myplugin-alert-title       { display: block; font-weight: 700; }
.myplugin-alert-description { display: block; }
.myplugin-alert-dismiss {
    position: absolute;
    top: 10px;
    inset-inline-end: 12px;
    padding: 0;
    background: none;
    border: 0;
    font-size: 18px;
    line-height: 1;
    color: inherit;
    cursor: pointer;
}
```
