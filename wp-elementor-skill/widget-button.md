# Widget Boilerplate — Button

> **When to use this file:** Load this boilerplate whenever building a widget that has a
> primary clickable call-to-action — a button, submit trigger, or any linked element with
> text and optional icon.
>
> Verified against `elementor/includes/widgets/traits/button-trait.php` (Elementor 3.35+).
> Control IDs, defaults, selectors, and order match Elementor's native Button widget exactly.

---

### Button Widget Boilerplate

Verified against `elementor/includes/widgets/traits/button-trait.php` (Elementor 3.35+).

```php
protected function register_controls(): void {

    // =========================================================
    // TAB: CONTENT
    // =========================================================

    $this->start_controls_section( 'section_button', [
        'label' => esc_html__( 'Button', 'myplugin' ),
        'tab'   => \Elementor\Controls_Manager::TAB_CONTENT,
    ] );

    // Type — prefix_class writes e.g. 'elementor-button-info' on the widget wrapper.
    // Empty string = Default (no extra class added).
    $this->add_control( 'button_type', [
        'label'        => esc_html__( 'Type', 'myplugin' ),
        'type'         => \Elementor\Controls_Manager::SELECT,
        'default'      => '',
        'options'      => [
            ''        => esc_html__( 'Default', 'myplugin' ),
            'info'    => esc_html__( 'Info',    'myplugin' ),
            'success' => esc_html__( 'Success', 'myplugin' ),
            'warning' => esc_html__( 'Warning', 'myplugin' ),
            'danger'  => esc_html__( 'Danger',  'myplugin' ),
        ],
        'prefix_class' => 'elementor-button-',
    ] );

    // ✅ Elementor native default is 'Click me' — match exactly
    $this->add_control( 'text', [
        'label'       => esc_html__( 'Text', 'myplugin' ),
        'type'        => \Elementor\Controls_Manager::TEXT,
        'default'     => esc_html__( 'Click me', 'myplugin' ),
        'placeholder' => esc_html__( 'Click me', 'myplugin' ),
        'dynamic'     => [ 'active' => true ],
    ] );

    $this->add_control( 'link', [
        'label'       => esc_html__( 'Link', 'myplugin' ),
        'type'        => \Elementor\Controls_Manager::URL,
        'placeholder' => esc_html__( 'https://your-link.com', 'myplugin' ),
        'dynamic'     => [ 'active' => true ],
    ] );

    $this->add_control( 'selected_icon', [
        'label'            => esc_html__( 'Icon', 'myplugin' ),
        'type'             => \Elementor\Controls_Manager::ICONS,
        'default'          => [ 'value' => '', 'library' => '' ],
        'label_block'      => true,
        'fa4compatibility' => 'icon',
    ] );

    $this->add_control( 'icon_align', [
        'label'     => esc_html__( 'Icon Position', 'myplugin' ),
        'type'      => \Elementor\Controls_Manager::SELECT,
        'default'   => 'left',
        'options'   => [
            'left'  => esc_html__( 'Before', 'myplugin' ),
            'right' => esc_html__( 'After',  'myplugin' ),
        ],
        'condition' => [ 'selected_icon[value]!' => '' ],
    ] );

    $this->add_control( 'icon_indent', [
        'label'     => esc_html__( 'Icon Spacing', 'myplugin' ),
        'type'      => \Elementor\Controls_Manager::SLIDER,
        'default'   => [ 'size' => 8, 'unit' => 'px' ],
        'range'     => [ 'px' => [ 'min' => 0, 'max' => 50 ] ],
        'condition' => [ 'selected_icon[value]!' => '' ],
        'selectors' => [
            // ✅ These class names match Elementor's native button HTML output
            '{{WRAPPER}} .elementor-button .elementor-align-icon-right' => 'margin-left: {{SIZE}}{{UNIT}};',
            '{{WRAPPER}} .elementor-button .elementor-align-icon-left'  => 'margin-right: {{SIZE}}{{UNIT}};',
        ],
    ] );

    $this->add_control( 'button_id', [
        'label'       => esc_html__( 'Button ID', 'myplugin' ),
        'type'        => \Elementor\Controls_Manager::TEXT,
        'default'     => '',
        'description' => esc_html__( 'Please make sure the ID is unique and not used elsewhere on the page. This field allows A-z, 0-9 & underscore chars without spaces.', 'myplugin' ),
        'separator'   => 'before',
        'dynamic'     => [ 'active' => true ],
    ] );

    $this->end_controls_section();

    // =========================================================
    // TAB: STYLE
    // =========================================================

    $this->start_controls_section( 'section_style', [
        'label' => esc_html__( 'Button', 'myplugin' ),
        'tab'   => \Elementor\Controls_Manager::TAB_STYLE,
    ] );

    // ✅ Elementor native uses prefix_class for alignment, not a CSS selector.
    // 'elementor%s-button-align-' resolves to e.g. 'elementor-button-align-center'
    // on desktop and 'elementor-tablet-button-align-center' on tablet.
    $this->add_responsive_control( 'align', [
        'label'        => esc_html__( 'Alignment', 'myplugin' ),
        'type'         => \Elementor\Controls_Manager::CHOOSE,
        'options'      => [
            'left'    => [ 'title' => esc_html__( 'Left',      'myplugin' ), 'icon' => 'eicon-text-align-left'    ],
            'center'  => [ 'title' => esc_html__( 'Center',    'myplugin' ), 'icon' => 'eicon-text-align-center'  ],
            'right'   => [ 'title' => esc_html__( 'Right',     'myplugin' ), 'icon' => 'eicon-text-align-right'   ],
            'justify' => [ 'title' => esc_html__( 'Justified', 'myplugin' ), 'icon' => 'eicon-text-align-justify' ],
        ],
        'prefix_class' => 'elementor%s-button-align-',
        // ✅ No 'default' — empty inherits document flow without writing inline style
    ] );

    // ✅ Size — uses prefix_class so Elementor's own CSS defines the preset padding values
    $this->add_control( 'size', [
        'label'          => esc_html__( 'Size', 'myplugin' ),
        'type'           => \Elementor\Controls_Manager::SELECT,
        'default'        => 'sm',
        'options'        => [
            'xs' => esc_html__( 'Extra Small', 'myplugin' ),
            'sm' => esc_html__( 'Small',       'myplugin' ),
            'md' => esc_html__( 'Medium',      'myplugin' ),
            'lg' => esc_html__( 'Large',       'myplugin' ),
            'xl' => esc_html__( 'Extra Large', 'myplugin' ),
        ],
        'prefix_class'   => 'elementor-size-',
        'style_transfer' => true,
    ] );

    $this->add_group_control(
        \Elementor\Group_Control_Typography::get_type(),
        [
            'name'     => 'typography',
            'global'   => [ 'default' => \Elementor\Core\Kits\Documents\Tabs\Global_Typography::TYPOGRAPHY_ACCENT ],
            // ✅ Dual selector — targets both <a> and <button> render() output
            'selector' => '{{WRAPPER}} a.elementor-button, {{WRAPPER}} .elementor-button',
        ]
    );

    // ✅ text_shadow comes BEFORE the Normal/Hover tabs — matches Elementor native order
    $this->add_group_control(
        \Elementor\Group_Control_Text_Shadow::get_type(),
        [
            'name'     => 'text_shadow',
            'selector' => '{{WRAPPER}} a.elementor-button, {{WRAPPER}} .elementor-button',
        ]
    );

    // ── Normal / Hover tabs ───────────────────────────────────
    $this->start_controls_tabs( 'tabs_button_style' );

        $this->start_controls_tab( 'tab_button_normal', [
            'label' => esc_html__( 'Normal', 'myplugin' ),
        ] );

            $this->add_control( 'button_text_color', [
                'label'     => esc_html__( 'Text Color', 'myplugin' ),
                'type'      => \Elementor\Controls_Manager::COLOR,
                'default'   => '',
                // ✅ Dual selector matches both <a> and <button> output
                'selectors' => [
                    '{{WRAPPER}} a.elementor-button, {{WRAPPER}} .elementor-button' => 'fill: {{VALUE}}; color: {{VALUE}};',
                ],
            ] );

            $this->add_group_control(
                \Elementor\Group_Control_Background::get_type(),
                [
                    'name'           => 'background',
                    'types'          => [ 'classic', 'gradient' ],
                    'exclude'        => [ 'image' ],
                    'selector'       => '{{WRAPPER}} a.elementor-button, {{WRAPPER}} .elementor-button',
                    'fields_options' => [
                        'background' => [ 'default' => 'classic' ],
                    ],
                ]
            );

        $this->end_controls_tab();

        $this->start_controls_tab( 'tab_button_hover', [
            'label' => esc_html__( 'Hover', 'myplugin' ),
        ] );

            $this->add_control( 'hover_color', [
                'label'     => esc_html__( 'Text Color', 'myplugin' ),
                'type'      => \Elementor\Controls_Manager::COLOR,
                'default'   => '',
                'selectors' => [
                    '{{WRAPPER}} a.elementor-button:hover, {{WRAPPER}} .elementor-button:hover, {{WRAPPER}} a.elementor-button:focus, {{WRAPPER}} .elementor-button:focus' => 'color: {{VALUE}};',
                    '{{WRAPPER}} a.elementor-button:hover svg, {{WRAPPER}} .elementor-button:hover svg' => 'fill: {{VALUE}};',
                ],
            ] );

            $this->add_group_control(
                \Elementor\Group_Control_Background::get_type(),
                [
                    'name'     => 'button_background_hover',
                    'types'    => [ 'classic', 'gradient' ],
                    'exclude'  => [ 'image' ],
                    'selector' => '{{WRAPPER}} a.elementor-button:hover, {{WRAPPER}} .elementor-button:hover, {{WRAPPER}} a.elementor-button:focus, {{WRAPPER}} .elementor-button:focus',
                ]
            );

            $this->add_control( 'button_hover_border_color', [
                'label'     => esc_html__( 'Border Color', 'myplugin' ),
                'type'      => \Elementor\Controls_Manager::COLOR,
                'default'   => '',
                'condition' => [ 'border_border!' => '' ],
                'selectors' => [
                    '{{WRAPPER}} a.elementor-button:hover, {{WRAPPER}} .elementor-button:hover, {{WRAPPER}} a.elementor-button:focus, {{WRAPPER}} .elementor-button:focus' => 'border-color: {{VALUE}};',
                ],
            ] );

            // ⛔ DO NOT include hover_animation by default.
            // It loads Elementor's animation CSS library on every page using this widget.
            // Only uncomment if the client explicitly requires animation support.
            //
            // $this->add_control( 'hover_animation', [
            //     'label' => esc_html__( 'Hover Animation', 'myplugin' ),
            //     'type'  => \Elementor\Controls_Manager::HOVER_ANIMATION,
            // ] );

        $this->end_controls_tab();

    $this->end_controls_tabs();

    $this->add_group_control(
        \Elementor\Group_Control_Border::get_type(),
        [
            'name'      => 'border',
            'selector'  => '{{WRAPPER}} a.elementor-button, {{WRAPPER}} .elementor-button',
            'separator' => 'before',
        ]
    );

    $this->add_responsive_control( 'border_radius', [
        'label'      => esc_html__( 'Border Radius', 'myplugin' ),
        'type'       => \Elementor\Controls_Manager::DIMENSIONS,
        'size_units' => [ 'px', '%', 'em', 'rem' ],
        'selectors'  => [
            '{{WRAPPER}} a.elementor-button, {{WRAPPER}} .elementor-button' => 'border-radius: {{TOP}}{{UNIT}} {{RIGHT}}{{UNIT}} {{BOTTOM}}{{UNIT}} {{LEFT}}{{UNIT}};',
        ],
    ] );

    $this->add_group_control(
        \Elementor\Group_Control_Box_Shadow::get_type(),
        [
            'name'     => 'button_box_shadow',
            'selector' => '{{WRAPPER}} a.elementor-button, {{WRAPPER}} .elementor-button',
        ]
    );

    // ✅ Elementor native control ID is 'text_padding' not 'padding'
    $this->add_responsive_control( 'text_padding', [
        'label'      => esc_html__( 'Padding', 'myplugin' ),
        'type'       => \Elementor\Controls_Manager::DIMENSIONS,
        'size_units' => [ 'px', '%', 'em', 'rem' ],
        'selectors'  => [
            '{{WRAPPER}} a.elementor-button, {{WRAPPER}} .elementor-button' => 'padding: {{TOP}}{{UNIT}} {{RIGHT}}{{UNIT}} {{BOTTOM}}{{UNIT}} {{LEFT}}{{UNIT}};',
        ],
        'separator'  => 'before',
    ] );

    $this->end_controls_section();
}
```

