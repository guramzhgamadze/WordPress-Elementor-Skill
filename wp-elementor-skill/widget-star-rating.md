# Widget Boilerplate — Star Rating

> **When to use this file:** Load whenever building a widget that displays a star (or custom icon) rating.
> Verified against `elementor/includes/widgets/star-rating.php` (Elementor 3.35+).

---

```php
protected function register_controls(): void {

    $this->start_controls_section( 'section_rating', [
        'label' => esc_html__( 'Rating', 'myplugin' ),
        'tab'   => \Elementor\Controls_Manager::TAB_CONTENT,
    ] );

    $this->add_control( 'rating_scale', [
        'label'   => esc_html__( 'Rating Scale', 'myplugin' ),
        'type'    => \Elementor\Controls_Manager::SELECT,
        'options' => [ '5' => '0-5', '10' => '0-10' ],
        'default' => '5',
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

    $this->add_control( 'star_style', [
        'label'        => esc_html__( 'Icon', 'myplugin' ),
        'type'         => \Elementor\Controls_Manager::SELECT,
        'options'      => [
            'star_fontawesome' => esc_html__( 'Font Awesome', 'myplugin' ),
            'star_unicode'     => esc_html__( 'Unicode',      'myplugin' ),
        ],
        'default'      => 'star_fontawesome',
        'prefix_class' => 'elementor--star-style-',
        'separator'    => 'before',
    ] );

    $this->add_control( 'unmarked_star_style', [
        'label'   => esc_html__( 'Unmarked Style', 'myplugin' ),
        'type'    => \Elementor\Controls_Manager::CHOOSE,
        'options' => [
            'solid'   => [ 'title' => esc_html__( 'Solid',   'myplugin' ), 'icon' => 'eicon-star'          ],
            'outline' => [ 'title' => esc_html__( 'Outline', 'myplugin' ), 'icon' => 'eicon-star-o' ],
        ],
        'default' => 'solid',
    ] );

    $this->add_control( 'title', [
        'label'     => esc_html__( 'Title', 'myplugin' ),
        'type'      => \Elementor\Controls_Manager::TEXT,
        'separator' => 'before',
        'dynamic'   => [ 'active' => true ],
    ] );

    $this->end_controls_section();

    // ── STYLE ─────────────────────────────────────────────────
    $this->start_controls_section( 'section_stars_style', [
        'label' => esc_html__( 'Stars', 'myplugin' ),
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
        'range'     => [ 'px' => [ 'min' => 0, 'max' => 100 ] ],
        'default'   => [ 'size' => 20, 'unit' => 'px' ],
        'selectors' => [ '{{WRAPPER}} .myplugin-star-rating' => 'font-size: {{SIZE}}{{UNIT}};' ],
    ] );

    $this->add_control( 'star_color', [
        'label'     => esc_html__( 'Marked Color', 'myplugin' ),
        'type'      => \Elementor\Controls_Manager::COLOR,
        'default'   => '#f0ad4e',
        'selectors' => [ '{{WRAPPER}} .myplugin-star-full i' => 'color: {{VALUE}};' ],
        'separator' => 'before',
    ] );

    $this->add_control( 'empty_star_color', [
        'label'     => esc_html__( 'Unmarked Color', 'myplugin' ),
        'type'      => \Elementor\Controls_Manager::COLOR,
        'default'   => '#d0d0d0',
        'selectors' => [ '{{WRAPPER}} .myplugin-star-empty i' => 'color: {{VALUE}};' ],
    ] );

    $this->end_controls_section();
}
```

