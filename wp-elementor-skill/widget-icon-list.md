# Widget Boilerplate — Icon List

> **When to use this file:** Load whenever building a bullet-list widget where each item has an icon, text, and optional link.
> Verified against `elementor/includes/widgets/icon-list.php` (Elementor 3.35+).

---

```php
protected function register_controls(): void {

    // ── CONTENT ───────────────────────────────────────────────
    $this->start_controls_section( 'section_icon_list', [
        'label' => esc_html__( 'Icon List', 'myplugin' ),
        'tab'   => \Elementor\Controls_Manager::TAB_CONTENT,
    ] );

    $this->add_control( 'view', [
        'label'        => esc_html__( 'Layout', 'myplugin' ),
        'type'         => \Elementor\Controls_Manager::CHOOSE,
        'default'      => 'traditional',
        'options'      => [
            'traditional' => [ 'title' => esc_html__( 'Default', 'myplugin' ), 'icon' => 'eicon-editor-list-ul' ],
            'inline'      => [ 'title' => esc_html__( 'Inline',  'myplugin' ), 'icon' => 'eicon-ellipsis-h'     ],
        ],
        'prefix_class' => 'elementor-icon-list--layout-',
    ] );

    $repeater = new \Elementor\Repeater();

    $repeater->add_control( 'text', [
        'label'       => esc_html__( 'Text', 'myplugin' ),
        'type'        => \Elementor\Controls_Manager::TEXT,
        'label_block' => true,
        'default'     => esc_html__( 'Text', 'myplugin' ),
        'placeholder' => esc_html__( 'List Item', 'myplugin' ),
        'dynamic'     => [ 'active' => true ],
    ] );

    $repeater->add_control( 'selected_icon', [
        'label'            => esc_html__( 'Icon', 'myplugin' ),
        'type'             => \Elementor\Controls_Manager::ICONS,
        'default'          => [ 'value' => 'fas fa-check', 'library' => 'fa-solid' ],
        'fa4compatibility' => 'icon',
    ] );

    $repeater->add_control( 'link', [
        'label'   => esc_html__( 'Link', 'myplugin' ),
        'type'    => \Elementor\Controls_Manager::URL,
        'dynamic' => [ 'active' => true ],
    ] );

    $this->add_control( 'icon_list', [
        'label'       => '',
        'type'        => \Elementor\Controls_Manager::REPEATER,
        'fields'      => $repeater->get_controls(),
        'default'     => [
            [ 'text' => esc_html__( 'List Item #1', 'myplugin' ), 'selected_icon' => [ 'value' => 'fas fa-check', 'library' => 'fa-solid' ] ],
            [ 'text' => esc_html__( 'List Item #2', 'myplugin' ), 'selected_icon' => [ 'value' => 'fas fa-check', 'library' => 'fa-solid' ] ],
            [ 'text' => esc_html__( 'List Item #3', 'myplugin' ), 'selected_icon' => [ 'value' => 'fas fa-check', 'library' => 'fa-solid' ] ],
        ],
        'title_field' => '{{{ elementor.helpers.renderIcon( this, selected_icon, {}, "i", "panel" ) || \'<i class="\' + icon + \'"></i>\' }}} {{{ text }}}',
    ] );

    $this->end_controls_section();

    // ── STYLE: List ───────────────────────────────────────────
    $this->start_controls_section( 'section_icon_list_style', [
        'label' => esc_html__( 'List', 'myplugin' ),
        'tab'   => \Elementor\Controls_Manager::TAB_STYLE,
    ] );

    $this->add_responsive_control( 'space_between', [
        'label'     => esc_html__( 'Space Between', 'myplugin' ),
        'type'      => \Elementor\Controls_Manager::SLIDER,
        'range'     => [ 'px' => [ 'min' => 0, 'max' => 50 ] ],
        'default'   => [ 'size' => 5, 'unit' => 'px' ],
        'selectors' => [
            '{{WRAPPER}} .myplugin-icon-list-item:not(:last-child)' => 'padding-bottom: {{SIZE}}{{UNIT}};',
        ],
    ] );

    $this->add_control( 'divider', [
        'label'     => esc_html__( 'Divider', 'myplugin' ),
        'type'      => \Elementor\Controls_Manager::SWITCHER,
        'separator' => 'before',
    ] );

    $this->add_control( 'divider_style', [
        'label'     => esc_html__( 'Style', 'myplugin' ),
        'type'      => \Elementor\Controls_Manager::SELECT,
        'default'   => 'solid',
        'options'   => [
            'solid'  => esc_html__( 'Solid',  'myplugin' ),
            'double' => esc_html__( 'Double', 'myplugin' ),
            'dotted' => esc_html__( 'Dotted', 'myplugin' ),
            'dashed' => esc_html__( 'Dashed', 'myplugin' ),
        ],
        'condition' => [ 'divider' => 'yes' ],
        'selectors' => [
            '{{WRAPPER}} .myplugin-icon-list-item:not(:last-child)::after' => 'border-top-style: {{VALUE}};',
        ],
    ] );

    $this->end_controls_section();

    // ── STYLE: Icon ───────────────────────────────────────────
    $this->start_controls_section( 'section_icon_style', [
        'label' => esc_html__( 'Icon', 'myplugin' ),
        'tab'   => \Elementor\Controls_Manager::TAB_STYLE,
    ] );

    $this->add_responsive_control( 'icon_size', [
        'label'     => esc_html__( 'Size', 'myplugin' ),
        'type'      => \Elementor\Controls_Manager::SLIDER,
        'default'   => [ 'size' => 14, 'unit' => 'px' ],
        'range'     => [ 'px' => [ 'min' => 6, 'max' => 300 ] ],
        'selectors' => [
            '{{WRAPPER}} .myplugin-icon-list-icon i'   => 'font-size: {{SIZE}}{{UNIT}};',
            '{{WRAPPER}} .myplugin-icon-list-icon svg' => 'width: {{SIZE}}{{UNIT}}; height: {{SIZE}}{{UNIT}};',
        ],
    ] );

    $this->add_control( 'icon_color', [
        'label'     => esc_html__( 'Color', 'myplugin' ),
        'type'      => \Elementor\Controls_Manager::COLOR,
        'default'   => '',
        'selectors' => [
            '{{WRAPPER}} .myplugin-icon-list-icon i'   => 'color: {{VALUE}};',
            '{{WRAPPER}} .myplugin-icon-list-icon svg' => 'fill: {{VALUE}};',
        ],
    ] );

    $this->end_controls_section();

    // ── STYLE: Text ───────────────────────────────────────────
    $this->start_controls_section( 'section_text_style', [
        'label' => esc_html__( 'Text', 'myplugin' ),
        'tab'   => \Elementor\Controls_Manager::TAB_STYLE,
    ] );

    $this->add_control( 'text_color', [
        'label'     => esc_html__( 'Text Color', 'myplugin' ),
        'type'      => \Elementor\Controls_Manager::COLOR,
        'default'   => '',
        'selectors' => [ '{{WRAPPER}} .myplugin-icon-list-text' => 'color: {{VALUE}};' ],
    ] );

    $this->add_group_control(
        \Elementor\Group_Control_Typography::get_type(),
        [
            'name'     => 'icon_typography',
            'selector' => '{{WRAPPER}} .myplugin-icon-list-item',
            'global'   => [ 'default' => \Elementor\Core\Kits\Documents\Tabs\Global_Typography::TYPOGRAPHY_TEXT ],
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
    ?>
    <ul class="myplugin-icon-list-items">
        <?php foreach ( $settings['icon_list'] as $index => $item ) :
            $repeater_key = $this->get_repeater_setting_key( 'text', 'icon_list', $index );
            $this->add_render_attribute( $repeater_key, 'class', 'myplugin-icon-list-text' );
            // ✅ add_inline_editing_attributes() is deprecated since Elementor 3.x — do not use.

            $tag = ! empty( $item['link']['url'] ) ? 'a' : 'span';
            if ( 'a' === $tag ) {
                $link_key = $this->get_repeater_setting_key( 'link', 'icon_list', $index );
                $this->add_link_attributes( $link_key, $item['link'] );
            }
            ?>
            <li class="myplugin-icon-list-item elementor-repeater-item-<?php echo esc_attr( $item['_id'] ); ?>">
                <<?php echo esc_attr( $tag ); ?> <?php echo 'a' === $tag ? $this->get_render_attribute_string( $link_key ) : ''; // phpcs:ignore WordPress.Security.EscapeOutput.OutputNotEscaped -- get_render_attribute_string() returns pre-sanitized attribute markup ?>>
                    <?php if ( ! empty( $item['selected_icon']['value'] ) ) : ?>
                        <span class="myplugin-icon-list-icon">
                            <?php \Elementor\Icons_Manager::render_icon( $item['selected_icon'], [ 'aria-hidden' => 'true' ] ); ?>
                        </span>
                    <?php endif; ?>
                    <span <?php $this->print_render_attribute_string( $repeater_key ); ?>><?php echo esc_html( $item['text'] ); ?></span>
                </<?php echo esc_attr( $tag ); ?>>
            </li>
        <?php endforeach; ?>
    </ul>
    <?php
}
```

