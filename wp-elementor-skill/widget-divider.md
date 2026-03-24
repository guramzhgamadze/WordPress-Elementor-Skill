# Widget Boilerplate — Divider

> **When to use this file:** Load whenever building a widget that separates content sections with a styled line.
> Verified against `elementor/includes/widgets/divider.php` (Elementor 3.35+).

---

```php
protected function register_controls(): void {

    // ── CONTENT ───────────────────────────────────────────────
    $this->start_controls_section( 'section_divider', [
        'label' => esc_html__( 'Divider', 'myplugin' ),
        'tab'   => \Elementor\Controls_Manager::TAB_CONTENT,
    ] );

    $this->add_control( 'style', [
        'label'        => esc_html__( 'Style', 'myplugin' ),
        'type'         => \Elementor\Controls_Manager::SELECT,
        'default'      => 'solid',
        'options'      => [
            'solid'  => esc_html__( 'Solid',  'myplugin' ),
            'double' => esc_html__( 'Double', 'myplugin' ),
            'dotted' => esc_html__( 'Dotted', 'myplugin' ),
            'dashed' => esc_html__( 'Dashed', 'myplugin' ),
        ],
        'selectors'    => [
            '{{WRAPPER}} .myplugin-divider-separator' => 'border-top-style: {{VALUE}};',
        ],
        'prefix_class' => 'myplugin-divider-style-',
    ] );

    $this->add_responsive_control( 'width', [
        'label'      => esc_html__( 'Width', 'myplugin' ),
        'type'       => \Elementor\Controls_Manager::SLIDER,
        'size_units' => [ '%', 'px', 'vw' ],
        'range'      => [
            '%'  => [ 'min' => 1,  'max' => 100  ],
            'px' => [ 'min' => 1,  'max' => 1000 ],
            'vw' => [ 'min' => 1,  'max' => 100  ],
        ],
        'default'    => [ 'unit' => '%', 'size' => 100 ],
        'selectors'  => [
            '{{WRAPPER}} .myplugin-divider-separator' => 'width: {{SIZE}}{{UNIT}};',
        ],
    ] );

    $this->add_responsive_control( 'align', [
        'label'     => esc_html__( 'Alignment', 'myplugin' ),
        'type'      => \Elementor\Controls_Manager::CHOOSE,
        'options'   => [
            'left'   => [ 'title' => esc_html__( 'Left',   'myplugin' ), 'icon' => 'eicon-text-align-left'   ],
            'center' => [ 'title' => esc_html__( 'Center', 'myplugin' ), 'icon' => 'eicon-text-align-center' ],
            'right'  => [ 'title' => esc_html__( 'Right',  'myplugin' ), 'icon' => 'eicon-text-align-right'  ],
        ],
        'selectors' => [
            '{{WRAPPER}} .myplugin-divider' => 'text-align: {{VALUE}};',
            // ✅ margin trick centres the separator line regardless of width
            '{{WRAPPER}} .myplugin-divider-separator' => 'margin: 0 auto; margin-{{VALUE}}: 0;',
        ],
    ] );

    // Add Element — none / text / icon
    $this->add_control( 'look', [
        'label'       => esc_html__( 'Add Element', 'myplugin' ),
        'type'        => \Elementor\Controls_Manager::CHOOSE,
        'options'     => [
            'line'      => [ 'title' => esc_html__( 'None', 'myplugin' ), 'icon' => 'eicon-minus' ],
            'line_text' => [ 'title' => esc_html__( 'Text', 'myplugin' ), 'icon' => 'eicon-t-letter' ],
            'line_icon' => [ 'title' => esc_html__( 'Icon', 'myplugin' ), 'icon' => 'eicon-star' ],
        ],
        'default'     => 'line',
        'prefix_class'=> 'myplugin-divider-',
        'label_block' => false,
    ] );

    $this->add_control( 'text', [
        'label'     => esc_html__( 'Text', 'myplugin' ),
        'type'      => \Elementor\Controls_Manager::TEXT,
        'default'   => esc_html__( 'Divider', 'myplugin' ),
        'condition' => [ 'look' => 'line_text' ],
        'dynamic'   => [ 'active' => true ],
    ] );

    $this->add_control( 'selected_icon', [
        'label'     => esc_html__( 'Icon', 'myplugin' ),
        'type'      => \Elementor\Controls_Manager::ICONS,
        'default'   => [ 'value' => 'fas fa-star', 'library' => 'fa-solid' ],
        'condition' => [ 'look' => 'line_icon' ],
    ] );

    // Text/icon position relative to the line
    $this->add_control( 'text_align', [
        'label'     => esc_html__( 'Position', 'myplugin' ),
        'type'      => \Elementor\Controls_Manager::CHOOSE,
        'options'   => [
            'left'   => [ 'title' => esc_html__( 'Left',   'myplugin' ), 'icon' => 'eicon-h-align-left'   ],
            'center' => [ 'title' => esc_html__( 'Center', 'myplugin' ), 'icon' => 'eicon-h-align-center' ],
            'right'  => [ 'title' => esc_html__( 'Right',  'myplugin' ), 'icon' => 'eicon-h-align-right'  ],
        ],
        'default'   => 'center',
        'condition' => [ 'look!' => 'line' ],
    ] );

    $this->add_control( 'middle_width', [
        'label'     => esc_html__( 'Spacing', 'myplugin' ),
        'type'      => \Elementor\Controls_Manager::SLIDER,
        'range'     => [ 'px' => [ 'min' => 1, 'max' => 50 ] ],
        'default'   => [ 'size' => 10, 'unit' => 'px' ],
        'condition' => [ 'look!' => 'line' ],
        'selectors' => [
            '{{WRAPPER}} .myplugin-divider__text' => 'padding: 0 {{SIZE}}{{UNIT}};',
            '{{WRAPPER}} .myplugin-divider__icon' => 'padding: 0 {{SIZE}}{{UNIT}};',
        ],
    ] );

    $this->end_controls_section();

    // ── STYLE ─────────────────────────────────────────────────
    $this->start_controls_section( 'section_divider_style', [
        'label' => esc_html__( 'Divider', 'myplugin' ),
        'tab'   => \Elementor\Controls_Manager::TAB_STYLE,
    ] );

    $this->add_control( 'color', [
        'label'     => esc_html__( 'Color', 'myplugin' ),
        'type'      => \Elementor\Controls_Manager::COLOR,
        'global'    => [ 'default' => \Elementor\Core\Kits\Documents\Tabs\Global_Colors::COLOR_SECONDARY ],
        'selectors' => [
            '{{WRAPPER}} .myplugin-divider-separator' => 'border-top-color: {{VALUE}};',
            '{{WRAPPER}} .myplugin-divider__text'     => 'color: {{VALUE}};',
            '{{WRAPPER}} .myplugin-divider__icon i'   => 'color: {{VALUE}};',
            '{{WRAPPER}} .myplugin-divider__icon svg' => 'fill: {{VALUE}};',
        ],
    ] );

    $this->add_control( 'weight', [
        'label'     => esc_html__( 'Weight', 'myplugin' ),
        'type'      => \Elementor\Controls_Manager::SLIDER,
        'range'     => [ 'px' => [ 'min' => 1, 'max' => 10 ] ],
        'default'   => [ 'size' => 1, 'unit' => 'px' ],
        'selectors' => [
            '{{WRAPPER}} .myplugin-divider-separator' => 'border-top-width: {{SIZE}}{{UNIT}};',
        ],
    ] );

    $this->add_responsive_control( 'gap', [
        'label'      => esc_html__( 'Gap', 'myplugin' ),
        'type'       => \Elementor\Controls_Manager::SLIDER,
        'size_units' => [ 'px', 'em', 'rem' ],
        'range'      => [ 'px' => [ 'min' => 0, 'max' => 100 ] ],
        'default'    => [ 'size' => 15, 'unit' => 'px' ],
        'selectors'  => [
            '{{WRAPPER}} .myplugin-divider' => 'padding-top: {{SIZE}}{{UNIT}}; padding-bottom: {{SIZE}}{{UNIT}};',
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
    <div class="myplugin-divider">
        <span class="myplugin-divider-separator">
            <?php if ( 'line_text' === $settings['look'] && ! empty( $settings['text'] ) ) : ?>
                <span class="myplugin-divider__text"><?php echo esc_html( $settings['text'] ); ?></span>
            <?php elseif ( 'line_icon' === $settings['look'] && ! empty( $settings['selected_icon']['value'] ) ) : ?>
                <span class="myplugin-divider__icon">
                    <?php \Elementor\Icons_Manager::render_icon( $settings['selected_icon'], [ 'aria-hidden' => 'true' ] ); ?>
                </span>
            <?php endif; ?>
        </span>
    </div>
    <?php
}
```

> **Remove checklist:**
> - `look` + `text` + `selected_icon` + `text_align` + `middle_width` → remove if divider is always a plain line


**content_template() skeleton:**

```php
protected function content_template(): void {
    ?>
    <div class="myplugin-divider">
        <span class="myplugin-divider-separator">
            <# if ( 'line_text' === settings.look && settings.text ) { #>
                <span class="myplugin-divider__text">{{{ settings.text }}}</span>
            <# } else if ( 'line_icon' === settings.look && settings.selected_icon && settings.selected_icon.value ) {
                var iconHTML = elementor.helpers.renderIcon( view, settings.selected_icon, { 'aria-hidden': true }, 'i', 'object' );
            #>
                <span class="myplugin-divider__icon">{{{ iconHTML.value }}}</span>
            <# } #>
        </span>
    </div>
    <?php
}
```