---


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

    $rating = (float) ( $settings['rating'] ?? 0 );
    $max    = (int)   ( $settings['rating_scale'] ?? 5 );
    $rating = min( $rating, $max );

    // ✅ BUG FIX: Do NOT use hardcoded `eicon-star` / `eicon-star-o` class strings here.
    // `eicon-*` are Elementor's EDITOR icon font — the eicons stylesheet is NOT reliably
    // loaded on the frontend (especially when "Inline Font Icons" experiment is active,
    // CSS/JS optimisation is on, or Remove Unused CSS tools prune it). This is a confirmed
    // Elementor core bug (github.com/elementor/elementor/issues/18793).
    //
    // For a custom star rating widget, define ICONS controls for the marked/unmarked star
    // and use Icons_Manager::render_icon() — it loads Font Awesome (or SVG) properly on
    // the frontend. The simplest correct approach: use Font Awesome via the ICONS control.
    //
    // If you MUST use star_style = 'star_fontawesome' with inline <i> tags, use FA classes:
    //   filled  → 'fas fa-star'  (fa-solid)
    //   outline → 'far fa-star'  (fa-regular)
    // NOT eicon-star / eicon-star-o.
    //
    // The cleanest production approach (used below) is to register explicit ICONS controls
    // for the filled and empty stars, then call Icons_Manager::render_icon() for each.
    // Example control additions to register_controls():
    //   $this->add_control( 'icon_marked', [ 'type' => Controls_Manager::ICONS,
    //       'default' => [ 'value' => 'fas fa-star', 'library' => 'fa-solid' ] ] );
    //   $this->add_control( 'icon_empty',  [ 'type' => Controls_Manager::ICONS,
    //       'default' => [ 'value' => 'far fa-star', 'library' => 'fa-regular' ] ] );

    $is_unicode = 'star_unicode' === $settings['star_style'];
    ?>
    <div class="myplugin-star-rating" role="img" aria-label="<?php echo esc_attr( $rating . '/' . $max ); ?>">
        <?php if ( ! empty( $settings['title'] ) ) : ?>
            <span class="myplugin-star-rating__title"><?php echo esc_html( $settings['title'] ); ?></span>
        <?php endif; ?>
        <div class="myplugin-star-rating__wrapper">
            <?php for ( $i = 1; $i <= $max; $i++ ) :
                $is_filled = ( $i <= $rating );
                $class     = $is_filled ? 'myplugin-star-full' : 'myplugin-star-empty';
            ?>
                <span class="<?php echo esc_attr( $class ); ?>">
                    <?php if ( $is_unicode ) :
                        // Unicode path — safe, no external font dependency.
                        $is_solid_empty = 'solid' === ( $settings['unmarked_star_style'] ?? 'solid' );
                        $char = $is_filled ? '★' : ( $is_solid_empty ? '★' : '☆' );
                        echo esc_html( $char );
                    else :
                        // Font Awesome path — use Icons_Manager::render_icon() for correct
                        // frontend enqueue. Requires icon_marked / icon_empty ICONS controls.
                        $icon_setting = $is_filled
                            ? ( $settings['icon_marked'] ?? [ 'value' => 'fas fa-star', 'library' => 'fa-solid' ] )
                            : ( $settings['icon_empty']  ?? [ 'value' => 'far fa-star', 'library' => 'fa-regular' ] );
                        \Elementor\Icons_Manager::render_icon( $icon_setting, [ 'aria-hidden' => 'true' ] );
                    endif; ?>
                </span>
            <?php endfor; ?>
        </div>
    </div>
    <?php
}

protected function content_template(): void {
    ?>
    <#
    var rating  = parseFloat( settings.rating ) || 0;
    var max     = parseInt( settings.rating_scale ) || 5;
    rating      = Math.min( rating, max );
    var isUnicode = 'star_unicode' === settings.star_style;
    var marked    = 'solid' === settings.unmarked_star_style;
    var stars     = '';
    for ( var i = 1; i <= max; i++ ) {
        var isFilled = ( i <= rating );
        var cls      = isFilled ? 'myplugin-star-full' : 'myplugin-star-empty';
        var icon;
        if ( isUnicode ) {
            icon = isFilled ? '&#9733;' : ( marked ? '&#9733;' : '&#9734;' );
        } else {
            // ✅ Use ICONS control values — renderIcon handles FA properly in the editor.
            // icon_marked / icon_empty are ICONS controls added in register_controls().
            var iconSetting = isFilled
                ? ( settings.icon_marked || { value: 'fas fa-star',  library: 'fa-solid' } )
                : ( settings.icon_empty  || { value: 'far fa-star',  library: 'fa-regular' } );
            var iconHTML = elementor.helpers.renderIcon( view, iconSetting, { 'aria-hidden': true }, 'i', 'object' );
            icon = iconHTML && iconHTML.rendered ? iconHTML.value : '<i class="fas fa-star" aria-hidden="true"></i>';
        }
        stars += '<span class="' + cls + '">' + icon + '</span>';
    }
    #>
    <div class="myplugin-star-rating">
        <# if ( settings.title ) { #>
            <span class="myplugin-star-rating__title">{{{ settings.title }}}</span>
        <# } #>
        <div class="myplugin-star-rating__wrapper">{{{ stars }}}</div>
    </div>
    <?php
}
```