> **Remove checklist:**
> - `divider` + `divider_style` → remove if list items never need separators
> - `link` in repeater → remove if list items are never linked


**content_template() skeleton:**

```php
protected function content_template(): void {
    ?>
    <# if ( ! settings.icon_list || ! settings.icon_list.length ) { return; } #>
    <ul class="myplugin-icon-list-items">
        <# _.each( settings.icon_list, function( item ) {
            var iconHTML = elementor.helpers.renderIcon( view, item.selected_icon, { 'aria-hidden': true }, 'i', 'object' );
            var tag = item.link && item.link.url ? 'a' : 'span';
            // ✅ Official Elementor pattern — link.url used directly per official advanced example
            var linkAttr = 'a' === tag ? 'href="' + item.link.url + '"' : '';
        #>
        <li class="myplugin-icon-list-item elementor-repeater-item-{{ item._id }}">
            <{{{ tag }}} {{{ linkAttr }}}>
                <# if ( iconHTML && iconHTML.value ) { #>
                    <span class="myplugin-icon-list-icon">{{{ iconHTML.value }}}</span>
                <# } #>
                <span class="myplugin-icon-list-text">{{{ item.text }}}</span>
            </{{{ tag }}}>
        </li>
        <# } ); #>
    </ul>
    <?php
}
```
