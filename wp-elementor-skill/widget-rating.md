# Widget Boilerplate — Rating

> **When to use this file:** Load whenever building a structured data / schema-ready rating widget.
> Verified against `elementor/includes/widgets/rating.php` (Elementor 3.35+).
> ⚠️ This is the newer Rating widget (added ~3.17) — distinct from Star Rating. It supports
> schema markup and fractional ratings. Use Star Rating for purely decorative displays.

---

```php
protected function register_controls(): void {

    // ── CONTENT ───────────────────────────────────────────────
    $this->start_controls_section( 'section_rating', [
        'label' => esc_html__( 'Rating', 'myplugin' ),
        'tab'   => \Elementor\Controls_Manager::TAB_CONTENT,
    ] );

    $this->add_control( 'rating_type', [
        'label'   => esc_html__( 'Rating Type', 'myplugin' ),
        'type'    => \Elementor\Controls_Manager::SELECT,
        'default' => 'number',
        'options' => [
            'number'  => esc_html__( 'Number',   'myplugin' ),
            'percent' => esc_html__( 'Percent',  'myplugin' ),
        ],
    ] );

    $this->add_control( 'rating', [
        'label'   => esc_html__( 'Rating', 'myplugin' ),
        'type'    => \Elementor\Controls_Manager::NUMBER,
        'min'     => 0,
        'max'     => 10,
        'step'    => 0.1,
        'default' => 5,
        'dynamic' => [ 'active' => true ],
    ] );

    $this->add_control( 'icon', [
        'label'            => esc_html__( 'Icon', 'myplugin' ),
        'type'             => \Elementor\Controls_Manager::ICONS,
        'fa4compatibility' => 'icon',
        'default'          => [ 'value' => 'fas fa-star', 'library' => 'fa-solid' ],
        'separator'        => 'before',
    ] );

    $this->add_control( 'icon_count', [
        'label'   => esc_html__( 'Icon Count', 'myplugin' ),
        'type'    => \Elementor\Controls_Manager::NUMBER,
        'min'     => 1,
        'max'     => 10,
        'default' => 5,
    ] );

    $this->end_controls_section();

    // ── STYLE ─────────────────────────────────────────────────
    $this->start_controls_section( 'section_style_rating', [
        'label' => esc_html__( 'Rating', 'myplugin' ),
        'tab'   => \Elementor\Controls_Manager::TAB_STYLE,
    ] );

    $this->add_responsive_control( 'align', [
        'label'     => esc_html__( 'Alignment', 'myplugin' ),
        'type'      => \Elementor\Controls_Manager::CHOOSE,
        'options'   => [
            'left'   => [ 'title' => esc_html__( 'Left',   'myplugin' ), 'icon' => 'eicon-text-align-left'   ],
            'center' => [ 'title' => esc_html__( 'Center', 'myplugin' ), 'icon' => 'eicon-text-align-center' ],
            'right'  => [ 'title' => esc_html__( 'Right',  'myplugin' ), 'icon' => 'eicon-text-align-right'  ],
        ],
        'selectors' => [ '{{WRAPPER}}' => 'text-align: {{VALUE}};' ],
    ] );

    $this->add_responsive_control( 'icon_size', [
        'label'     => esc_html__( 'Icon Size', 'myplugin' ),
        'type'      => \Elementor\Controls_Manager::SLIDER,
        'range'     => [ 'px' => [ 'min' => 10, 'max' => 100 ] ],
        'default'   => [ 'size' => 20, 'unit' => 'px' ],
        'selectors' => [
            '{{WRAPPER}} .myplugin-rating-icon i'   => 'font-size: {{SIZE}}{{UNIT}};',
            '{{WRAPPER}} .myplugin-rating-icon svg' => 'width: {{SIZE}}{{UNIT}}; height: {{SIZE}}{{UNIT}};',
        ],
    ] );

    $this->add_responsive_control( 'icon_gap', [
        'label'     => esc_html__( 'Icon Gap', 'myplugin' ),
        'type'      => \Elementor\Controls_Manager::SLIDER,
        'range'     => [ 'px' => [ 'min' => 0, 'max' => 30 ] ],
        'default'   => [ 'size' => 4, 'unit' => 'px' ],
        'selectors' => [
            '{{WRAPPER}} .myplugin-rating-wrapper' => 'gap: {{SIZE}}{{UNIT}};',
        ],
    ] );

    $this->add_control( 'icon_color', [
        'label'     => esc_html__( 'Marked Color', 'myplugin' ),
        'type'      => \Elementor\Controls_Manager::COLOR,
        'default'   => '#f0ad4e',
        'selectors' => [
            '{{WRAPPER}} .myplugin-rating-icon--full i'   => 'color: {{VALUE}};',
            '{{WRAPPER}} .myplugin-rating-icon--full svg' => 'fill: {{VALUE}};',
        ],
        'separator' => 'before',
    ] );

    $this->add_control( 'icon_unmarked_color', [
        'label'     => esc_html__( 'Unmarked Color', 'myplugin' ),
        'type'      => \Elementor\Controls_Manager::COLOR,
        'default'   => '#d0d0d0',
        'selectors' => [
            '{{WRAPPER}} .myplugin-rating-icon--empty i'   => 'color: {{VALUE}};',
            '{{WRAPPER}} .myplugin-rating-icon--empty svg' => 'fill: {{VALUE}};',
        ],
    ] );

    $this->end_controls_section();
}
```

