# Widget Boilerplate — Counter

> **When to use this file:** Load whenever building a widget that animates a number counting up.
> Verified against `elementor/includes/widgets/counter.php` (Elementor 3.35+).

---

```php
protected function register_controls(): void {

    // ── CONTENT ───────────────────────────────────────────────
    $this->start_controls_section( 'section_counter', [
        'label' => esc_html__( 'Counter', 'myplugin' ),
        'tab'   => \Elementor\Controls_Manager::TAB_CONTENT,
    ] );

    $this->add_control( 'starting_number', [
        'label'   => esc_html__( 'Starting Number', 'myplugin' ),
        'type'    => \Elementor\Controls_Manager::NUMBER,
        'default' => 0,
        'dynamic' => [ 'active' => true ],
    ] );

    $this->add_control( 'ending_number', [
        'label'   => esc_html__( 'Ending Number', 'myplugin' ),
        'type'    => \Elementor\Controls_Manager::NUMBER,
        'default' => 100,
        'dynamic' => [ 'active' => true ],
    ] );

    $this->add_control( 'prefix', [
        'label'       => esc_html__( 'Number Prefix', 'myplugin' ),
        'type'        => \Elementor\Controls_Manager::TEXT,
        'default'     => '',
        'placeholder' => esc_html__( '$', 'myplugin' ),
        'dynamic'     => [ 'active' => true ],
    ] );

    $this->add_control( 'suffix', [
        'label'       => esc_html__( 'Number Suffix', 'myplugin' ),
        'type'        => \Elementor\Controls_Manager::TEXT,
        'default'     => '',
        'placeholder' => esc_html__( 'Plus', 'myplugin' ),
        'dynamic'     => [ 'active' => true ],
    ] );

    $this->add_control( 'duration', [
        'label'   => esc_html__( 'Animation Duration', 'myplugin' ),
        'type'    => \Elementor\Controls_Manager::NUMBER,
        'default' => 2000,
        'min'     => 100,
        'step'    => 100,
    ] );

    $this->add_control( 'thousand_separator', [
        'label'   => esc_html__( 'Thousand Separator', 'myplugin' ),
        'type'    => \Elementor\Controls_Manager::SWITCHER,
        'default' => 'yes',
    ] );

    $this->add_control( 'thousand_separator_char', [
        'label'     => esc_html__( 'Separator', 'myplugin' ),
        'type'      => \Elementor\Controls_Manager::SELECT,
        'condition' => [ 'thousand_separator' => 'yes' ],
        'options'   => [
            ''  => esc_html__( 'Default', 'myplugin' ),
            '.' => esc_html__( 'Dot',     'myplugin' ),
            ' ' => esc_html__( 'Space',   'myplugin' ),
        ],
    ] );

    $this->add_control( 'title', [
        'label'       => esc_html__( 'Title', 'myplugin' ),
        'type'        => \Elementor\Controls_Manager::TEXT,
        'label_block' => true,
        'default'     => esc_html__( 'Cool Number', 'myplugin' ),
        'placeholder' => esc_html__( 'Enter your counter title', 'myplugin' ),
        'separator'   => 'before',
        'dynamic'     => [ 'active' => true ],
    ] );

    $this->end_controls_section();

    // ── STYLE ─────────────────────────────────────────────────
    $this->start_controls_section( 'section_number', [
        'label' => esc_html__( 'Number', 'myplugin' ),
        'tab'   => \Elementor\Controls_Manager::TAB_STYLE,
    ] );

    $this->add_control( 'number_color', [
        'label'     => esc_html__( 'Text Color', 'myplugin' ),
        'type'      => \Elementor\Controls_Manager::COLOR,
        'global'    => [ 'default' => \Elementor\Core\Kits\Documents\Tabs\Global_Colors::COLOR_PRIMARY ],
        'selectors' => [ '{{WRAPPER}} .myplugin-counter-number-wrapper' => 'color: {{VALUE}};' ],
    ] );

    $this->add_group_control(
        \Elementor\Group_Control_Typography::get_type(),
        [
            'name'     => 'typography',
            'global'   => [ 'default' => \Elementor\Core\Kits\Documents\Tabs\Global_Typography::TYPOGRAPHY_PRIMARY ],
            'selector' => '{{WRAPPER}} .myplugin-counter-number-wrapper',
        ]
    );

    $this->end_controls_section();

    $this->start_controls_section( 'section_title', [
        'label' => esc_html__( 'Title', 'myplugin' ),
        'tab'   => \Elementor\Controls_Manager::TAB_STYLE,
    ] );

    $this->add_control( 'title_color', [
        'label'     => esc_html__( 'Text Color', 'myplugin' ),
        'type'      => \Elementor\Controls_Manager::COLOR,
        'global'    => [ 'default' => \Elementor\Core\Kits\Documents\Tabs\Global_Colors::COLOR_SECONDARY ],
        'selectors' => [ '{{WRAPPER}} .myplugin-counter-title' => 'color: {{VALUE}};' ],
    ] );

    $this->add_group_control(
        \Elementor\Group_Control_Typography::get_type(),
        [
            'name'     => 'title_typography',
            'global'   => [ 'default' => \Elementor\Core\Kits\Documents\Tabs\Global_Typography::TYPOGRAPHY_SECONDARY ],
            'selector' => '{{WRAPPER}} .myplugin-counter-title',
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
    $this->add_render_attribute( 'counter', [
        'class'                       => 'myplugin-counter-number',
        'data-duration'               => $settings['duration'],
        'data-to-value'               => $settings['ending_number'],
        'data-from-value'             => $settings['starting_number'],
        'data-delimiter'              => $settings['thousand_separator'] ? ( $settings['thousand_separator_char'] ?: ',' ) : '',
    ] );
    ?>
    <div class="myplugin-counter">
        <div class="myplugin-counter-number-wrapper">
            <span class="myplugin-counter-number-prefix"><?php echo esc_html( $settings['prefix'] ); ?></span>
            <span <?php $this->print_render_attribute_string( 'counter' ); ?>><?php echo esc_html( $settings['starting_number'] ); ?></span>
            <span class="myplugin-counter-number-suffix"><?php echo esc_html( $settings['suffix'] ); ?></span>
        </div>
        <?php if ( ! empty( $settings['title'] ) ) : ?>
            <p class="myplugin-counter-title"><?php echo esc_html( $settings['title'] ); ?></p>
        <?php endif; ?>
    </div>
    <?php
}
```

