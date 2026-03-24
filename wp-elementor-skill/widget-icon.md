# Widget Boilerplate — Icon

> **When to use this file:** Load whenever building a widget that displays a single standalone icon.
> Verified against `elementor/includes/widgets/icon.php` (Elementor 3.35+).

---

```php
protected function register_controls(): void {

    // ── CONTENT ───────────────────────────────────────────────
    $this->start_controls_section( 'section_icon', [
        'label' => esc_html__( 'Icon', 'myplugin' ),
        'tab'   => \Elementor\Controls_Manager::TAB_CONTENT,
    ] );

    $this->add_control( 'selected_icon', [
        'label'            => esc_html__( 'Icon', 'myplugin' ),
        'type'             => \Elementor\Controls_Manager::ICONS,
        'fa4compatibility' => 'icon',
        'default'          => [ 'value' => 'fas fa-star', 'library' => 'fa-solid' ],
    ] );

    $this->add_control( 'link', [
        'label'     => esc_html__( 'Link', 'myplugin' ),
        'type'      => \Elementor\Controls_Manager::URL,
        'dynamic'   => [ 'active' => true ],
        'separator' => 'before',
    ] );

    $this->end_controls_section();

    // ── STYLE ─────────────────────────────────────────────────
    $this->start_controls_section( 'section_style_icon', [
        'label' => esc_html__( 'Icon', 'myplugin' ),
        'tab'   => \Elementor\Controls_Manager::TAB_STYLE,
    ] );

    $this->add_responsive_control( 'align', [
        'label'     => esc_html__( 'Alignment', 'myplugin' ),
        'type'      => \Elementor\Controls_Manager::CHOOSE,
        'options'   => [
            'left'   => [ 'title' => esc_html__( 'Left',   'myplugin' ), 'icon' => 'eicon-text-align-left'   ],
            'center' => [ 'title' => esc_html__( 'Center', 'myplugin' ), 'icon' => 'eicon-text-align-center' ],
            'right'  => [ 'title' => esc_html__( 'Right',  'myplugin' ), 'icon' => 'eicon-text-align-right'  ],
        ],
        'selectors' => [ '{{WRAPPER}}' => 'text-align: {{VALUE}};' ],
    ] );

    $this->start_controls_tabs( 'icon_colors' );

        $this->start_controls_tab( 'icon_colors_normal', [
            'label' => esc_html__( 'Normal', 'myplugin' ),
        ] );

            $this->add_control( 'primary_color', [
                'label'     => esc_html__( 'Primary Color', 'myplugin' ),
                'type'      => \Elementor\Controls_Manager::COLOR,
                'default'   => '',
                'selectors' => [
                    '{{WRAPPER}} .myplugin-icon i'   => 'color: {{VALUE}};',
                    '{{WRAPPER}} .myplugin-icon svg' => 'fill: {{VALUE}};',
                ],
                'global'    => [ 'default' => \Elementor\Core\Kits\Documents\Tabs\Global_Colors::COLOR_PRIMARY ],
            ] );

        $this->end_controls_tab();

        $this->start_controls_tab( 'icon_colors_hover', [
            'label' => esc_html__( 'Hover', 'myplugin' ),
        ] );

            $this->add_control( 'hover_primary_color', [
                'label'     => esc_html__( 'Primary Color', 'myplugin' ),
                'type'      => \Elementor\Controls_Manager::COLOR,
                'default'   => '',
                'selectors' => [
                    '{{WRAPPER}} .myplugin-icon:hover i'   => 'color: {{VALUE}};',
                    '{{WRAPPER}} .myplugin-icon:hover svg' => 'fill: {{VALUE}};',
                ],
            ] );

            // ⛔ hover_animation excluded by default — loads animation CSS library
            // $this->add_control( 'hover_animation', [ 'type' => \Elementor\Controls_Manager::HOVER_ANIMATION ] );

        $this->end_controls_tab();

    $this->end_controls_tabs();

    $this->add_responsive_control( 'size', [
        'label'     => esc_html__( 'Size', 'myplugin' ),
        'type'      => \Elementor\Controls_Manager::SLIDER,
        'range'     => [ 'px' => [ 'min' => 6, 'max' => 300 ] ],
        'default'   => [ 'size' => 50, 'unit' => 'px' ],
        'selectors' => [
            '{{WRAPPER}} .myplugin-icon'     => 'font-size: {{SIZE}}{{UNIT}};',
            '{{WRAPPER}} .myplugin-icon svg' => 'width: {{SIZE}}{{UNIT}}; height: {{SIZE}}{{UNIT}};',
        ],
        'separator' => 'before',
    ] );

    $this->add_responsive_control( 'icon_padding', [
        'label'     => esc_html__( 'Padding', 'myplugin' ),
        'type'      => \Elementor\Controls_Manager::SLIDER,
        'range'     => [ 'em' => [ 'min' => 0, 'max' => 5 ] ],
        'default'   => [ 'unit' => 'em' ],
        'selectors' => [
            '{{WRAPPER}} .myplugin-icon' => 'padding: {{SIZE}}{{UNIT}};',
        ],
    ] );

    $this->add_group_control(
        \Elementor\Group_Control_Border::get_type(),
        [
            'name'      => 'border',
            'selector'  => '{{WRAPPER}} .myplugin-icon',
            'separator' => 'before',
        ]
    );

    $this->add_responsive_control( 'border_radius', [
        'label'      => esc_html__( 'Border Radius', 'myplugin' ),
        'type'       => \Elementor\Controls_Manager::DIMENSIONS,
        'size_units' => [ 'px', '%', 'em', 'rem' ],
        'selectors'  => [
            '{{WRAPPER}} .myplugin-icon' => 'border-radius: {{TOP}}{{UNIT}} {{RIGHT}}{{UNIT}} {{BOTTOM}}{{UNIT}} {{LEFT}}{{UNIT}};',
        ],
    ] );

    $this->add_group_control(
        \Elementor\Group_Control_Box_Shadow::get_type(),
        [
            'name'     => 'box_shadow',
            'selector' => '{{WRAPPER}} .myplugin-icon',
        ]
    );

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

    if ( empty( $settings['selected_icon']['value'] ) ) {
        return;
    }

    $icon_tag = 'div';
    if ( ! empty( $settings['link']['url'] ) ) {
        $this->add_link_attributes( 'link', $settings['link'] );
        $icon_tag = 'a';
    }
    ?>
    <<?php echo esc_attr( $icon_tag ); ?> class="myplugin-icon" <?php echo 'a' === $icon_tag ? $this->get_render_attribute_string( 'link' ) : ''; // phpcs:ignore WordPress.Security.EscapeOutput.OutputNotEscaped -- get_render_attribute_string() returns pre-sanitized attribute markup ?>>
        <?php \Elementor\Icons_Manager::render_icon( $settings['selected_icon'], [ 'aria-hidden' => 'true' ] ); ?>
    </<?php echo esc_attr( $icon_tag ); ?>>
    <?php
}
```

> **Remove checklist:**
> - `link` → remove if icon is decorative only
> - `border` + `border_radius` + `box_shadow` → remove for plain icon-only use
> - `icon_padding` → remove if icon has no background or border
> - `hover_animation` → excluded by default


**content_template() skeleton:**

```php
protected function content_template(): void {
    ?>
    <#
    if ( ! settings.selected_icon || ! settings.selected_icon.value ) { return; }
    var iconHTML = elementor.helpers.renderIcon( view, settings.selected_icon, { 'aria-hidden': true }, 'i', 'object' );
    var tag = settings.link && settings.link.url ? 'a' : 'div';
    // ✅ Official Elementor pattern — link.url used directly per official advanced example
    var linkAttr = 'a' === tag ? 'href="' + settings.link.url + '"' : '';
    #>
    <{{{ tag }}} class="myplugin-icon" {{{ linkAttr }}}>
        {{{ iconHTML.value }}}
    </{{{ tag }}}>
    <?php
}
```
