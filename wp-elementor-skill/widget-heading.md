# Widget Boilerplate — Heading

> **When to use this file:** Load whenever building a widget with a primary headline/title element.
> Verified against `elementor/includes/widgets/heading.php` (Elementor 3.35+).

---

```php
protected function register_controls(): void {

    // ── CONTENT ───────────────────────────────────────────────
    $this->start_controls_section( 'section_title', [
        'label' => esc_html__( 'Title', 'myplugin' ),
        'tab'   => \Elementor\Controls_Manager::TAB_CONTENT,
    ] );

    $this->add_control( 'title', [
        'label'       => esc_html__( 'Title', 'myplugin' ),
        'type'        => \Elementor\Controls_Manager::TEXTAREA,
        'dynamic'     => [ 'active' => true ],
        'placeholder' => esc_html__( 'Enter your title', 'myplugin' ),
        'default'     => esc_html__( 'Add Your Heading Text Here', 'myplugin' ),
    ] );

    $this->add_control( 'link', [
        'label'     => esc_html__( 'Link', 'myplugin' ),
        'type'      => \Elementor\Controls_Manager::URL,
        'dynamic'   => [ 'active' => true ],
        'separator' => 'before',
    ] );

    $this->add_control( 'size', [
        'label'   => esc_html__( 'Size', 'myplugin' ),
        'type'    => \Elementor\Controls_Manager::SELECT,
        'default' => 'default',
        'options' => [
            'default' => esc_html__( 'Default', 'myplugin' ),
            'small'   => esc_html__( 'Small',   'myplugin' ),
            'medium'  => esc_html__( 'Medium',  'myplugin' ),
            'large'   => esc_html__( 'Large',   'myplugin' ),
            'xl'      => esc_html__( 'XL',      'myplugin' ),
            'xxl'     => esc_html__( 'XXL',     'myplugin' ),
        ],
    ] );

    // ✅ header_size = the HTML tag — h1–h6, div, span, p
    $this->add_control( 'header_size', [
        'label'   => esc_html__( 'HTML Tag', 'myplugin' ),
        'type'    => \Elementor\Controls_Manager::SELECT,
        'options' => [
            'h1'   => 'H1',
            'h2'   => 'H2',
            'h3'   => 'H3',
            'h4'   => 'H4',
            'h5'   => 'H5',
            'h6'   => 'H6',
            'div'  => 'div',
            'span' => 'span',
            'p'    => 'p',
        ],
        'default' => 'h2',
    ] );

    $this->end_controls_section();

    // ── STYLE ─────────────────────────────────────────────────
    $this->start_controls_section( 'section_title_style', [
        'label' => esc_html__( 'Title', 'myplugin' ),
        'tab'   => \Elementor\Controls_Manager::TAB_STYLE,
    ] );

    $this->add_responsive_control( 'align', [
        'label'     => esc_html__( 'Alignment', 'myplugin' ),
        'type'      => \Elementor\Controls_Manager::CHOOSE,
        'options'   => [
            'left'    => [ 'title' => esc_html__( 'Left',    'myplugin' ), 'icon' => 'eicon-text-align-left'    ],
            'center'  => [ 'title' => esc_html__( 'Center',  'myplugin' ), 'icon' => 'eicon-text-align-center'  ],
            'right'   => [ 'title' => esc_html__( 'Right',   'myplugin' ), 'icon' => 'eicon-text-align-right'   ],
            'justify' => [ 'title' => esc_html__( 'Justify', 'myplugin' ), 'icon' => 'eicon-text-align-justify' ],
        ],
        'selectors' => [ '{{WRAPPER}} .myplugin-heading' => 'text-align: {{VALUE}};' ],
    ] );

    $this->add_control( 'title_color', [
        'label'     => esc_html__( 'Text Color', 'myplugin' ),
        'type'      => \Elementor\Controls_Manager::COLOR,
        'global'    => [ 'default' => \Elementor\Core\Kits\Documents\Tabs\Global_Colors::COLOR_PRIMARY ],
        'selectors' => [ '{{WRAPPER}} .myplugin-heading, {{WRAPPER}} .myplugin-heading a' => 'color: {{VALUE}};' ],
    ] );

    $this->add_group_control(
        \Elementor\Group_Control_Typography::get_type(),
        [
            'name'     => 'typography',
            'global'   => [ 'default' => \Elementor\Core\Kits\Documents\Tabs\Global_Typography::TYPOGRAPHY_PRIMARY ],
            'selector' => '{{WRAPPER}} .myplugin-heading',
        ]
    );

    $this->add_group_control(
        \Elementor\Group_Control_Text_Shadow::get_type(),
        [
            'name'     => 'text_shadow',
            'selector' => '{{WRAPPER}} .myplugin-heading',
        ]
    );

    $this->add_control( 'blend_mode', [
        'label'     => esc_html__( 'Blend Mode', 'myplugin' ),
        'type'      => \Elementor\Controls_Manager::SELECT,
        'options'   => [
            ''            => esc_html__( 'Normal',      'myplugin' ),
            'multiply'    => esc_html__( 'Multiply',    'myplugin' ),
            'screen'      => esc_html__( 'Screen',      'myplugin' ),
            'overlay'     => esc_html__( 'Overlay',     'myplugin' ),
            'darken'      => esc_html__( 'Darken',      'myplugin' ),
            'lighten'     => esc_html__( 'Lighten',     'myplugin' ),
            'color-dodge' => esc_html__( 'Color Dodge', 'myplugin' ),
            'saturation'  => esc_html__( 'Saturation',  'myplugin' ),
            'color'       => esc_html__( 'Color',       'myplugin' ),
            'difference'  => esc_html__( 'Difference',  'myplugin' ),
            'exclusion'   => esc_html__( 'Exclusion',   'myplugin' ),
            'hue'         => esc_html__( 'Hue',         'myplugin' ),
            'luminosity'  => esc_html__( 'Luminosity',  'myplugin' ),
        ],
        'selectors' => [ '{{WRAPPER}} .myplugin-heading' => 'mix-blend-mode: {{VALUE}};' ],
        'separator' => 'none',
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

    if ( empty( $settings['title'] ) ) {
        return;
    }

    $this->add_render_attribute( 'title', 'class', 'myplugin-heading' );

    if ( ! empty( $settings['size'] ) && 'default' !== $settings['size'] ) {
        $this->add_render_attribute( 'title', 'class', 'elementor-size-' . $settings['size'] );
    }

    $title = $settings['title'];

    if ( ! empty( $settings['link']['url'] ) ) {
        $this->add_link_attributes( 'url', $settings['link'] );
        // ✅ wp_kses_post() on $title — TEXTAREA control returns raw user input.
        // get_settings_for_display() does not auto-escape TEXTAREA values.
        $title = sprintf( '<a %1$s>%2$s</a>', $this->get_render_attribute_string( 'url' ), wp_kses_post( $title ) );
    } else {
        $title = wp_kses_post( $title );
    }

    $tag = \Elementor\Utils::validate_html_tag( $settings['header_size'] );
    // phpcs:ignore WordPress.Security.EscapeOutput.OutputNotEscaped -- tag validated, title kses'd, attrs via add_render_attribute
    printf( '<%1$s %2$s>%3$s</%1$s>', $tag, $this->get_render_attribute_string( 'title' ), $title );
}

protected function content_template(): void {
    ?>
    <#
    var title = settings.title;
    if ( settings.link && settings.link.url ) {
        // ✅ Official Elementor pattern — use link.url directly (per developers.elementor.com/docs/widgets/advanced-example/)
        // Content templates run in the trusted editor context; Elementor's own Link control
        // validates and sanitises URLs server-side before they reach the editor preview.
        title = '<a href="' + settings.link.url + '">' + title + '</a>';
    }
    var tag  = elementor.helpers.validateHTMLTag( settings.header_size );
    var size = settings.size && 'default' !== settings.size ? ' elementor-size-' + settings.size : '';
    #>
    <{{{ tag }}} class="myplugin-heading{{{ size }}}">{{{ title }}}</{{{ tag }}}>
    <?php
}
```

> **Remove checklist:**
> - `size` → remove if heading size is controlled purely via typography
> - `link` → remove if heading is never linked
> - `blend_mode` → remove for simple designs
> - `text_shadow` → remove for flat designs
