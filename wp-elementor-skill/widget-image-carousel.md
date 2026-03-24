# Widget Boilerplate — Image Carousel

> **When to use this file:** Load whenever building a widget showing a rotating image slider/carousel.
> Verified against `elementor/includes/widgets/image-carousel.php` (Elementor 3.35+).
> ✅ Relies on Swiper JS (bundled with Elementor) for carousel behaviour.

---

```php
protected function register_controls(): void {

    // ── CONTENT ───────────────────────────────────────────────
    $this->start_controls_section( 'section_image_carousel', [
        'label' => esc_html__( 'Image Carousel', 'myplugin' ),
        'tab'   => \Elementor\Controls_Manager::TAB_CONTENT,
    ] );

    $this->add_control( 'carousel', [
        'label'   => esc_html__( 'Add Images', 'myplugin' ),
        'type'    => \Elementor\Controls_Manager::GALLERY,
        'default' => [],
        'dynamic' => [ 'active' => true ],
    ] );

    $this->add_group_control(
        \Elementor\Group_Control_Image_Size::get_type(),
        [ 'name' => 'thumbnail', 'default' => 'medium' ]
    );

    $this->add_responsive_control( 'slides_to_show', [
        'label'   => esc_html__( 'Slides to Show', 'myplugin' ),
        'type'    => \Elementor\Controls_Manager::SELECT,
        'default' => '3',
        'options' => [ '1' => '1', '2' => '2', '3' => '3', '4' => '4', '5' => '5', '6' => '6' ],
    ] );

    $this->add_control( 'slides_to_scroll', [
        'label'   => esc_html__( 'Slides to Scroll', 'myplugin' ),
        'type'    => \Elementor\Controls_Manager::SELECT,
        'default' => '1',
        'options' => [ '1' => '1', '2' => '2', '3' => '3' ],
    ] );

    $this->add_control( 'image_carousel_link', [
        'label'   => esc_html__( 'Link', 'myplugin' ),
        'type'    => \Elementor\Controls_Manager::SELECT,
        'default' => 'none',
        'options' => [
            'none'   => esc_html__( 'None',        'myplugin' ),
            'file'   => esc_html__( 'Media File',  'myplugin' ),
            'custom' => esc_html__( 'Custom URL',  'myplugin' ),
        ],
    ] );

    $this->add_control( 'link_to', [
        'label'       => esc_html__( 'Link', 'myplugin' ),
        'type'        => \Elementor\Controls_Manager::URL,
        'placeholder' => esc_html__( 'https://your-link.com', 'myplugin' ),
        'condition'   => [ 'image_carousel_link' => 'custom' ],
        'dynamic'     => [ 'active' => true ],
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
        'condition' => [ 'image_carousel_link' => 'file' ],
    ] );

    $this->end_controls_section();

    // ── CONTENT: Additional Options ───────────────────────────
    $this->start_controls_section( 'section_additional_options', [
        'label' => esc_html__( 'Additional Options', 'myplugin' ),
        'tab'   => \Elementor\Controls_Manager::TAB_CONTENT,
    ] );

    $this->add_control( 'autoplay', [
        'label'   => esc_html__( 'Autoplay', 'myplugin' ),
        'type'    => \Elementor\Controls_Manager::SWITCHER,
        'default' => 'yes',
    ] );

    $this->add_control( 'pause_on_hover', [
        'label'     => esc_html__( 'Pause on Hover', 'myplugin' ),
        'type'      => \Elementor\Controls_Manager::SWITCHER,
        'default'   => 'yes',
        'condition' => [ 'autoplay' => 'yes' ],
    ] );

    $this->add_control( 'autoplay_speed', [
        'label'     => esc_html__( 'Autoplay Speed', 'myplugin' ),
        'type'      => \Elementor\Controls_Manager::NUMBER,
        'default'   => 5000,
        'condition' => [ 'autoplay' => 'yes' ],
    ] );

    $this->add_control( 'infinite', [
        'label'   => esc_html__( 'Infinite Loop', 'myplugin' ),
        'type'    => \Elementor\Controls_Manager::SWITCHER,
        'default' => 'yes',
    ] );

    $this->add_control( 'transition', [
        'label'   => esc_html__( 'Transition', 'myplugin' ),
        'type'    => \Elementor\Controls_Manager::SELECT,
        'default' => 'slide',
        'options' => [
            'slide' => esc_html__( 'Slide', 'myplugin' ),
            'fade'  => esc_html__( 'Fade',  'myplugin' ),
        ],
    ] );

    $this->add_control( 'transition_speed', [
        'label'   => esc_html__( 'Transition Speed', 'myplugin' ),
        'type'    => \Elementor\Controls_Manager::NUMBER,
        'default' => 500,
    ] );

    $this->add_control( 'navigation', [
        'label'   => esc_html__( 'Navigation', 'myplugin' ),
        'type'    => \Elementor\Controls_Manager::SELECT,
        'default' => 'arrows',
        'options' => [
            'both'   => esc_html__( 'Arrows and Dots', 'myplugin' ),
            'arrows' => esc_html__( 'Arrows',           'myplugin' ),
            'dots'   => esc_html__( 'Dots',             'myplugin' ),
            'none'   => esc_html__( 'None',             'myplugin' ),
        ],
    ] );

    $this->end_controls_section();

    // ── STYLE ─────────────────────────────────────────────────
    $this->start_controls_section( 'section_style_navigation', [
        'label' => esc_html__( 'Navigation', 'myplugin' ),
        'tab'   => \Elementor\Controls_Manager::TAB_STYLE,
    ] );

    $this->add_control( 'heading_arrows', [
        'label' => esc_html__( 'Arrows', 'myplugin' ),
        'type'  => \Elementor\Controls_Manager::HEADING,
    ] );

    $this->add_responsive_control( 'arrows_size', [
        'label'     => esc_html__( 'Size', 'myplugin' ),
        'type'      => \Elementor\Controls_Manager::SLIDER,
        'default'   => [ 'size' => 20, 'unit' => 'px' ],
        'range'     => [ 'px' => [ 'min' => 10, 'max' => 100 ] ],
        'selectors' => [ '{{WRAPPER}} .swiper-button-prev, {{WRAPPER}} .swiper-button-next' => 'font-size: {{SIZE}}{{UNIT}};' ],
    ] );

    $this->add_control( 'arrows_color', [
        'label'     => esc_html__( 'Color', 'myplugin' ),
        'type'      => \Elementor\Controls_Manager::COLOR,
        'selectors' => [ '{{WRAPPER}} .swiper-button-prev, {{WRAPPER}} .swiper-button-next' => 'color: {{VALUE}};' ],
    ] );

    $this->end_controls_section();

    $this->start_controls_section( 'section_style_image', [
        'label' => esc_html__( 'Image', 'myplugin' ),
        'tab'   => \Elementor\Controls_Manager::TAB_STYLE,
    ] );

    $this->add_responsive_control( 'image_border_radius', [
        'label'      => esc_html__( 'Border Radius', 'myplugin' ),
        'type'       => \Elementor\Controls_Manager::DIMENSIONS,
        'size_units' => [ 'px', '%', 'em', 'rem' ],
        'selectors'  => [
            '{{WRAPPER}} .myplugin-carousel-image' => 'border-radius: {{TOP}}{{UNIT}} {{RIGHT}}{{UNIT}} {{BOTTOM}}{{UNIT}} {{LEFT}}{{UNIT}};',
        ],
    ] );

    $this->end_controls_section();
}
```

