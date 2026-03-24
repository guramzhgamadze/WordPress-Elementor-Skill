# Widget Boilerplate — Icon Box

> **When to use this file:** Load whenever building a widget combining an icon with a title and description.
> Verified against `elementor/includes/widgets/icon-box.php` (Elementor 3.35+).

---

```php
protected function register_controls(): void {

    // ── CONTENT ───────────────────────────────────────────────
    $this->start_controls_section( 'section_icon_box', [
        'label' => esc_html__( 'Icon Box', 'myplugin' ),
        'tab'   => \Elementor\Controls_Manager::TAB_CONTENT,
    ] );

    $this->add_control( 'selected_icon', [
        'label'            => esc_html__( 'Icon', 'myplugin' ),
        'type'             => \Elementor\Controls_Manager::ICONS,
        'fa4compatibility' => 'icon',
        'default'          => [ 'value' => 'fas fa-star', 'library' => 'fa-solid' ],
    ] );

    $this->add_control( 'title_text', [
        'label'       => esc_html__( 'Title & Description', 'myplugin' ),
        'type'        => \Elementor\Controls_Manager::TEXT,
        'default'     => esc_html__( 'This is the heading', 'myplugin' ),
        'placeholder' => esc_html__( 'Enter your title', 'myplugin' ),
        'label_block' => true,
        'dynamic'     => [ 'active' => true ],
    ] );

    $this->add_control( 'description_text', [
        'label'       => '',
        'type'        => \Elementor\Controls_Manager::TEXTAREA,
        'default'     => esc_html__( 'Lorem ipsum dolor sit amet, consectetur adipiscing elit. Ut elit tellus, luctus nec ullamcorper mattis pulvinar.', 'myplugin' ),
        'placeholder' => esc_html__( 'Enter your description', 'myplugin' ),
        'rows'        => 10,
        'dynamic'     => [ 'active' => true ],
    ] );

    $this->add_control( 'link', [
        'label'     => esc_html__( 'Link', 'myplugin' ),
        'type'      => \Elementor\Controls_Manager::URL,
        'dynamic'   => [ 'active' => true ],
        'separator' => 'before',
    ] );

    $this->add_control( 'title_size', [
        'label'   => esc_html__( 'Title HTML Tag', 'myplugin' ),
        'type'    => \Elementor\Controls_Manager::SELECT,
        'options' => [
            'h1' => 'H1', 'h2' => 'H2', 'h3' => 'H3',
            'h4' => 'H4', 'h5' => 'H5', 'h6' => 'H6',
            'div' => 'div', 'span' => 'span', 'p' => 'p',
        ],
        'default' => 'h3',
    ] );

    $this->end_controls_section();

    // ── STYLE: Icon ───────────────────────────────────────────
    $this->start_controls_section( 'section_style_icon', [
        'label' => esc_html__( 'Icon', 'myplugin' ),
        'tab'   => \Elementor\Controls_Manager::TAB_STYLE,
    ] );

    $this->add_responsive_control( 'position', [
        'label'   => esc_html__( 'Position', 'myplugin' ),
        'type'    => \Elementor\Controls_Manager::CHOOSE,
        'default' => 'top',
        'options' => [
            'left'  => [ 'title' => esc_html__( 'Left',  'myplugin' ), 'icon' => 'eicon-h-align-left'  ],
            'top'   => [ 'title' => esc_html__( 'Top',   'myplugin' ), 'icon' => 'eicon-v-align-top'   ],
            'right' => [ 'title' => esc_html__( 'Right', 'myplugin' ), 'icon' => 'eicon-h-align-right' ],
        ],
        'prefix_class' => 'elementor-position%s-',
    ] );

    $this->add_responsive_control( 'icon_size', [
        'label'     => esc_html__( 'Size', 'myplugin' ),
        'type'      => \Elementor\Controls_Manager::SLIDER,
        'range'     => [ 'px' => [ 'min' => 6, 'max' => 300 ] ],
        'default'   => [ 'size' => 50, 'unit' => 'px' ],
        'selectors' => [
            '{{WRAPPER}} .myplugin-icon-box-icon'     => 'font-size: {{SIZE}}{{UNIT}};',
            '{{WRAPPER}} .myplugin-icon-box-icon svg' => 'width: {{SIZE}}{{UNIT}}; height: {{SIZE}}{{UNIT}};',
        ],
    ] );

    $this->start_controls_tabs( 'icon_colors' );

        $this->start_controls_tab( 'icon_colors_normal', [ 'label' => esc_html__( 'Normal', 'myplugin' ) ] );

            $this->add_control( 'primary_color', [
                'label'     => esc_html__( 'Primary Color', 'myplugin' ),
                'type'      => \Elementor\Controls_Manager::COLOR,
                'global'    => [ 'default' => \Elementor\Core\Kits\Documents\Tabs\Global_Colors::COLOR_PRIMARY ],
                'selectors' => [
                    '{{WRAPPER}} .myplugin-icon-box-icon i'   => 'color: {{VALUE}};',
                    '{{WRAPPER}} .myplugin-icon-box-icon svg' => 'fill: {{VALUE}};',
                ],
            ] );

        $this->end_controls_tab();

        $this->start_controls_tab( 'icon_colors_hover', [ 'label' => esc_html__( 'Hover', 'myplugin' ) ] );

            $this->add_control( 'hover_primary_color', [
                'label'     => esc_html__( 'Primary Color', 'myplugin' ),
                'type'      => \Elementor\Controls_Manager::COLOR,
                'selectors' => [
                    '{{WRAPPER}}:hover .myplugin-icon-box-icon i'   => 'color: {{VALUE}};',
                    '{{WRAPPER}}:hover .myplugin-icon-box-icon svg' => 'fill: {{VALUE}};',
                ],
            ] );

        $this->end_controls_tab();

    $this->end_controls_tabs();

    $this->end_controls_section();

    // ── STYLE: Content ────────────────────────────────────────
    $this->start_controls_section( 'section_style_content', [
        'label' => esc_html__( 'Content', 'myplugin' ),
        'tab'   => \Elementor\Controls_Manager::TAB_STYLE,
    ] );

    $this->add_responsive_control( 'text_align', [
        'label'     => esc_html__( 'Alignment', 'myplugin' ),
        'type'      => \Elementor\Controls_Manager::CHOOSE,
        'options'   => [
            'left'   => [ 'title' => esc_html__( 'Left',   'myplugin' ), 'icon' => 'eicon-text-align-left'   ],
            'center' => [ 'title' => esc_html__( 'Center', 'myplugin' ), 'icon' => 'eicon-text-align-center' ],
            'right'  => [ 'title' => esc_html__( 'Right',  'myplugin' ), 'icon' => 'eicon-text-align-right'  ],
        ],
        'selectors' => [ '{{WRAPPER}} .myplugin-icon-box-content' => 'text-align: {{VALUE}};' ],
    ] );

    $this->add_control( 'heading_title', [
        'label'     => esc_html__( 'Title', 'myplugin' ),
        'type'      => \Elementor\Controls_Manager::HEADING,
        'separator' => 'before',
    ] );

    $this->add_control( 'title_color', [
        'label'     => esc_html__( 'Color', 'myplugin' ),
        'type'      => \Elementor\Controls_Manager::COLOR,
        'global'    => [ 'default' => \Elementor\Core\Kits\Documents\Tabs\Global_Colors::COLOR_PRIMARY ],
        'selectors' => [ '{{WRAPPER}} .myplugin-icon-box-title, {{WRAPPER}} .myplugin-icon-box-title a' => 'color: {{VALUE}};' ],
    ] );

    $this->add_group_control(
        \Elementor\Group_Control_Typography::get_type(),
        [
            'name'     => 'title_typography',
            'global'   => [ 'default' => \Elementor\Core\Kits\Documents\Tabs\Global_Typography::TYPOGRAPHY_PRIMARY ],
            'selector' => '{{WRAPPER}} .myplugin-icon-box-title',
        ]
    );

    $this->add_control( 'heading_description', [
        'label'     => esc_html__( 'Description', 'myplugin' ),
        'type'      => \Elementor\Controls_Manager::HEADING,
        'separator' => 'before',
    ] );

    $this->add_control( 'description_color', [
        'label'     => esc_html__( 'Color', 'myplugin' ),
        'type'      => \Elementor\Controls_Manager::COLOR,
        'global'    => [ 'default' => \Elementor\Core\Kits\Documents\Tabs\Global_Colors::COLOR_TEXT ],
        'selectors' => [ '{{WRAPPER}} .myplugin-icon-box-description' => 'color: {{VALUE}};' ],
    ] );

    $this->add_group_control(
        \Elementor\Group_Control_Typography::get_type(),
        [
            'name'     => 'description_typography',
            'global'   => [ 'default' => \Elementor\Core\Kits\Documents\Tabs\Global_Typography::TYPOGRAPHY_TEXT ],
            'selector' => '{{WRAPPER}} .myplugin-icon-box-description',
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
    $tag      = \Elementor\Utils::validate_html_tag( $settings['title_size'] );

    $title = $settings['title_text'];
    if ( ! empty( $settings['link']['url'] ) ) {
        $this->add_link_attributes( 'link', $settings['link'] );
        $title = '<a ' . $this->get_render_attribute_string( 'link' ) . '>' . esc_html( $title ) . '</a>';
    }
    ?>
    <div class="myplugin-icon-box">
        <div class="myplugin-icon-box-icon">
            <?php \Elementor\Icons_Manager::render_icon( $settings['selected_icon'], [ 'aria-hidden' => 'true' ] ); ?>
        </div>
        <div class="myplugin-icon-box-content">
            <<?php echo esc_attr( $tag ); ?> class="myplugin-icon-box-title">
                <?php echo wp_kses_post( $title ); ?>
            </<?php echo esc_attr( $tag ); ?>>
            <?php if ( ! empty( $settings['description_text'] ) ) : ?>
                <p class="myplugin-icon-box-description"><?php echo wp_kses_post( $settings['description_text'] ); ?></p>
            <?php endif; ?>
        </div>
    </div>
    <?php
}
```

> **Remove checklist:**
> - `link` → remove if the box is never clickable
> - `position` → remove if icon is always top-aligned
> - Hover color tabs → remove if no hover state needed


**content_template() skeleton:**

```php
protected function content_template(): void {
    ?>
    <#
    var iconHTML = elementor.helpers.renderIcon( view, settings.selected_icon, { 'aria-hidden': true }, 'i', 'object' );
    var tag = elementor.helpers.validateHTMLTag( settings.title_size );
    var title = settings.title_text;
    if ( settings.link && settings.link.url ) {
        // ✅ Official Elementor pattern — link.url used directly per official advanced example
        title = '<a href="' + settings.link.url + '">' + title + '</a>';
    }
    #>
    <div class="myplugin-icon-box">
        <div class="myplugin-icon-box-icon">{{{ iconHTML.value }}}</div>
        <div class="myplugin-icon-box-content">
            <{{{ tag }}} class="myplugin-icon-box-title">{{{ title }}}</{{{ tag }}}>
            <# if ( settings.description_text ) { #>
                <p class="myplugin-icon-box-description">{{{ settings.description_text }}}</p>
            <# } #>
        </div>
    </div>
    <?php
}
```
