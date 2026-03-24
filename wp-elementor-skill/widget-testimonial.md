# Widget Boilerplate — Testimonial

> **When to use this file:** Load whenever building a widget displaying a customer quote with name, job title, and image.
> Verified against `elementor/includes/widgets/testimonial.php` (Elementor 3.35+).

---

```php
protected function register_controls(): void {

    // ── CONTENT ───────────────────────────────────────────────
    $this->start_controls_section( 'section_testimonial', [
        'label' => esc_html__( 'Testimonial', 'myplugin' ),
        'tab'   => \Elementor\Controls_Manager::TAB_CONTENT,
    ] );

    $this->add_control( 'testimonial_content', [
        'label'   => esc_html__( 'Content', 'myplugin' ),
        'type'    => \Elementor\Controls_Manager::TEXTAREA,
        'rows'    => 10,
        'default' => esc_html__( 'Click edit button to change this text. Lorem ipsum dolor sit amet consectetur adipiscing elit dolor', 'myplugin' ),
        'dynamic' => [ 'active' => true ],
    ] );

    $this->add_control( 'testimonial_image', [
        'label'   => esc_html__( 'Choose Image', 'myplugin' ),
        'type'    => \Elementor\Controls_Manager::MEDIA,
        'default' => [ 'url' => \Elementor\Utils::get_placeholder_image_src() ],
        'dynamic' => [ 'active' => true ],
    ] );

    // ✅ REQUIRED — render() calls get_attachment_image_html( $settings, 'thumbnail', 'testimonial_image' )
    // which reads the 'testimonial_image_size' control generated here.
    $this->add_group_control(
        \Elementor\Group_Control_Image_Size::get_type(),
        [
            'name'      => 'testimonial_image', // generates 'testimonial_image_size' + 'testimonial_image_custom_dimension'
            'default'   => 'thumbnail',
            'separator' => 'none',
        ]
    );

    $this->add_control( 'testimonial_name', [
        'label'   => esc_html__( 'Name', 'myplugin' ),
        'type'    => \Elementor\Controls_Manager::TEXT,
        'default' => esc_html__( 'John Doe', 'myplugin' ),
        'dynamic' => [ 'active' => true ],
    ] );

    $this->add_control( 'testimonial_job', [
        'label'   => esc_html__( 'Title', 'myplugin' ),
        'type'    => \Elementor\Controls_Manager::TEXT,
        'default' => esc_html__( 'Designer', 'myplugin' ),
        'dynamic' => [ 'active' => true ],
    ] );

    $this->add_control( 'link', [
        'label'     => esc_html__( 'Link', 'myplugin' ),
        'type'      => \Elementor\Controls_Manager::URL,
        'dynamic'   => [ 'active' => true ],
        'separator' => 'before',
    ] );

    $this->end_controls_section();

    // ── STYLE ─────────────────────────────────────────────────
    $this->start_controls_section( 'section_style_testimonial_content', [
        'label' => esc_html__( 'Content', 'myplugin' ),
        'tab'   => \Elementor\Controls_Manager::TAB_STYLE,
    ] );

    $this->add_responsive_control( 'testimonial_alignment', [
        'label'     => esc_html__( 'Alignment', 'myplugin' ),
        'type'      => \Elementor\Controls_Manager::CHOOSE,
        'options'   => [
            'left'   => [ 'title' => esc_html__( 'Left',   'myplugin' ), 'icon' => 'eicon-text-align-left'   ],
            'center' => [ 'title' => esc_html__( 'Center', 'myplugin' ), 'icon' => 'eicon-text-align-center' ],
            'right'  => [ 'title' => esc_html__( 'Right',  'myplugin' ), 'icon' => 'eicon-text-align-right'  ],
        ],
        'selectors' => [ '{{WRAPPER}} .myplugin-testimonial' => 'text-align: {{VALUE}};' ],
    ] );

    $this->add_control( 'content_color', [
        'label'     => esc_html__( 'Text Color', 'myplugin' ),
        'type'      => \Elementor\Controls_Manager::COLOR,
        'global'    => [ 'default' => \Elementor\Core\Kits\Documents\Tabs\Global_Colors::COLOR_TEXT ],
        'selectors' => [ '{{WRAPPER}} .myplugin-testimonial-content' => 'color: {{VALUE}};' ],
    ] );

    $this->add_group_control(
        \Elementor\Group_Control_Typography::get_type(),
        [
            'name'     => 'content_typography',
            'global'   => [ 'default' => \Elementor\Core\Kits\Documents\Tabs\Global_Typography::TYPOGRAPHY_TEXT ],
            'selector' => '{{WRAPPER}} .myplugin-testimonial-content',
        ]
    );

    $this->end_controls_section();

    $this->start_controls_section( 'section_style_testimonial_image', [
        'label' => esc_html__( 'Image', 'myplugin' ),
        'tab'   => \Elementor\Controls_Manager::TAB_STYLE,
    ] );

    $this->add_responsive_control( 'image_size', [
        'label'     => esc_html__( 'Size', 'myplugin' ),
        'type'      => \Elementor\Controls_Manager::SLIDER,
        'range'     => [ 'px' => [ 'min' => 20, 'max' => 200 ] ],
        'default'   => [ 'size' => 60, 'unit' => 'px' ],
        'selectors' => [
            '{{WRAPPER}} .myplugin-testimonial-image img' => 'width: {{SIZE}}{{UNIT}}; height: {{SIZE}}{{UNIT}};',
        ],
    ] );

    $this->add_responsive_control( 'image_border_radius', [
        'label'      => esc_html__( 'Border Radius', 'myplugin' ),
        'type'       => \Elementor\Controls_Manager::DIMENSIONS,
        'size_units' => [ 'px', '%' ],
        'default'    => [ 'unit' => '%', 'top' => 50, 'right' => 50, 'bottom' => 50, 'left' => 50, 'isLinked' => true ],
        'selectors'  => [
            '{{WRAPPER}} .myplugin-testimonial-image img' => 'border-radius: {{TOP}}{{UNIT}} {{RIGHT}}{{UNIT}} {{BOTTOM}}{{UNIT}} {{LEFT}}{{UNIT}};',
        ],
    ] );

    $this->end_controls_section();

    $this->start_controls_section( 'section_style_testimonial_name', [
        'label' => esc_html__( 'Name', 'myplugin' ),
        'tab'   => \Elementor\Controls_Manager::TAB_STYLE,
    ] );

    $this->add_control( 'name_color', [
        'label'     => esc_html__( 'Text Color', 'myplugin' ),
        'type'      => \Elementor\Controls_Manager::COLOR,
        'global'    => [ 'default' => \Elementor\Core\Kits\Documents\Tabs\Global_Colors::COLOR_PRIMARY ],
        'selectors' => [ '{{WRAPPER}} .myplugin-testimonial-name' => 'color: {{VALUE}};' ],
    ] );

    $this->add_group_control(
        \Elementor\Group_Control_Typography::get_type(),
        [
            'name'     => 'name_typography',
            'global'   => [ 'default' => \Elementor\Core\Kits\Documents\Tabs\Global_Typography::TYPOGRAPHY_PRIMARY ],
            'selector' => '{{WRAPPER}} .myplugin-testimonial-name',
        ]
    );

    $this->end_controls_section();

    $this->start_controls_section( 'section_style_testimonial_job', [
        'label' => esc_html__( 'Title', 'myplugin' ),
        'tab'   => \Elementor\Controls_Manager::TAB_STYLE,
    ] );

    $this->add_control( 'job_color', [
        'label'     => esc_html__( 'Text Color', 'myplugin' ),
        'type'      => \Elementor\Controls_Manager::COLOR,
        'global'    => [ 'default' => \Elementor\Core\Kits\Documents\Tabs\Global_Colors::COLOR_SECONDARY ],
        'selectors' => [ '{{WRAPPER}} .myplugin-testimonial-job' => 'color: {{VALUE}};' ],
    ] );

    $this->add_group_control(
        \Elementor\Group_Control_Typography::get_type(),
        [
            'name'     => 'job_typography',
            'global'   => [ 'default' => \Elementor\Core\Kits\Documents\Tabs\Global_Typography::TYPOGRAPHY_SECONDARY ],
            'selector' => '{{WRAPPER}} .myplugin-testimonial-job',
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
    <div class="myplugin-testimonial">
        <div class="myplugin-testimonial-content">
            <?php echo wp_kses_post( $settings['testimonial_content'] ); ?>
        </div>
        <div class="myplugin-testimonial-meta">
            <?php if ( ! empty( $settings['testimonial_image']['url'] ) ) : ?>
                <div class="myplugin-testimonial-image">
                    <?php echo wp_kses_post( \Elementor\Group_Control_Image_Size::get_attachment_image_html( $settings, 'thumbnail', 'testimonial_image' ) ); ?>
                </div>
            <?php endif; ?>
            <div class="myplugin-testimonial-details">
                <?php if ( ! empty( $settings['testimonial_name'] ) ) : ?>
                    <div class="myplugin-testimonial-name"><?php echo esc_html( $settings['testimonial_name'] ); ?></div>
                <?php endif; ?>
                <?php if ( ! empty( $settings['testimonial_job'] ) ) : ?>
                    <div class="myplugin-testimonial-job"><?php echo esc_html( $settings['testimonial_job'] ); ?></div>
                <?php endif; ?>
            </div>
        </div>
    </div>
    <?php
}
```