> ⚠️ **Required: Counter JS animation.** The `data-duration`, `data-to-value`, and
> `data-from-value` attributes output by `render()` do nothing without a JS driver.
> Use an `IntersectionObserver` to start the animation when the widget scrolls into view:
> ```js
> ( () => {
>   'use strict';
>   const animateCounter = ( el ) => {
>     const from     = +el.dataset.fromValue || 0;
>     const to       = +el.dataset.toValue   || 0;
>     const duration = +el.dataset.duration  || 2000;
>     const delim    = el.dataset.delimiter  || '';
>     const start    = performance.now();
>     const tick = ( now ) => {
>       const elapsed  = Math.min( now - start, duration );
>       const progress = elapsed / duration;
>       const current  = Math.round( from + ( to - from ) * progress );
>       el.textContent = delim
>         ? current.toLocaleString()
>         : String( current );
>       if ( elapsed < duration ) requestAnimationFrame( tick );
>     };
>     // ✅ Respect prefers-reduced-motion — skip animation, jump to final value
>     if ( window.matchMedia( '(prefers-reduced-motion: reduce)' ).matches ) {
>       el.textContent = String( to );
>       return;
>     }
>     requestAnimationFrame( tick );
>   };
>
>   window.addEventListener( 'elementor/frontend/init', () => {
>     window.elementorFrontend.hooks.addAction(
>       'frontend/element_ready/myplugin-widget.default',
>       ( $scope ) => {
>         if ( ! $scope || ! $scope[0] ) return;
>         const counter = $scope[0].querySelector( '.myplugin-counter-number' );
>         if ( ! counter ) return;
>         const observer = new IntersectionObserver( ( entries ) => {
>           entries.forEach( entry => {
>             if ( entry.isIntersecting ) {
>               animateCounter( counter );
>               observer.unobserve( entry.target );
>             }
>           } );
>         }, { threshold: 0.5 } );
>         observer.observe( counter );
>       }
>     );
>   } );
> } )();
> ```

> **Remove checklist:**
> - `prefix` + `suffix` → remove if counter shows a plain number only
> - `thousand_separator` → remove if numbers are always small
> - `title` → remove if counter has no label


**content_template() skeleton:**

```php
protected function content_template(): void {
    ?>
    <div class="myplugin-counter">
        <div class="myplugin-counter-number-wrapper">
            <span class="myplugin-counter-number-prefix">{{{ settings.prefix }}}</span>
            <span class="myplugin-counter-number"
                data-duration="{{ settings.duration }}"
                data-to-value="{{ settings.ending_number }}"
                data-from-value="{{ settings.starting_number }}">{{{ settings.starting_number }}}</span>
            <span class="myplugin-counter-number-suffix">{{{ settings.suffix }}}</span>
        </div>
        <# if ( settings.title ) { #>
            <p class="myplugin-counter-title">{{{ settings.title }}}</p>
        <# } #>
    </div>
    <?php
}
```