**render() + content_template() skeleton:**

```php
public function has_widget_inner_wrapper(): bool {
    return false;
}

protected function is_dynamic_content(): bool {
    return false;
}

protected function render(): void {
    $settings = $this->get_settings_for_display();

    $rating     = (float) ( $settings['rating'] ?? 0 );
    $icon_count = (int) ( $settings['icon_count'] ?? 5 );

    // Normalise to 0–icon_count range
    if ( 'percent' === $settings['rating_type'] ) {
        $rating = ( $rating / 100 ) * $icon_count;
    } else {
        $rating = min( $rating, 10 ); // max scale is 10
    }

    $full_icons  = floor( $rating );
    $partial     = $rating - $full_icons; // fractional part 0–1
    $empty_icons = $icon_count - $full_icons - ( $partial > 0 ? 1 : 0 );
    ?>
    <div class="myplugin-rating"
         role="img"
         aria-label="<?php echo esc_attr( round( $rating, 1 ) . '/' . $icon_count ); ?>">
        <div class="myplugin-rating__wrapper">
            <?php
            // Full icons
            for ( $i = 0; $i < $full_icons; $i++ ) :
                echo '<span class="myplugin-rating-icon myplugin-rating-icon--full">';
                \Elementor\Icons_Manager::render_icon( $settings['icon'], [ 'aria-hidden' => 'true' ] );
                echo '</span>';
            endfor;

            // Partial icon — clip span to fractional width.
            // ✅ Required CSS for partial clip to work:
            //    .myplugin-rating-icon--partial { position: relative; overflow: hidden; display: inline-block; }
            //    .myplugin-rating-icon--partial i,
            //    .myplugin-rating-icon--partial svg { position: absolute; left: 0; }
            // Without overflow:hidden the full icon shows even when width is e.g. 30%.
            if ( $partial > 0 ) :
                echo '<span class="myplugin-rating-icon myplugin-rating-icon--partial" style="width:' . esc_attr( round( $partial * 100 ) ) . '%">';
                \Elementor\Icons_Manager::render_icon( $settings['icon'], [ 'aria-hidden' => 'true' ] );
                echo '</span>';
            endif;

            // Empty icons
            for ( $i = 0; $i < $empty_icons; $i++ ) :
                echo '<span class="myplugin-rating-icon myplugin-rating-icon--empty">';
                \Elementor\Icons_Manager::render_icon( $settings['icon'], [ 'aria-hidden' => 'true' ] );
                echo '</span>';
            endfor;
            ?>
        </div>
    </div>
    <?php
}

protected function content_template(): void {
    ?>
    <#
    var rating    = parseFloat( settings.rating ) || 0;
    var iconCount = parseInt( settings.icon_count ) || 5;
    if ( 'percent' === settings.rating_type ) { rating = ( rating / 100 ) * iconCount; }
    else { rating = Math.min( rating, 10 ); }
    var full    = Math.floor( rating );
    var partial = rating - full;
    var empty   = iconCount - full - ( partial > 0 ? 1 : 0 );
    var iconHTML = elementor.helpers.renderIcon( view, settings.icon, { 'aria-hidden': true }, 'i', 'object' );
    var icon = iconHTML && iconHTML.value ? iconHTML.value : '<i class="fas fa-star" aria-hidden="true"></i>'; // ✅ FA fallback, not eicon-star (eicons not reliable on frontend)
    var stars = '';
    for ( var i = 0; i < full; i++ )    { stars += '<span class="myplugin-rating-icon myplugin-rating-icon--full">'    + icon + '</span>'; }
    if ( partial > 0 )                   { stars += '<span class="myplugin-rating-icon myplugin-rating-icon--partial" style="width:' + Math.round(partial*100) + '%">' + icon + '</span>'; }
    for ( var j = 0; j < empty; j++ )   { stars += '<span class="myplugin-rating-icon myplugin-rating-icon--empty">'   + icon + '</span>'; }
    #>
    <div class="myplugin-rating">
        <div class="myplugin-rating__wrapper">{{{ stars }}}</div>
    </div>
    <?php
}
```