> **Remove checklist:**
> - `open_lightbox` → remove if carousel images don't link to media file
> - `pause_on_hover` → remove if autoplay is always off
> - Navigation style section → remove if navigation is always `none`

**render() + content_template() skeleton:**

```php
// ✅ REQUIRED: Declare Swiper as a dependency so Elementor enqueues it only on pages
// using this widget. Swiper is bundled with Elementor — do NOT register it yourself.
// Source: developers.elementor.com/docs/widgets/widget-dependencies/
public function get_script_depends(): array {
    return [ 'swiper' ];
}

public function get_style_depends(): array {
    return [ 'swiper' ];
}

public function has_widget_inner_wrapper(): bool {
    return false;
}

protected function is_dynamic_content(): bool {
    return false;
}

protected function render(): void {
    $settings = $this->get_settings_for_display();

    if ( empty( $settings['carousel'] ) ) {
        return;
    }

    $show_arrows = in_array( $settings['navigation'], [ 'arrows', 'both' ], true );
    $show_dots   = in_array( $settings['navigation'], [ 'dots', 'both' ], true );

    // ✅ Swiper is bundled with Elementor — declare 'swiper' as a script dependency
    // in get_script_depends() so Elementor loads it only on pages using this widget.
    $this->add_render_attribute( 'carousel-wrapper', [
        'class'                => 'myplugin-image-carousel swiper',
        'data-autoplay'        => 'yes' === $settings['autoplay'] ? $settings['autoplay_speed'] : '0',
        'data-loop'            => 'yes' === $settings['infinite'] ? 'true' : 'false',
        'data-slides-to-show'  => $settings['slides_to_show'],
        'data-slides-to-scroll'=> $settings['slides_to_scroll'],
        'data-effect'          => $settings['transition'],
        'data-speed'           => $settings['transition_speed'],
        'data-pause-on-hover'  => 'yes' === ( $settings['pause_on_hover'] ?? '' ) ? 'true' : 'false',
    ] );
    ?>
    <div class="myplugin-carousel-wrapper">
        <div <?php $this->print_render_attribute_string( 'carousel-wrapper' ); ?>>
            <div class="swiper-wrapper">
                <?php foreach ( $settings['carousel'] as $attachment ) :
                    $image_html = \Elementor\Group_Control_Image_Size::get_attachment_image_html( $settings, 'thumbnail', $attachment );
                    if ( empty( $image_html ) ) continue;

                    $link_tag_open  = '';
                    $link_tag_close = '';

                    if ( 'file' === $settings['image_carousel_link'] ) {
                        $open_lb = esc_attr( $settings['open_lightbox'] ?? 'default' );
                        $link_tag_open  = '<a href="' . esc_url( wp_get_attachment_url( $attachment['id'] ) ) . '" data-elementor-open-lightbox="' . $open_lb . '">';
                        $link_tag_close = '</a>';
                    } elseif ( 'custom' === $settings['image_carousel_link'] && ! empty( $settings['link_to']['url'] ) ) {
                        $this->add_link_attributes( 'carousel-link', $settings['link_to'] );
                        $link_tag_open  = '<a ' . $this->get_render_attribute_string( 'carousel-link' ) . '>';
                        $link_tag_close = '</a>';
                    }
                    ?>
                    <div class="swiper-slide">
                        <figure class="myplugin-carousel-image">
                            <?php echo $link_tag_open; // phpcs:ignore WordPress.Security.EscapeOutput ?>
                                <?php echo wp_kses_post( $image_html ); ?>
                            <?php echo $link_tag_close; // phpcs:ignore WordPress.Security.EscapeOutput ?>
                        </figure>
                    </div>
                <?php endforeach; ?>
            </div>

            <?php if ( $show_dots ) : ?>
                <div class="swiper-pagination"></div>
            <?php endif; ?>
        </div>

        <?php if ( $show_arrows ) : ?>
            <div class="swiper-button-prev"></div>
            <div class="swiper-button-next"></div>
        <?php endif; ?>
    </div>
    <?php
}

protected function content_template(): void {
    ?>
    <#
    if ( ! settings.carousel || ! settings.carousel.length ) { return; }
    var showArrows = -1 !== [ 'arrows', 'both' ].indexOf( settings.navigation );
    var showDots   = -1 !== [ 'dots',   'both' ].indexOf( settings.navigation );
    #>
    <div class="myplugin-carousel-wrapper">
        <div class="myplugin-image-carousel swiper">
            <div class="swiper-wrapper">
                <# _.each( settings.carousel, function( attachment ) { #>
                <div class="swiper-slide">
                    <figure class="myplugin-carousel-image">
                        <img src="{{ attachment.url }}" alt="">
                    </figure>
                </div>
                <# } ); #>
            </div>
            <# if ( showDots ) { #><div class="swiper-pagination"></div><# } #>
        </div>
        <# if ( showArrows ) { #>
            <div class="swiper-button-prev"></div>
            <div class="swiper-button-next"></div>
        <# } #>
    </div>
    <?php
}
```

