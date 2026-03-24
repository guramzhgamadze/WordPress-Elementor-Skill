# Widget Boilerplate — Image

> **When to use this file:** Load this boilerplate whenever building a widget that includes
> a primary image element — hero images, thumbnails, team photos, logos, or any single focal
> image with optional caption and link.
>
> Verified against `elementor/includes/widgets/image.php` (Elementor 3.35+).
> Uses `Group_Control_Image_Size` + `get_attachment_image_html()` — the correct Elementor-native
> approach. Never use a hardcoded SELECT or `wp_get_attachment_image()` directly.

---

### Image Widget Boilerplate

Verified against `elementor/includes/widgets/image.php` (Elementor 3.35+).

**Critical difference from naive implementations:**
Elementor uses `Group_Control_Image_Size` for image size — NOT a SELECT with hardcoded sizes.
This group control handles all registered WordPress sizes plus custom dimensions automatically,
and must be paired with `Group_Control_Image_Size::get_attachment_image_html()` in render().
Using `wp_get_attachment_image()` directly bypasses the custom dimension control entirely.

```php
protected function register_controls(): void {

    // =========================================================
    // TAB: CONTENT
    // =========================================================

    $this->start_controls_section( 'section_image', [
        'label' => esc_html__( 'Image', 'myplugin' ),
        'tab'   => \Elementor\Controls_Manager::TAB_CONTENT,
    ] );

    $this->add_control( 'image', [
        'label'   => esc_html__( 'Choose Image', 'myplugin' ),
        'type'    => \Elementor\Controls_Manager::MEDIA,
        'default' => [
            // ✅ Shows Elementor's grey placeholder — keeps widget visible with no image set
            'url' => \Elementor\Utils::get_placeholder_image_src(),
        ],
        'dynamic' => [ 'active' => true ],
    ] );

    // ✅ Group_Control_Image_Size generates two controls automatically:
    //    '{name}_size'             → SELECT of all registered WP image sizes + 'custom'
    //    '{name}_custom_dimension' → IMAGE_DIMENSIONS inputs, shown only when size = 'custom'
    // The 'name' value here determines the control ID prefix — must match get_attachment_image_html() call.
    $this->add_group_control(
        \Elementor\Group_Control_Image_Size::get_type(),
        [
            'name'      => 'image',    // generates 'image_size' and 'image_custom_dimension'
            'default'   => 'large',
            'separator' => 'none',
        ]
    );

    $this->add_control( 'caption_source', [
        'label'   => esc_html__( 'Caption', 'myplugin' ),
        'type'    => \Elementor\Controls_Manager::SELECT,
        'default' => 'none',
        'options' => [
            'none'       => esc_html__( 'None',               'myplugin' ),
            'attachment' => esc_html__( 'Attachment Caption', 'myplugin' ),
            'custom'     => esc_html__( 'Custom Caption',     'myplugin' ),
        ],
    ] );

    $this->add_control( 'caption', [
        'label'       => esc_html__( 'Custom Caption', 'myplugin' ),
        'type'        => \Elementor\Controls_Manager::TEXT,
        'default'     => '',
        'placeholder' => esc_html__( 'Enter your image caption', 'myplugin' ),
        'condition'   => [ 'caption_source' => 'custom' ],
        'dynamic'     => [ 'active' => true ],
    ] );

    $this->add_control( 'link_to', [
        'label'   => esc_html__( 'Link', 'myplugin' ),
        'type'    => \Elementor\Controls_Manager::SELECT,
        'default' => 'none',
        'options' => [
            'none'   => esc_html__( 'None',       'myplugin' ),
            'file'   => esc_html__( 'Media File', 'myplugin' ),
            'custom' => esc_html__( 'Custom URL', 'myplugin' ),
        ],
    ] );

    $this->add_control( 'link', [
        'label'       => esc_html__( 'Link URL', 'myplugin' ),
        'type'        => \Elementor\Controls_Manager::URL,
        'placeholder' => esc_html__( 'https://your-link.com', 'myplugin' ),
        'condition'   => [ 'link_to' => 'custom' ],
        'dynamic'     => [ 'active' => true ],
        'label_block' => true,
    ] );

    // ✅ open_lightbox — control ID matches Elementor native exactly
    $this->add_control( 'open_lightbox', [
        'label'     => esc_html__( 'Lightbox', 'myplugin' ),
        'type'      => \Elementor\Controls_Manager::SELECT,
        'default'   => 'default',
        'options'   => [
            'default' => esc_html__( 'Default', 'myplugin' ),
            'yes'     => esc_html__( 'Yes',     'myplugin' ),
            'no'      => esc_html__( 'No',      'myplugin' ),
        ],
        'condition' => [ 'link_to' => 'file' ],
    ] );

    $this->end_controls_section();

    // =========================================================
    // TAB: STYLE
    // =========================================================

    $this->start_controls_section( 'section_style_image', [
        'label' => esc_html__( 'Image', 'myplugin' ),
        'tab'   => \Elementor\Controls_Manager::TAB_STYLE,
    ] );

    // ✅ No 'default' — Elementor native leaves this empty (inherits document flow)
    $this->add_responsive_control( 'align', [
        'label'     => esc_html__( 'Alignment', 'myplugin' ),
        'type'      => \Elementor\Controls_Manager::CHOOSE,
        'options'   => [
            'left'   => [ 'title' => esc_html__( 'Left',   'myplugin' ), 'icon' => 'eicon-text-align-left'   ],
            'center' => [ 'title' => esc_html__( 'Center', 'myplugin' ), 'icon' => 'eicon-text-align-center' ],
            'right'  => [ 'title' => esc_html__( 'Right',  'myplugin' ), 'icon' => 'eicon-text-align-right'  ],
        ],
        'selectors' => [
            '{{WRAPPER}}' => 'text-align: {{VALUE}};',
        ],
    ] );

    // ✅ Width/Max Width/Height — size intentionally empty (no forced value), unit set only.
    // Empty size = browser/CSS default. Setting a size writes inline style and overrides cascade.
    $this->add_responsive_control( 'width', [
        'label'      => esc_html__( 'Width', 'myplugin' ),
        'type'       => \Elementor\Controls_Manager::SLIDER,
        'default'    => [ 'unit' => '%' ],
        'size_units' => [ 'px', '%', 'vw', 'em', 'rem', 'custom' ],
        'range'      => [
            'px' => [ 'min' => 1, 'max' => 1200 ],
            '%'  => [ 'min' => 1, 'max' => 100  ],
            'vw' => [ 'min' => 1, 'max' => 100  ],
        ],
        'selectors'  => [
            // ✅ Elementor native targets '{{WRAPPER}} img' directly — no wrapper class
            '{{WRAPPER}} img' => 'width: {{SIZE}}{{UNIT}};',
        ],
    ] );

    $this->add_responsive_control( 'max_width', [
        'label'      => esc_html__( 'Max Width', 'myplugin' ),
        'type'       => \Elementor\Controls_Manager::SLIDER,
        'default'    => [ 'unit' => '%' ],
        'size_units' => [ 'px', '%', 'vw', 'em', 'rem', 'custom' ],
        'range'      => [
            'px' => [ 'min' => 1, 'max' => 1200 ],
            '%'  => [ 'min' => 1, 'max' => 100  ],
            'vw' => [ 'min' => 1, 'max' => 100  ],
        ],
        'selectors'  => [
            '{{WRAPPER}} img' => 'max-width: {{SIZE}}{{UNIT}};',
        ],
    ] );

    $this->add_responsive_control( 'height', [
        'label'      => esc_html__( 'Height', 'myplugin' ),
        'type'       => \Elementor\Controls_Manager::SLIDER,
        'default'    => [ 'unit' => 'px' ],
        'size_units' => [ 'px', 'vh', 'em', 'rem', 'custom' ],
        'range'      => [
            'px' => [ 'min' => 1, 'max' => 1200 ],
            'vh' => [ 'min' => 1, 'max' => 100  ],
        ],
        'selectors'  => [
            '{{WRAPPER}} img' => 'height: {{SIZE}}{{UNIT}}; object-fit: cover;',
        ],
    ] );

    // ── Normal / Hover tabs ───────────────────────────────────
    $this->start_controls_tabs( 'tabs_image_effects' );

        $this->start_controls_tab( 'tab_image_normal', [
            'label' => esc_html__( 'Normal', 'myplugin' ),
        ] );

            // ✅ No default — writing opacity:1 inline blocks CSS hover transitions
            $this->add_control( 'opacity', [
                'label'     => esc_html__( 'Opacity', 'myplugin' ),
                'type'      => \Elementor\Controls_Manager::SLIDER,
                'range'     => [ 'px' => [ 'min' => 0.00, 'max' => 1, 'step' => 0.01 ] ],
                'selectors' => [
                    '{{WRAPPER}} img' => 'opacity: {{SIZE}};',
                ],
            ] );

            $this->add_group_control(
                \Elementor\Group_Control_Css_Filter::get_type(),
                [
                    'name'     => 'css_filters',
                    'selector' => '{{WRAPPER}} img',
                ]
            );

        $this->end_controls_tab();

        $this->start_controls_tab( 'tab_image_hover', [
            'label' => esc_html__( 'Hover', 'myplugin' ),
        ] );

            $this->add_control( 'opacity_hover', [
                'label'     => esc_html__( 'Opacity', 'myplugin' ),
                'type'      => \Elementor\Controls_Manager::SLIDER,
                'range'     => [ 'px' => [ 'min' => 0.00, 'max' => 1, 'step' => 0.01 ] ],
                'selectors' => [
                    '{{WRAPPER}}:hover img' => 'opacity: {{SIZE}};',
                ],
            ] );

            $this->add_group_control(
                \Elementor\Group_Control_Css_Filter::get_type(),
                [
                    'name'     => 'css_filters_hover',
                    'selector' => '{{WRAPPER}}:hover img',
                ]
            );

            $this->add_control( 'hover_transition', [
                'label'     => esc_html__( 'Transition Duration', 'myplugin' ),
                'type'      => \Elementor\Controls_Manager::SLIDER,
                'range'     => [ 'px' => [ 'min' => 0, 'max' => 3, 'step' => 0.1 ] ],
                'selectors' => [
                    '{{WRAPPER}} img' => 'transition-duration: {{SIZE}}s;',
                ],
            ] );

            // ⛔ DO NOT include hover_animation by default — loads animation CSS library.
            // Only uncomment if the client explicitly requires image hover animation.
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
            'name'      => 'image_border',
            // ✅ Elementor native targets '{{WRAPPER}} img' directly
            'selector'  => '{{WRAPPER}} img',
            'separator' => 'before',
        ]
    );

    $this->add_responsive_control( 'image_border_radius', [
        'label'      => esc_html__( 'Border Radius', 'myplugin' ),
        'type'       => \Elementor\Controls_Manager::DIMENSIONS,
        'size_units' => [ 'px', '%', 'em', 'rem' ],
        'selectors'  => [
            '{{WRAPPER}} img' => 'border-radius: {{TOP}}{{UNIT}} {{RIGHT}}{{UNIT}} {{BOTTOM}}{{UNIT}} {{LEFT}}{{UNIT}};',
        ],
    ] );

    $this->add_group_control(
        \Elementor\Group_Control_Box_Shadow::get_type(),
        [
            'name'     => 'image_box_shadow',
            'selector' => '{{WRAPPER}} img',
        ]
    );

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

    if ( empty( $settings['image']['url'] ) ) {
        return;
    }

    $has_caption = $this->has_caption( $settings );
    $link        = $this->get_link_url( $settings );

    $this->add_render_attribute( 'wrapper', 'class', 'elementor-image' );

    if ( $link ) {
        $this->add_link_attributes( 'link', $link );
        $this->add_render_attribute( 'link', [
            'class'                        => 'elementor-clickable',
            'data-elementor-open-lightbox' => $settings['open_lightbox'],
        ] );
    }
    ?>
    <div <?php $this->print_render_attribute_string( 'wrapper' ); ?>>
        <?php if ( $has_caption ) : ?><figure class="wp-caption"><?php endif; ?>
        <?php if ( $link ) : ?><a <?php $this->print_render_attribute_string( 'link' ); ?>><?php endif; ?>

        <?php
        // ✅ ALWAYS use get_attachment_image_html() — not wp_get_attachment_image().
        // This is the only method that correctly reads both 'image_size' and
        // 'image_custom_dimension' controls generated by Group_Control_Image_Size.
        echo \Elementor\Group_Control_Image_Size::get_attachment_image_html( $settings );
        ?>

        <?php if ( $link ) : ?></a><?php endif; ?>
        <?php if ( $has_caption ) : ?>
            <figcaption class="widget-image-caption wp-caption-text">
                <?php echo esc_html( $this->get_caption( $settings ) ); ?>
            </figcaption>
        </figure>
        <?php endif; ?>
    </div>
    <?php
}

private function has_caption( array $settings ): bool {
    return ( ! empty( $settings['caption_source'] ) && 'none' !== $settings['caption_source'] );
}

private function get_caption( array $settings ): string {
    if ( 'custom' === $settings['caption_source'] ) {
        return $settings['caption'] ?? '';
    }
    if ( 'attachment' === $settings['caption_source'] && ! empty( $settings['image']['id'] ) ) {
        return (string) wp_get_attachment_caption( (int) $settings['image']['id'] );
    }
    return '';
}

private function get_link_url( array $settings ): ?array {
    if ( 'none' === $settings['link_to'] ) {
        return null;
    }
    if ( 'custom' === $settings['link_to'] ) {
        return ! empty( $settings['link']['url'] ) ? $settings['link'] : null;
    }
    return [ 'url' => $settings['image']['url'] ]; // 'file'
}

protected function content_template(): void {
    ?>
    <#
    if ( ! settings.image.url ) { return; }

    // ✅ Official Elementor docs use 'const' here — matches developers.elementor.com/docs/widgets/rendering-media/
    const image = {
        id:        settings.image.id,
        url:       settings.image.url,
        size:      settings.image_size,
        dimension: settings.image_custom_dimension,
        model:     view.model, // ✅ view.model replaces deprecated view.getEditModel() (removed Elementor 3.x)
    };
    const image_url = elementor.imagesManager.getImageUrl( image );
    // ✅ Official docs use strict empty string check
    if ( '' === image_url ) { return; }

    const hasCaption = 'none' !== settings.caption_source;
    let link_url;
    if ( 'custom' === settings.link_to ) {
        // ✅ Official Elementor pattern — link.url used directly per official advanced example
        link_url = settings.link.url || '';
    }
    if ( 'file' === settings.link_to ) { link_url = settings.image.url; }

    // ⛔ hover_animation excluded — uncomment if control is re-enabled:
    // var imgClass = settings.hover_animation ? 'elementor-animation-' + settings.hover_animation : '';
    #>
    <div class="elementor-image">
        <# if ( hasCaption ) { #><figure class="wp-caption"><# } #>
        <# if ( link_url ) { #>
            <a class="elementor-clickable" data-elementor-open-lightbox="{{ settings.open_lightbox }}" href="{{ link_url }}">
        <# } #>
        <img src="{{ image_url }}" alt="">
        <# if ( link_url ) { #></a><# } #>
        <# if ( hasCaption ) { #>
            <figcaption class="widget-image-caption wp-caption-text">
                {{{ 'custom' === settings.caption_source ? settings.caption : '' }}}
            </figcaption>
        </figure>
        <# } #>
    </div>
    <?php
}
```

> **Remove checklist:**
> - `caption_source` + `caption` → remove if image never has a caption
> - `link_to` + `link` + `open_lightbox` → remove if image is purely decorative
> - `max_width` → remove if image always fills its container
> - `height` → remove if image must maintain natural aspect ratio
> - `image_border_radius` → remove for always-square/unrounded images
> - `hover_transition` → remove if no CSS hover effect applied to the image
> - `hover_animation` → **excluded by default** (commented out)
> - Opacity controls (both tabs) → remove for always-fully-opaque images
