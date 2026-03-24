# Widget Boilerplate — Container

> **When to use this file:** Load this boilerplate whenever building a widget that acts as a
> layout wrapper — a section, card shell, hero block, or any element that wraps other content.
>
> Mirrors Elementor's native Container (Flexbox) panel exactly in control structure and order.
> Use these controls inside `register_controls()` of your `Widget_Base` subclass.

---

### Container Widget Boilerplate

Mirrors Elementor's native Container (Flexbox) panel. Use whenever a widget acts as a
layout wrapper — a section, card shell, hero block, or any element that contains other
widgets or content groups.

> ⚠️ This boilerplate targets Elementor's Container element structure. When building a
> custom widget that internally renders a container-like wrapper, use these controls inside
> `register_controls()` of your `Widget_Base` subclass to match the UX pattern users already
> know from native containers.

```php
protected function register_controls(): void {

    // =========================================================
    // TAB: LAYOUT
    // =========================================================

    // ── LAYOUT ────────────────────────────────────────────────
    // ✅ TAB_LAYOUT is NOT available in Widget_Base — it exists only on the native Container/Section
    // element. Custom widgets only support TAB_CONTENT, TAB_STYLE, TAB_ADVANCED.
    // Use TAB_CONTENT for layout controls in custom widgets.
    $this->start_controls_section( 'section_layout', [
        'label' => esc_html__( 'Container', 'myplugin' ),
        'tab'   => \Elementor\Controls_Manager::TAB_CONTENT,
    ] );

    $this->add_control( 'container_layout', [
        'label'   => esc_html__( 'Container Layout', 'myplugin' ),
        'type'    => \Elementor\Controls_Manager::SELECT,
        'default' => 'flexbox',
        'options' => [
            'flexbox' => esc_html__( 'Flexbox', 'myplugin' ),
            // 'grid' => esc_html__( 'Grid', 'myplugin' ), // Elementor Pro 3.16+ only
        ],
    ] );

    $this->add_control( 'content_width', [
        'label'   => esc_html__( 'Content Width', 'myplugin' ),
        'type'    => \Elementor\Controls_Manager::SELECT,
        'default' => 'full',
        'options' => [
            'full'  => esc_html__( 'Full Width', 'myplugin' ),
            'boxed' => esc_html__( 'Boxed',      'myplugin' ),
        ],
        'selectors_dictionary' => [
            'full'  => '100%',
            'boxed' => 'var(--container-max-width, 1140px)',
        ],
        'selectors' => [
            '{{WRAPPER}} .myplugin-container__inner' => 'max-width: {{VALUE}};',
        ],
    ] );

    $this->add_responsive_control( 'width', [
        'label'      => esc_html__( 'Width', 'myplugin' ),
        'type'       => \Elementor\Controls_Manager::SLIDER,
        'default'    => [ 'size' => 90, 'unit' => '%' ],
        'size_units' => [ 'px', '%', 'vw' ],
        'range'      => [
            'px' => [ 'min' => 0, 'max' => 2000 ],
            '%'  => [ 'min' => 0, 'max' => 100  ],
            'vw' => [ 'min' => 0, 'max' => 100  ],
        ],
        'selectors'  => [
            '{{WRAPPER}} .myplugin-container' => 'width: {{SIZE}}{{UNIT}};',
        ],
    ] );

    $this->add_responsive_control( 'min_height', [
        'label'       => esc_html__( 'Min Height', 'myplugin' ),
        'type'        => \Elementor\Controls_Manager::SLIDER,
        'default'     => [ 'size' => 100, 'unit' => 'vh' ],
        'size_units'  => [ 'px', 'vh', 'vw', 'em', 'rem' ],
        'range'       => [
            'px' => [ 'min' => 0, 'max' => 1500 ],
            'vh' => [ 'min' => 0, 'max' => 100  ],
            'vw' => [ 'min' => 0, 'max' => 100  ],
        ],
        'selectors'   => [
            '{{WRAPPER}} .myplugin-container' => 'min-height: {{SIZE}}{{UNIT}};',
        ],
        'description' => esc_html__( 'To achieve full height Container use 100vh.', 'myplugin' ),
    ] );

    $this->add_control( 'items_heading', [
        'label'     => esc_html__( 'Items', 'myplugin' ),
        'type'      => \Elementor\Controls_Manager::HEADING,
        'separator' => 'before',
    ] );

    $this->add_responsive_control( 'flex_direction', [
        'label'     => esc_html__( 'Direction', 'myplugin' ),
        'type'      => \Elementor\Controls_Manager::CHOOSE,
        'default'   => 'row',
        'options'   => [
            'row'            => [ 'title' => esc_html__( 'Row',            'myplugin' ), 'icon' => 'eicon-arrow-right' ],
            'column'         => [ 'title' => esc_html__( 'Column',         'myplugin' ), 'icon' => 'eicon-arrow-down'  ],
            'row-reverse'    => [ 'title' => esc_html__( 'Row Reverse',    'myplugin' ), 'icon' => 'eicon-arrow-left'  ],
            'column-reverse' => [ 'title' => esc_html__( 'Column Reverse', 'myplugin' ), 'icon' => 'eicon-arrow-up'    ],
        ],
        'selectors' => [
            '{{WRAPPER}} .myplugin-container' => 'flex-direction: {{VALUE}};',
        ],
    ] );

    $this->add_responsive_control( 'justify_content', [
        'label'     => esc_html__( 'Justify Content', 'myplugin' ),
        'type'      => \Elementor\Controls_Manager::CHOOSE,
        'default'   => 'flex-start',
        'options'   => [
            'flex-start'    => [ 'title' => esc_html__( 'Start',         'myplugin' ), 'icon' => 'eicon-flex eicon-justify-start-h'         ],
            'center'        => [ 'title' => esc_html__( 'Center',        'myplugin' ), 'icon' => 'eicon-flex eicon-justify-center-h'        ],
            'flex-end'      => [ 'title' => esc_html__( 'End',           'myplugin' ), 'icon' => 'eicon-flex eicon-justify-end-h'           ],
            'space-between' => [ 'title' => esc_html__( 'Space Between', 'myplugin' ), 'icon' => 'eicon-flex eicon-justify-space-between-h' ],
            'space-around'  => [ 'title' => esc_html__( 'Space Around',  'myplugin' ), 'icon' => 'eicon-flex eicon-justify-space-around-h'  ],
            'space-evenly'  => [ 'title' => esc_html__( 'Space Evenly',  'myplugin' ), 'icon' => 'eicon-flex eicon-justify-space-evenly-h'  ],
        ],
        'selectors' => [
            '{{WRAPPER}} .myplugin-container' => 'justify-content: {{VALUE}};',
        ],
    ] );

    $this->add_responsive_control( 'align_items', [
        'label'     => esc_html__( 'Align Items', 'myplugin' ),
        'type'      => \Elementor\Controls_Manager::CHOOSE,
        'default'   => 'flex-start',
        'options'   => [
            'flex-start' => [ 'title' => esc_html__( 'Start',   'myplugin' ), 'icon' => 'eicon-flex eicon-align-start-v'   ],
            'center'     => [ 'title' => esc_html__( 'Center',  'myplugin' ), 'icon' => 'eicon-flex eicon-align-center-v'  ],
            'flex-end'   => [ 'title' => esc_html__( 'End',     'myplugin' ), 'icon' => 'eicon-flex eicon-align-end-v'     ],
            'stretch'    => [ 'title' => esc_html__( 'Stretch', 'myplugin' ), 'icon' => 'eicon-flex eicon-align-stretch-v' ],
        ],
        'selectors' => [
            '{{WRAPPER}} .myplugin-container' => 'align-items: {{VALUE}};',
        ],
    ] );

    // ✅ Two separate gap controls — universally safe across all Elementor versions.
    // Controls_Manager::GAPS exists since 3.7 but its ROW.SIZE/COLUMN.SIZE selector tokens
    // are not documented in official Elementor developer docs. Use separate sliders instead.
    $this->add_responsive_control( 'column_gap', [
        'label'      => esc_html__( 'Column Gap', 'myplugin' ),
        'type'       => \Elementor\Controls_Manager::SLIDER,
        'default'    => [ 'size' => 20, 'unit' => 'px' ],
        'size_units' => [ 'px', '%', 'em', 'rem', 'vw' ],
        'range'      => [
            'px' => [ 'min' => 0, 'max' => 200 ],
            '%'  => [ 'min' => 0, 'max' => 100  ],
        ],
        'selectors'  => [
            '{{WRAPPER}} .myplugin-container' => 'column-gap: {{SIZE}}{{UNIT}};',
        ],
    ] );

    $this->add_responsive_control( 'row_gap', [
        'label'      => esc_html__( 'Row Gap', 'myplugin' ),
        'type'       => \Elementor\Controls_Manager::SLIDER,
        'default'    => [ 'size' => 20, 'unit' => 'px' ],
        'size_units' => [ 'px', '%', 'em', 'rem', 'vw' ],
        'range'      => [
            'px' => [ 'min' => 0, 'max' => 200 ],
            '%'  => [ 'min' => 0, 'max' => 100  ],
        ],
        'selectors'  => [
            '{{WRAPPER}} .myplugin-container' => 'row-gap: {{SIZE}}{{UNIT}};',
        ],
    ] );

    $this->add_responsive_control( 'flex_wrap', [
        'label'       => esc_html__( 'Wrap', 'myplugin' ),
        'type'        => \Elementor\Controls_Manager::CHOOSE,
        'default'     => 'nowrap',
        'options'     => [
            'nowrap' => [ 'title' => esc_html__( 'No Wrap', 'myplugin' ), 'icon' => 'eicon-flex eicon-nowrap' ],
            'wrap'   => [ 'title' => esc_html__( 'Wrap',    'myplugin' ), 'icon' => 'eicon-flex eicon-wrap'   ],
        ],
        'selectors'   => [
            '{{WRAPPER}} .myplugin-container' => 'flex-wrap: {{VALUE}};',
        ],
        'description' => esc_html__( 'Items within the container can stay in a single line (No wrap), or break into multiple lines (Wrap).', 'myplugin' ),
    ] );

    $this->end_controls_section();

    $this->start_controls_section( 'section_additional', [
        'label' => esc_html__( 'Additional Options', 'myplugin' ),
        'tab'   => \Elementor\Controls_Manager::TAB_CONTENT,
    ] );
    // Add widget-specific layout extras here (overflow, z-index, etc.)
    $this->end_controls_section();

    // =========================================================
    // TAB: STYLE
    // =========================================================

    $this->start_controls_section( 'section_background', [
        'label' => esc_html__( 'Background', 'myplugin' ),
        'tab'   => \Elementor\Controls_Manager::TAB_STYLE,
    ] );

    $this->start_controls_tabs( 'tabs_background' );

        $this->start_controls_tab( 'tab_background_normal', [
            'label' => esc_html__( 'Normal', 'myplugin' ),
        ] );

            $this->add_group_control(
                \Elementor\Group_Control_Background::get_type(),
                [
                    'name'     => 'background',
                    // ✅ 'classic' and 'gradient' only — per official Elementor docs:
                    // "Video and slideshow types are supported only at section/container
                    // level, not widget level."
                    // Source: developers.elementor.com/docs/editor-controls/group-control-background/
                    'types'    => [ 'classic', 'gradient' ],
                    'selector' => '{{WRAPPER}} .myplugin-container',
                ]
            );

        $this->end_controls_tab();

        $this->start_controls_tab( 'tab_background_hover', [
            'label' => esc_html__( 'Hover', 'myplugin' ),
        ] );

            $this->add_group_control(
                \Elementor\Group_Control_Background::get_type(),
                [
                    'name'     => 'background_hover',
                    'types'    => [ 'classic', 'gradient' ],
                    'selector' => '{{WRAPPER}} .myplugin-container:hover',
                ]
            );

        $this->end_controls_tab();

    $this->end_controls_tabs();

    // ✅ Scrolling Effects & Mouse Effects — Elementor Pro only.
    // These controls register fine on free Elementor but have no effect without Pro.
    // Remove entirely if plugin targets free Elementor only.
    $this->add_control( 'scrolling_effects', [
        'label'       => esc_html__( 'Scrolling Effects', 'myplugin' ),
        'type'        => \Elementor\Controls_Manager::SWITCHER,
        'default'     => '',
        'separator'   => 'before',
        // ⚠️ PRO ONLY
        'description' => esc_html__( 'Requires Elementor Pro.', 'myplugin' ),
    ] );

    $this->add_control( 'mouse_effects', [
        'label'       => esc_html__( 'Mouse Effects', 'myplugin' ),
        'type'        => \Elementor\Controls_Manager::SWITCHER,
        'default'     => '',
        // ⚠️ PRO ONLY
        'description' => esc_html__( 'Requires Elementor Pro.', 'myplugin' ),
    ] );

    $this->end_controls_section();

    $this->start_controls_section( 'section_background_overlay', [
        'label' => esc_html__( 'Background Overlay', 'myplugin' ),
        'tab'   => \Elementor\Controls_Manager::TAB_STYLE,
    ] );

    $this->start_controls_tabs( 'tabs_background_overlay' );

        $this->start_controls_tab( 'tab_background_overlay_normal', [
            'label' => esc_html__( 'Normal', 'myplugin' ),
        ] );

            $this->add_group_control(
                \Elementor\Group_Control_Background::get_type(),
                [
                    'name'     => 'background_overlay',
                    'types'    => [ 'classic', 'gradient' ],
                    'selector' => '{{WRAPPER}} .myplugin-container__overlay',
                ]
            );

        $this->end_controls_tab();

        $this->start_controls_tab( 'tab_background_overlay_hover', [
            'label' => esc_html__( 'Hover', 'myplugin' ),
        ] );

            $this->add_group_control(
                \Elementor\Group_Control_Background::get_type(),
                [
                    'name'     => 'background_overlay_hover',
                    'types'    => [ 'classic', 'gradient' ],
                    'selector' => '{{WRAPPER}} .myplugin-container:hover .myplugin-container__overlay',
                ]
            );

        $this->end_controls_tab();

    $this->end_controls_tabs();

    $this->end_controls_section();

    $this->start_controls_section( 'section_border', [
        'label' => esc_html__( 'Border', 'myplugin' ),
        'tab'   => \Elementor\Controls_Manager::TAB_STYLE,
    ] );

    $this->start_controls_tabs( 'tabs_border' );

        $this->start_controls_tab( 'tab_border_normal', [
            'label' => esc_html__( 'Normal', 'myplugin' ),
        ] );

            $this->add_group_control(
                \Elementor\Group_Control_Border::get_type(),
                [
                    'name'     => 'border',
                    'selector' => '{{WRAPPER}} .myplugin-container',
                ]
            );

            // ✅ Default 16px all sides matches Elementor's native container default
            $this->add_responsive_control( 'border_radius', [
                'label'      => esc_html__( 'Border Radius', 'myplugin' ),
                'type'       => \Elementor\Controls_Manager::DIMENSIONS,
                'size_units' => [ 'px', '%', 'em', 'rem' ],
                'default'    => [
                    'top'      => '16',
                    'right'    => '16',
                    'bottom'   => '16',
                    'left'     => '16',
                    'unit'     => 'px',
                    'isLinked' => true,
                ],
                'selectors'  => [
                    '{{WRAPPER}} .myplugin-container' => 'border-radius: {{TOP}}{{UNIT}} {{RIGHT}}{{UNIT}} {{BOTTOM}}{{UNIT}} {{LEFT}}{{UNIT}};',
                ],
            ] );

            $this->add_group_control(
                \Elementor\Group_Control_Box_Shadow::get_type(),
                [
                    'name'     => 'box_shadow',
                    'selector' => '{{WRAPPER}} .myplugin-container',
                ]
            );

        $this->end_controls_tab();

        $this->start_controls_tab( 'tab_border_hover', [
            'label' => esc_html__( 'Hover', 'myplugin' ),
        ] );

            $this->add_group_control(
                \Elementor\Group_Control_Border::get_type(),
                [
                    'name'     => 'border_hover',
                    'selector' => '{{WRAPPER}} .myplugin-container:hover',
                ]
            );

            $this->add_responsive_control( 'border_radius_hover', [
                'label'      => esc_html__( 'Border Radius', 'myplugin' ),
                'type'       => \Elementor\Controls_Manager::DIMENSIONS,
                'size_units' => [ 'px', '%', 'em', 'rem' ],
                'selectors'  => [
                    '{{WRAPPER}} .myplugin-container:hover' => 'border-radius: {{TOP}}{{UNIT}} {{RIGHT}}{{UNIT}} {{BOTTOM}}{{UNIT}} {{LEFT}}{{UNIT}};',
                ],
            ] );

            $this->add_group_control(
                \Elementor\Group_Control_Box_Shadow::get_type(),
                [
                    'name'     => 'box_shadow_hover',
                    'selector' => '{{WRAPPER}} .myplugin-container:hover',
                ]
            );

        $this->end_controls_tab();

    $this->end_controls_tabs();

    $this->end_controls_section();

    // Shape Divider has no native Controls_Manager API. Implement via CSS or SVG in render().
    $this->start_controls_section( 'section_shape_divider', [
        'label' => esc_html__( 'Shape Divider', 'myplugin' ),
        'tab'   => \Elementor\Controls_Manager::TAB_STYLE,
    ] );

    $this->add_control( 'shape_divider_notice', [
        'type'            => \Elementor\Controls_Manager::RAW_HTML,
        'raw'             => esc_html__( 'Shape dividers are implemented via CSS or inline SVG in render(). Add your shape controls here if needed.', 'myplugin' ),
        'content_classes' => 'elementor-descriptor',
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

    $this->add_render_attribute( 'container', 'class', 'myplugin-container' );
    ?>
    <div <?php $this->print_render_attribute_string( 'container' ); ?>>
        <div class="myplugin-container__overlay"></div>
        <div class="myplugin-container__inner">
            <?php // Render child content here ?>
        </div>
    </div>
    <?php
}
```

> **Remove checklist:**
> - `container_layout` → remove if widget only ever uses Flexbox
> - `content_width` + inner wrapper div → remove if container is always full width
> - `min_height` → remove for widgets that size naturally to their content
> - `flex_wrap` → remove if items must always stay single line
> - `scrolling_effects` + `mouse_effects` → remove if Elementor Pro is not required
> - `section_background_overlay` + overlay div → remove if no overlay needed
> - `section_shape_divider` → remove if no shape divider needed
> - Border Radius default 16px → adjust or clear for sharp-corner designs

---


**content_template() skeleton:**

```php
protected function content_template(): void {
    ?>
    <div class="myplugin-container">
        <div class="myplugin-container__overlay"></div>
        <div class="myplugin-container__inner"></div>
    </div>
    <?php
}
```
