# Widget Boilerplate — Alert

> **When to use this file:** Load whenever building a colored notice/alert box widget.
> Verified against `elementor/includes/widgets/alert.php` (Elementor 3.35+).

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
        'prefix_class' => 'elementor-alert-',
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
    ?>
    <div class="myplugin-alert" role="alert">
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
    <# if ( settings.alert_title || settings.alert_description ) { #>
    <div class="myplugin-alert" role="alert">
        <# if ( 'show' === settings.show_dismiss ) { #>
            <button type="button" class="myplugin-alert-dismiss">&times;</button>
        <# } #>
        <# if ( settings.alert_title ) { #>
            <span class="myplugin-alert-title">{{{ settings.alert_title }}}</span>
        <# } #>
        <# if ( settings.alert_description ) { #>
            <span class="myplugin-alert-description">{{{ settings.alert_description }}}</span>
        <# } #>
    </div>
    <# } #>
    <?php
}
```
