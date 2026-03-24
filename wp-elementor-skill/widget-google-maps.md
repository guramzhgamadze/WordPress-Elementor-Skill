# Widget Boilerplate — Google Maps

> **When to use this file:** Load whenever building a widget that embeds a Google Maps iframe.
> Verified against `elementor/includes/widgets/google-maps.php` (Elementor 3.35+).

---

```php
protected function register_controls(): void {

    $this->start_controls_section( 'section_map', [
        'label' => esc_html__( 'Map', 'myplugin' ),
        'tab'   => \Elementor\Controls_Manager::TAB_CONTENT,
    ] );

    $this->add_control( 'address', [
        'label'       => esc_html__( 'Location', 'myplugin' ),
        'type'        => \Elementor\Controls_Manager::TEXT,
        'placeholder' => esc_html__( 'London Eye, London, UK', 'myplugin' ),
        'default'     => esc_html__( 'London Eye, London, UK', 'myplugin' ),
        'label_block' => true,
        'dynamic'     => [ 'active' => true ],
    ] );

    $this->add_control( 'zoom', [
        'label'   => esc_html__( 'Zoom', 'myplugin' ),
        'type'    => \Elementor\Controls_Manager::SLIDER,
        'default' => [ 'size' => 10 ],
        'range'   => [ 'px' => [ 'min' => 1, 'max' => 20 ] ],
    ] );

    $this->end_controls_section();

    // ── STYLE ─────────────────────────────────────────────────
    $this->start_controls_section( 'section_map_style', [
        'label' => esc_html__( 'Map', 'myplugin' ),
        'tab'   => \Elementor\Controls_Manager::TAB_STYLE,
    ] );

    $this->add_responsive_control( 'height', [
        'label'     => esc_html__( 'Height', 'myplugin' ),
        'type'      => \Elementor\Controls_Manager::SLIDER,
        'range'     => [ 'px' => [ 'min' => 40, 'max' => 1440 ] ],
        'default'   => [ 'size' => 300, 'unit' => 'px' ],
        'selectors' => [ '{{WRAPPER}} .myplugin-map' => 'height: {{SIZE}}{{UNIT}};' ],
    ] );

    $this->add_group_control(
        \Elementor\Group_Control_Css_Filter::get_type(),
        [ 'name' => 'map_filter', 'selector' => '{{WRAPPER}} .myplugin-map iframe' ]
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
    if ( empty( $settings['address'] ) ) return;

    $zoom    = (int) ( $settings['zoom']['size'] ?? 10 );
    $address = rawurlencode( $settings['address'] );

    // ✅ TWO URL PATTERNS — choose based on whether a Google API key is configured.
    //
    // PATTERN A (no API key) — legacy Maps embed, no key required, still functional:
    //   https://maps.google.com/maps?q={address}&t=m&z={zoom}&output=embed&iwloc=near
    //   This is the same fallback Elementor's native widget uses when no API key is set.
    //   It is NOT deprecated for no-key usage — Google still serves it and Elementor relies
    //   on it. Source: elementor/includes/widgets/google-maps.php (Elementor 3.35+)
    //
    // PATTERN B (with API key) — official Maps Embed API, recommended by Google:
    //   https://www.google.com/maps/embed/v1/place?key={API_KEY}&q={address}&zoom={zoom}
    //   Requires a Google Cloud project with Maps Embed API enabled (free, unlimited calls).
    //   Source: developers.google.com/maps/documentation/embed/get-started
    //
    // Use a plugin option or site setting to switch between patterns:
    $api_key = get_option( 'myplugin_google_maps_api_key', '' );

    if ( ! empty( $api_key ) ) {
        // Pattern B — Maps Embed API (preferred when key is available)
        $src = sprintf(
            'https://www.google.com/maps/embed/v1/place?key=%s&q=%s&zoom=%d',
            rawurlencode( sanitize_text_field( $api_key ) ),
            $address,
            $zoom
        );
    } else {
        // Pattern A — no-key fallback (still supported by Google, used by Elementor native)
        $src = sprintf(
            'https://maps.google.com/maps?q=%s&t=m&z=%d&output=embed&iwloc=near',
            $address,
            $zoom
        );
    }
    ?>
    <div class="myplugin-map">
        <iframe loading="lazy" src="<?php echo esc_url( $src ); ?>" title="<?php echo esc_attr( $settings['address'] ); ?>" aria-label="<?php echo esc_attr( $settings['address'] ); ?>"></iframe>
    </div>
    <?php
}
```


**content_template() skeleton:**

```php
protected function content_template(): void {
    ?>
    <# if ( settings.address ) { #>
    <div class="myplugin-map">
        <div class="myplugin-map-placeholder" style="display:flex;align-items:center;justify-content:center;background:#eee;height:300px;">
            <i class="eicon-map-pin" aria-hidden="true" style="font-size:48px;color:#aaa;"></i>
        </div>
    </div>
    <# } #>
    <?php
}
```
