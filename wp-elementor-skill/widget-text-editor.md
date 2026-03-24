# Widget Boilerplate — Text Editor

> **When to use this file:** Load whenever building a widget with a rich text / WYSIWYG body content area.
> Verified against `elementor/includes/widgets/text-editor.php` (Elementor 3.35+).

---

```php
protected function register_controls(): void {

    // ── CONTENT ───────────────────────────────────────────────
    $this->start_controls_section( 'section_editor', [
        'label' => esc_html__( 'Text Editor', 'myplugin' ),
        'tab'   => \Elementor\Controls_Manager::TAB_CONTENT,
    ] );

    $this->add_control( 'editor', [
        'label'   => '',
        'type'    => \Elementor\Controls_Manager::WYSIWYG,
        'default' => esc_html__( 'Click edit button to change this text. Lorem ipsum dolor sit amet, consectetur adipiscing elit. Ut elit tellus, luctus nec ullamcorper mattis, pulvinar dapibus leo.', 'myplugin' ),
        'dynamic' => [ 'active' => true ],
    ] );

    $this->end_controls_section();

    // ── STYLE ─────────────────────────────────────────────────
    $this->start_controls_section( 'section_style', [
        'label' => esc_html__( 'Text Editor', 'myplugin' ),
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
        'selectors' => [ '{{WRAPPER}} .myplugin-text-editor' => 'text-align: {{VALUE}};' ],
    ] );

    $this->add_control( 'text_color', [
        'label'     => esc_html__( 'Text Color', 'myplugin' ),
        'type'      => \Elementor\Controls_Manager::COLOR,
        'default'   => '',
        'selectors' => [
            '{{WRAPPER}} .myplugin-text-editor'    => 'color: {{VALUE}};',
            '{{WRAPPER}} .myplugin-text-editor p'  => 'color: {{VALUE}};',
        ],
    ] );

    $this->add_group_control(
        \Elementor\Group_Control_Typography::get_type(),
        [
            'name'     => 'typography',
            'global'   => [ 'default' => \Elementor\Core\Kits\Documents\Tabs\Global_Typography::TYPOGRAPHY_TEXT ],
            'selector' => '{{WRAPPER}} .myplugin-text-editor',
        ]
    );

    $this->add_group_control(
        \Elementor\Group_Control_Text_Shadow::get_type(),
        [
            'name'     => 'text_shadow',
            'selector' => '{{WRAPPER}} .myplugin-text-editor',
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
    $this->add_render_attribute( 'editor', 'class', [ 'myplugin-text-editor', 'elementor-clearfix' ] );
    // ✅ wp_kses_post() — WYSIWYG output must allow HTML tags but still be sanitized.
    // Official Elementor docs omit wp_kses_post() in examples, but our skill's php-standards.md
    // mandates escaping all output. WYSIWYG content is trusted editor HTML, so wp_kses_post is correct.
    echo '<div ' . $this->get_render_attribute_string( 'editor' ) . '>' . wp_kses_post( $settings['editor'] ) . '</div>';
}

protected function content_template(): void {
    ?>
    <div class="myplugin-text-editor elementor-clearfix">{{{ settings.editor }}}</div>
    <?php
}
```

> **Remove checklist:**
> - `text_shadow` → remove for flat designs