> **Remove checklist:**
> - `testimonial_image` → remove if no portrait/avatar needed
> - `link` → remove if testimonial is not linked
> - `testimonial_job` → remove if job title is not required


**content_template() skeleton:**

```php
protected function content_template(): void {
    ?>
    <#
    // ✅ view.model replaces deprecated view.getEditModel() (removed in Elementor 3.x).
    // elementor.imagesManager.getImageUrl() reads image size from the model to pick srcset.
    var image = {
        id:        settings.testimonial_image.id,
        url:       settings.testimonial_image.url,
        size:      settings.testimonial_image_size,
        dimension: settings.testimonial_image_custom_dimension,
        model:     view.model,
    };
    var image_url = elementor.imagesManager.getImageUrl( image );
    #>
    <div class="myplugin-testimonial">
        <div class="myplugin-testimonial-content">{{{ settings.testimonial_content }}}</div>
        <div class="myplugin-testimonial-meta">
            <# if ( image_url ) { #>
            <div class="myplugin-testimonial-image"><img src="{{ image_url }}" alt=""></div>
            <# } #>
            <div class="myplugin-testimonial-details">
                <# if ( settings.testimonial_name ) { #>
                    <div class="myplugin-testimonial-name">{{{ settings.testimonial_name }}}</div>
                <# } #>
                <# if ( settings.testimonial_job ) { #>
                    <div class="myplugin-testimonial-job">{{{ settings.testimonial_job }}}</div>
                <# } #>
            </div>
        </div>
    </div>
    <?php
}
```
