# Widget Boilerplate — Image Gallery

> **When to use this file:** Load whenever building a widget that displays a grid of images.
> Verified against `elementor/includes/widgets/image-gallery.php` (Elementor 3.35+).

---

```php
protected function register_controls(): void {

    // ── CONTENT ───────────────────────────────────────────────
    $this->start_controls_section( 'section_gallery', [
        'label' => esc_html__( 'Image Gallery', 'myplugin' ),
        'tab'   => \Elementor\Controls_Manager::TAB_CONTENT,
    ] );

    // ✅ GALLERY control — stores array of [ 'id' => int, 'url' => string ]
    $this->add_control( 'wp_gallery', [
        'label'   => esc_html__( 'Add Images', 'myplugin' ),
        'type'    => \Elementor\Controls_Manager::GALLERY,
        'default' => [],
        'dynamic' => [ 'active' => true ],
    ] );

    $this->add_group_control(
        \Elementor\Group_Control_Image_Size::get_type(),
        [
            'name'    => 'thumbnail',
            'default' => 'medium',
        ]
    );

    $this->add_responsive_control( 'gallery_columns', [
        'label'   => esc_html__( 'Columns', 'myplugin' ),
        'type'    => \Elementor\Controls_Manager::SELECT,
        'default' => '3',
        'options' => [ '1' => '1', '2' => '2', '3' => '3', '4' => '4', '5' => '5', '6' => '6' ],
    ] );

    $this->add_control( 'gallery_link', [
        'label'   => esc_html__( 'Link', 'myplugin' ),
        'type'    => \Elementor\Controls_Manager::SELECT,
        'default' => 'file',
        'options' => [
            'file'  => esc_html__( 'Media File', 'myplugin' ),
            'none'  => esc_html__( 'None',        'myplugin' ),
        ],
    ] );

    $this->add_control( 'open_lightbox', [
        'label'     => esc_html__( 'Lightbox', 'myplugin' ),
        'type'      => \Elementor\Controls_Manager::SELECT,
        'default'   => 'default',
        'options'   => [
            'default' => esc_html__( 'Default', 'myplugin' ),
            'yes'     => esc_html__( 'Yes',     'myplugin' ),
            'no'      => esc_html__( 'No',      'myplugin' ),
        ],
        'condition' => [ 'gallery_link' => 'file' ],
    ] );

    $this->add_control( 'gallery_rand', [
        'label'   => esc_html__( 'Ordering', 'myplugin' ),
        'type'    => \Elementor\Controls_Manager::SELECT,
        'default' => '',
        'options' => [
            ''     => esc_html__( 'Default',  'myplugin' ),
            'rand' => esc_html__( 'Random',   'myplugin' ),
        ],
    ] );

    $this->end_controls_section();

    // ── STYLE ─────────────────────────────────────────────────
    $this->start_controls_section( 'section_gallery_images_style', [
        'label' => esc_html__( 'Images', 'myplugin' ),
        'tab'   => \Elementor\Controls_Manager::TAB_STYLE,
    ] );

    $this->add_responsive_control( 'columns_gap', [
        'label'     => esc_html__( 'Columns Gap', 'myplugin' ),
        'type'      => \Elementor\Controls_Manager::SLIDER,
        'range'     => [ 'px' => [ 'min' => 0, 'max' => 100 ] ],
        'default'   => [ 'size' => 10, 'unit' => 'px' ],
        'selectors' => [ '{{WRAPPER}} .myplugin-gallery-item' => 'padding-right: calc( {{SIZE}}{{UNIT}}/2 ); padding-left: calc( {{SIZE}}{{UNIT}}/2 );' ],
    ] );

    $this->add_responsive_control( 'rows_gap', [
        'label'     => esc_html__( 'Rows Gap', 'myplugin' ),
        'type'      => \Elementor\Controls_Manager::SLIDER,
        'range'     => [ 'px' => [ 'min' => 0, 'max' => 100 ] ],
        'default'   => [ 'size' => 10, 'unit' => 'px' ],
        'selectors' => [ '{{WRAPPER}} .myplugin-gallery-item' => 'padding-bottom: {{SIZE}}{{UNIT}};' ],
    ] );

    $this->add_responsive_control( 'image_border_radius', [
        'label'      => esc_html__( 'Border Radius', 'myplugin' ),
        'type'       => \Elementor\Controls_Manager::DIMENSIONS,
        'size_units' => [ 'px', '%', 'em', 'rem' ],
        'selectors'  => [
            '{{WRAPPER}} .myplugin-gallery-item img' => 'border-radius: {{TOP}}{{UNIT}} {{RIGHT}}{{UNIT}} {{BOTTOM}}{{UNIT}} {{LEFT}}{{UNIT}};',
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

// ✅ Return true when 'Random' ordering is selected — shuffle() produces different output
// every render, so output caching must be bypassed to avoid freezing one random order forever.
// Return false for fixed ordering (Default) to benefit from output caching.
// Source: developers.elementor.com/docs/widgets/widget-output-caching/
protected function is_dynamic_content(): bool {
    return 'rand' === $this->get_settings( 'gallery_rand' );
}

protected function render(): void {
    $settings = $this->get_settings_for_display();

    if ( empty( $settings['wp_gallery'] ) ) {
        return;
    }

    $gallery = $settings['wp_gallery'];
    if ( 'rand' === $settings['gallery_rand'] ) {
        shuffle( $gallery );
    }
    ?>
    <div class="myplugin-gallery myplugin-gallery-columns-<?php echo esc_attr( $settings['gallery_columns'] ); ?>">
        <?php foreach ( $gallery as $attachment ) :
            $image_html = \Elementor\Group_Control_Image_Size::get_attachment_image_html( $settings, 'thumbnail', $attachment );
            if ( empty( $image_html ) ) continue;

            $link_tag_open  = '';
            $link_tag_close = '';
            if ( 'file' === $settings['gallery_link'] ) {
                $link_tag_open  = '<a href="' . esc_url( wp_get_attachment_url( $attachment['id'] ) ) . '" data-elementor-open-lightbox="' . esc_attr( $settings['open_lightbox'] ) . '">';
                $link_tag_close = '</a>';
            }
            ?>
            <div class="myplugin-gallery-item">
                <?php echo $link_tag_open; // phpcs:ignore ?>
                    <?php echo wp_kses_post( $image_html ); ?>
                <?php echo $link_tag_close; // phpcs:ignore ?>
            </div>
        <?php endforeach; ?>
    </div>
    <?php
}
```



**content_template() skeleton:**

```php
protected function content_template(): void {
    ?>
    <# if ( ! settings.wp_gallery || ! settings.wp_gallery.length ) { return; } #>
    <div class="myplugin-gallery myplugin-gallery-columns-{{ settings.gallery_columns }}">
        <# _.each( settings.wp_gallery, function( attachment ) { #>
        <div class="myplugin-gallery-item">
            <img src="{{ attachment.url }}" alt="">
        </div>
        <# } ); #>
    </div>
    <?php
}
```