> **Script dependency note:** Add `'swiper'` to `get_script_depends()` so Elementor loads
> the bundled Swiper JS only on pages containing this widget.

> ⚠️ **Required: Swiper JS initialization.** The `.swiper` markup alone renders as a static list.
> You MUST initialize Swiper via JS in your widget's frontend script. Declare `'swiper'` and
> `'elementor-frontend'` as dependencies in `get_script_depends()`, then initialize per-instance
> inside the `elementor/frontend/init` handler:
> ```js
> ( () => {
>   'use strict';
>   const init = ( scope ) => {
>     const el = scope.querySelector( '.myplugin-image-carousel.swiper' );
>     if ( ! el ) return;
>     const nav  = scope.dataset.navigation || 'none';  // read from data-* if needed
>     new Swiper( el, {
>       slidesPerView: 1,
>       loop:          true,
>       navigation: ( nav === 'arrows' || nav === 'both' ) ? {
>         nextEl: scope.querySelector( '.swiper-button-next' ),
>         prevEl: scope.querySelector( '.swiper-button-prev' ),
>       } : false,
>       pagination: ( nav === 'dots' || nav === 'both' ) ? {
>         el:        scope.querySelector( '.swiper-pagination' ),
>         clickable: true,
>       } : false,
>       // ✅ Respect prefers-reduced-motion — do not autoplay animations for users who opt out
>       autoplay: window.matchMedia( '(prefers-reduced-motion: reduce)' ).matches
>         ? false
>         : { delay: 3000, disableOnInteraction: true },
>     } );
>   };
>
>   window.addEventListener( 'elementor/frontend/init', () => {
>     window.elementorFrontend.hooks.addAction(
>       'frontend/element_ready/myplugin-widget.default',
>       ( $scope ) => {
>         if ( ! $scope || ! $scope[0] ) return;
>         init( $scope[0] );
>       }
>     );
>   } );
> } )();
> ```
> **Swiper version note:** Elementor bundles Swiper 8.x (class `.swiper`, NOT the legacy
> `.swiper-container`). Do NOT register your own Swiper copy — use `'swiper'` as a dependency
> to guarantee version consistency with Elementor's bundled copy.
