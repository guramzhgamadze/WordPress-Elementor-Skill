# Widget Boilerplate — Progress Bar

> **When to use this file:** Load whenever building a widget showing a percentage-based progress bar.
> Verified against `elementor/includes/widgets/progress.php` (Elementor 3.35+).

---

```php
protected function register_controls(): void {

    $this->start_controls_section( 'section_progress', [
        'label' => esc_html__( 'Progress Bar', 'myplugin' ),
        'tab'   => \Elementor\Controls_Manager::TAB_CONTENT,
    ] );

    $this->add_control( 'title', [
        'label'       => esc_html__( 'Title', 'myplugin' ),
        'type'        => \Elementor\Controls_Manager::TEXT,
        'placeholder' => esc_html__( 'Enter your title', 'myplugin' ),
        'default'     => esc_html__( 'Design', 'myplugin' ),
        'label_block' => true,
        'dynamic'     => [ 'active' => true ],
    ] );

    $this->add_control( 'percent', [
        'label'   => esc_html__( 'Percentage', 'myplugin' ),
        'type'    => \Elementor\Controls_Manager::SLIDER,
        'default' => [ 'size' => 50, 'unit' => '%' ],
        'range'   => [ '%' => [ 'min' => 0, 'max' => 100 ] ],
        'dynamic' => [ 'active' => true ],
    ] );

    $this->add_control( 'display_percentage', [
        'label'   => esc_html__( 'Display Percentage', 'myplugin' ),
        'type'    => \Elementor\Controls_Manager::SWITCHER,
        'default' => 'yes',
    ] );

    $this->add_control( 'inner_text', [
        'label'       => esc_html__( 'Inner Text', 'myplugin' ),
        'type'        => \Elementor\Controls_Manager::TEXT,
        'placeholder' => esc_html__( '75 of 100 points', 'myplugin' ),
        'default'     => esc_html__( '75 of 100 points', 'myplugin' ),
        'dynamic'     => [ 'active' => true ],
    ] );

    $this->end_controls_section();

    // ── STYLE ─────────────────────────────────────────────────
    $this->start_controls_section( 'section_progress_style', [
        'label' => esc_html__( 'Progress Bar', 'myplugin' ),
        'tab'   => \Elementor\Controls_Manager::TAB_STYLE,
    ] );

    $this->add_control( 'bar_color', [
        'label'     => esc_html__( 'Color', 'myplugin' ),
        'type'      => \Elementor\Controls_Manager::COLOR,
        'global'    => [ 'default' => \Elementor\Core\Kits\Documents\Tabs\Global_Colors::COLOR_PRIMARY ],
        'selectors' => [ '{{WRAPPER}} .myplugin-bar-fill' => 'background-color: {{VALUE}};' ],
    ] );

    $this->add_control( 'bar_bg_color', [
        'label'     => esc_html__( 'Background Color', 'myplugin' ),
        'type'      => \Elementor\Controls_Manager::COLOR,
        'selectors' => [ '{{WRAPPER}} .myplugin-progress-bar' => 'background-color: {{VALUE}};' ],
    ] );

    $this->add_responsive_control( 'bar_height', [
        'label'     => esc_html__( 'Height', 'myplugin' ),
        'type'      => \Elementor\Controls_Manager::SLIDER,
        'range'     => [ 'px' => [ 'min' => 1, 'max' => 200 ] ],
        'default'   => [ 'size' => 20, 'unit' => 'px' ],
        'selectors' => [ '{{WRAPPER}} .myplugin-progress-bar' => 'height: {{SIZE}}{{UNIT}};' ],
    ] );

    $this->add_responsive_control( 'border_radius', [
        'label'      => esc_html__( 'Border Radius', 'myplugin' ),
        'type'       => \Elementor\Controls_Manager::DIMENSIONS,
        'size_units' => [ 'px', '%', 'em', 'rem' ],
        'selectors'  => [
            '{{WRAPPER}} .myplugin-progress-bar, {{WRAPPER}} .myplugin-bar-fill' => 'border-radius: {{TOP}}{{UNIT}} {{RIGHT}}{{UNIT}} {{BOTTOM}}{{UNIT}} {{LEFT}}{{UNIT}};',
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
    $pct      = min( 100, (float) ( $settings['percent']['size'] ?? 0 ) );
    ?>
    <div class="myplugin-progress">
        <?php if ( ! empty( $settings['title'] ) ) : ?>
            <div class="myplugin-progress-title">
                <span><?php echo esc_html( $settings['title'] ); ?></span>
                <?php if ( 'yes' === $settings['display_percentage'] ) : ?>
                    <span class="myplugin-progress-percentage"><?php echo esc_html( $pct ); ?>%</span>
                <?php endif; ?>
            </div>
        <?php endif; ?>
        <div class="myplugin-progress-bar" role="progressbar" aria-valuenow="<?php echo esc_attr( $pct ); ?>" aria-valuemin="0" aria-valuemax="100">
            <div class="myplugin-bar-fill" style="width: <?php echo esc_attr( $pct ); ?>%">
                <?php if ( ! empty( $settings['inner_text'] ) ) : ?>
                    <span class="myplugin-bar-inner-text"><?php echo esc_html( $settings['inner_text'] ); ?></span>
                <?php endif; ?>
            </div>
        </div>
    </div>
    <?php
}
```


**content_template() skeleton:**

```php
protected function content_template(): void {
    ?>
    <#
    var pct = Math.min( 100, parseFloat( settings.percent && settings.percent.size || 0 ) );
    #>
    <div class="myplugin-progress">
        <# if ( settings.title ) { #>
        <div class="myplugin-progress-title">
            <span>{{{ settings.title }}}</span>
            <# if ( 'yes' === settings.display_percentage ) { #>
                <span class="myplugin-progress-percentage">{{ pct }}%</span>
            <# } #>
        </div>
        <# } #>
        <div class="myplugin-progress-bar" role="progressbar"
             aria-valuenow="{{ pct }}" aria-valuemin="0" aria-valuemax="100">
            <div class="myplugin-bar-fill" style="width:{{ pct }}%">
                <# if ( settings.inner_text ) { #>
                    <span class="myplugin-bar-inner-text">{{{ settings.inner_text }}}</span>
                <# } #>
            </div>
        </div>
    </div>
    <?php
}
```