**Matching render() skeleton:**

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

    if ( empty( $settings['text'] ) && empty( $settings['selected_icon']['value'] ) ) {
        return;
    }

    $this->add_render_attribute( 'wrapper', 'class', 'elementor-button-wrapper' );

    $this->add_render_attribute( 'button', [
        'class' => 'elementor-button',
        // ✅ Do NOT add role='button' here. When $tag resolves to <button>, the button
        // element already has an implicit role of 'button' per ARIA spec — adding it
        // explicitly is redundant. When $tag resolves to <a>, role='button' would
        // override the correct implicit role='link' semantic, breaking accessibility.
        // Source: w3.org/TR/wai-aria-1.2/#button / w3.org/TR/html-aam-1.0/#a-element
    ] );

    if ( ! empty( $settings['size'] ) ) {
        $this->add_render_attribute( 'button', 'class', 'elementor-size-' . $settings['size'] );
    }

    if ( ! empty( $settings['button_id'] ) ) {
        $this->add_render_attribute( 'button', 'id', sanitize_key( $settings['button_id'] ) );
    }

    // ⛔ hover_animation excluded by default — uncomment if control is re-enabled:
    // if ( ! empty( $settings['hover_animation'] ) ) {
    //     $this->add_render_attribute( 'button', 'class', 'elementor-animation-' . $settings['hover_animation'] );
    // }

    if ( ! empty( $settings['link']['url'] ) ) {
        $this->add_link_attributes( 'button', $settings['link'] );
        $this->add_render_attribute( 'button', 'class', 'elementor-button-link' );
        $tag = 'a';
    } else {
        $tag = 'button';
    }
    ?>
    <div <?php $this->print_render_attribute_string( 'wrapper' ); ?>>
        <<?php echo esc_attr( $tag ); ?> <?php $this->print_render_attribute_string( 'button' ); ?>>
            <span class="elementor-button-content-wrapper">
                <?php if ( ! empty( $settings['selected_icon']['value'] ) && 'left' === $settings['icon_align'] ) : ?>
                    <span class="elementor-button-icon elementor-align-icon-left">
                        <?php \Elementor\Icons_Manager::render_icon( $settings['selected_icon'], [ 'aria-hidden' => 'true' ] ); ?>
                    </span>
                <?php endif; ?>

                <?php if ( ! empty( $settings['text'] ) ) : ?>
                    <span class="elementor-button-text">
                        <?php echo esc_html( $settings['text'] ); ?>
                    </span>
                <?php endif; ?>

                <?php if ( ! empty( $settings['selected_icon']['value'] ) && 'right' === $settings['icon_align'] ) : ?>
                    <span class="elementor-button-icon elementor-align-icon-right">
                        <?php \Elementor\Icons_Manager::render_icon( $settings['selected_icon'], [ 'aria-hidden' => 'true' ] ); ?>
                    </span>
                <?php endif; ?>
            </span>
        </<?php echo esc_attr( $tag ); ?>>
    </div>
    <?php
}

