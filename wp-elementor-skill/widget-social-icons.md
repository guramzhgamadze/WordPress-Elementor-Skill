# Widget Boilerplate — Social Icons

> **When to use this file:** Load whenever building a widget showing a row of social media icon links.
> Verified against `elementor/includes/widgets/social-icons.php` (Elementor 3.35+ / V3 Widget_Base API, current through 4.2).

---

```php
protected function register_controls(): void {

    // ── CONTENT ───────────────────────────────────────────────
    $this->start_controls_section( 'section_social_icon', [
        'label' => esc_html__( 'Social Icons', 'myplugin' ),
        'tab'   => \Elementor\Controls_Manager::TAB_CONTENT,
    ] );

    $repeater = new \Elementor\Repeater();

    $repeater->add_control( 'social_icon', [
        'label'   => esc_html__( 'Icon', 'myplugin' ),
        'type'    => \Elementor\Controls_Manager::ICONS,
        'default' => [ 'value' => 'fab fa-wordpress', 'library' => 'fa-brands' ],
    ] );

    $repeater->add_control( 'link', [
        'label'   => esc_html__( 'Link', 'myplugin' ),
        'type'    => \Elementor\Controls_Manager::URL,
        'default' => [ 'is_external' => 'true' ],
        'dynamic' => [ 'active' => true ],
    ] );

    $repeater->add_control( 'item_icon_color', [
        'label'   => esc_html__( 'Color', 'myplugin' ),
        'type'    => \Elementor\Controls_Manager::SELECT,
        'default' => 'default',
        'options' => [
            // 'default' = inherit the global Primary/Hover colors from the Style tab.
            // (This boilerplate does not implement per-brand "official" colors — that would
            // require a full brand-color lookup table. Use 'custom' for a per-item color.)
            'default' => esc_html__( 'Default', 'myplugin' ),
            'custom'  => esc_html__( 'Custom',  'myplugin' ),
        ],
    ] );

    // ✅ Per-item color, shown only when item_icon_color = 'custom'. The {{CURRENT_ITEM}}
    // token resolves to `.elementor-repeater-item-{id}` — which render() must add to the
    // <a> (see below) for this selector to target the right item.
    $repeater->add_control( 'item_icon_primary_color', [
        'label'     => esc_html__( 'Custom Color', 'myplugin' ),
        'type'      => \Elementor\Controls_Manager::COLOR,
        'condition' => [ 'item_icon_color' => 'custom' ],
        'selectors' => [
            '{{WRAPPER}} {{CURRENT_ITEM}} i'   => 'color: {{VALUE}};',
            '{{WRAPPER}} {{CURRENT_ITEM}} svg' => 'fill: {{VALUE}};',
        ],
    ] );

    $this->add_control( 'social_icon_list', [
        'label'       => esc_html__( 'Social Icons', 'myplugin' ),
        'type'        => \Elementor\Controls_Manager::REPEATER,
        'fields'      => $repeater->get_controls(),
        'default'     => [
            [ 'social_icon' => [ 'value' => 'fab fa-facebook', 'library' => 'fa-brands' ], 'link' => [ 'url' => 'https://facebook.com', 'is_external' => 'true' ] ],
            // ✅ Twitter rebranded to X in July 2023. Font Awesome 6.4+ ships 'fa-x-twitter'
            // (fa-brands). Elementor bundles Font Awesome 6.x — fa-x-twitter is available.
            // fa-twitter still renders (FA keeps it as an alias) but fa-x-twitter is correct.
            [ 'social_icon' => [ 'value' => 'fab fa-x-twitter', 'library' => 'fa-brands' ], 'link' => [ 'url' => 'https://x.com', 'is_external' => 'true' ] ],
            [ 'social_icon' => [ 'value' => 'fab fa-instagram','library' => 'fa-brands' ], 'link' => [ 'url' => 'https://instagram.com','is_external' => 'true' ] ],
        ],
        'title_field' => '<# var migrated = "undefined" !== typeof __fa4_migrated, icon = ( migrated || ! icon ) ? social_icon.value : icon; #>{{{ icon }}}',
    ] );

    $this->end_controls_section();

    // ── STYLE ─────────────────────────────────────────────────
    $this->start_controls_section( 'section_social_style', [
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

    $this->add_responsive_control( 'icon_size', [
        'label'     => esc_html__( 'Size', 'myplugin' ),
        'type'      => \Elementor\Controls_Manager::SLIDER,
        'range'     => [ 'px' => [ 'min' => 6, 'max' => 300 ] ],
        'default'   => [ 'size' => 25, 'unit' => 'px' ],
        'selectors' => [
            '{{WRAPPER}} .myplugin-social-icon' => 'font-size: {{SIZE}}{{UNIT}};',
        ],
    ] );

    $this->add_responsive_control( 'icon_spacing', [
        'label'     => esc_html__( 'Spacing', 'myplugin' ),
        'type'      => \Elementor\Controls_Manager::SLIDER,
        'range'     => [ 'px' => [ 'min' => 0, 'max' => 100 ] ],
        'default'   => [ 'size' => 5, 'unit' => 'px' ],
        'selectors' => [
            '{{WRAPPER}} .myplugin-social-icon:not(:last-child)' => 'margin-right: {{SIZE}}{{UNIT}};',
        ],
    ] );

    $this->start_controls_tabs( 'tabs_social_icon_style' );

        $this->start_controls_tab( 'tab_social_icon_normal', [ 'label' => esc_html__( 'Normal', 'myplugin' ) ] );

            $this->add_control( 'icon_primary_color', [
                'label'     => esc_html__( 'Primary Color', 'myplugin' ),
                'type'      => \Elementor\Controls_Manager::COLOR,
                'selectors' => [
                    '{{WRAPPER}} .myplugin-social-icon i'   => 'color: {{VALUE}};',
                    '{{WRAPPER}} .myplugin-social-icon svg' => 'fill: {{VALUE}};',
                ],
            ] );

        $this->end_controls_tab();

        $this->start_controls_tab( 'tab_social_icon_hover', [ 'label' => esc_html__( 'Hover', 'myplugin' ) ] );

            $this->add_control( 'hover_primary_color', [
                'label'     => esc_html__( 'Primary Color', 'myplugin' ),
                'type'      => \Elementor\Controls_Manager::COLOR,
                'selectors' => [
                    '{{WRAPPER}} .myplugin-social-icon:hover i'   => 'color: {{VALUE}};',
                    '{{WRAPPER}} .myplugin-social-icon:hover svg' => 'fill: {{VALUE}};',
                ],
            ] );

        $this->end_controls_tab();

    $this->end_controls_tabs();

    $this->add_responsive_control( 'border_radius', [
        'label'      => esc_html__( 'Border Radius', 'myplugin' ),
        'type'       => \Elementor\Controls_Manager::DIMENSIONS,
        'size_units' => [ 'px', '%', 'em', 'rem' ],
        'selectors'  => [
            '{{WRAPPER}} .myplugin-social-icon' => 'border-radius: {{TOP}}{{UNIT}} {{RIGHT}}{{UNIT}} {{BOTTOM}}{{UNIT}} {{LEFT}}{{UNIT}};',
        ],
        'separator'  => 'before',
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
    <div class="myplugin-social-icons">
        <?php foreach ( $settings['social_icon_list'] as $item ) :
            if ( ! empty( $item['link']['url'] ) ) {
                $this->add_link_attributes( 'social-icon-' . $item['_id'], $item['link'] );
            }

            // ✅ ACCESSIBILITY: the icon is aria-hidden, so without this the link has NO
            // accessible name (WCAG 2.4.4 / 4.1.2 failure — screen readers announce an empty
            // link). Derive a readable label from the icon class (e.g. "fab fa-x-twitter" →
            // "X-twitter"), mirroring how Elementor's native widget adds an .elementor-screen-only
            // label. Hide it visually with a clip utility class (position:absolute; width:1px; etc.).
            $icon_value = $item['social_icon']['value'] ?? '';
            $sr_label   = $icon_value
                ? ucfirst( preg_replace( '/^.*fa-/', '', $icon_value ) )
                : esc_html__( 'Social link', 'myplugin' );
            ?>
            <a class="myplugin-social-icon elementor-repeater-item-<?php echo esc_attr( $item['_id'] ); ?>" <?php $this->print_render_attribute_string( 'social-icon-' . $item['_id'] ); ?>>
                <?php \Elementor\Icons_Manager::render_icon( $item['social_icon'], [ 'aria-hidden' => 'true' ] ); ?>
                <span class="myplugin-screen-only"><?php echo esc_html( $sr_label ); ?></span>
            </a>
        <?php endforeach; ?>
    </div>
    <?php
}
```


**content_template() skeleton:**

```php
protected function content_template(): void {
    ?>
    <# if ( ! settings.social_icon_list || ! settings.social_icon_list.length ) { return; } #>
    <div class="myplugin-social-icons">
        <# _.each( settings.social_icon_list, function( item ) {
            var iconHTML = elementor.helpers.renderIcon( view, item.social_icon, { 'aria-hidden': true }, 'i', 'object' );
            // ✅ Official Elementor pattern — link.url used directly per official advanced example
            var url = item.link && item.link.url ? item.link.url : '#';
        #>
        <a class="myplugin-social-icon elementor-repeater-item-{{ item._id }}" href="{{ url }}">{{{ iconHTML.value }}}</a>
        <# } ); #>
    </div>
    <?php
}
```