protected function content_template(): void {
    ?>
    <#
    var iconHTML  = elementor.helpers.renderIcon( view, settings.selected_icon, { 'aria-hidden': true }, 'i', 'object' );
    var hasIcon   = iconHTML && iconHTML.rendered;
    var tag       = settings.link && settings.link.url ? 'a' : 'button';
    var sizeClass = settings.size ? 'elementor-size-' + settings.size : '';
    // ⛔ hover_animation excluded — uncomment if control is re-enabled:
    // var animClass = settings.hover_animation ? 'elementor-animation-' + settings.hover_animation : '';
    #>
    <div class="elementor-button-wrapper">
        <{{{ tag }}} class="elementor-button {{{ sizeClass }}}">
            <span class="elementor-button-content-wrapper">
                <# if ( hasIcon && 'left' === settings.icon_align ) { #>
                    <span class="elementor-button-icon elementor-align-icon-left">{{{ iconHTML.value }}}</span>
                <# } #>
                <# if ( settings.text ) { #>
                    <span class="elementor-button-text">{{{ settings.text }}}</span>
                <# } #>
                <# if ( hasIcon && 'right' === settings.icon_align ) { #>
                    <span class="elementor-button-icon elementor-align-icon-right">{{{ iconHTML.value }}}</span>
                <# } #>
            </span>
        </{{{ tag }}}>
    </div>
    <?php
}
```

> **Remove checklist:**
> - `button_type` → remove if no semantic type variants needed
> - `size` → remove if button size is controlled purely via `text_padding`
> - `selected_icon` + `icon_align` + `icon_indent` → remove if no icon support needed
> - `button_id` → remove if button does not need JS/anchor targeting
> - `hover_animation` → **excluded by default** (commented out)
> - `button_hover_border_color` → remove if no border is used
> - `text_shadow` → remove for simple flat-design buttons

---
