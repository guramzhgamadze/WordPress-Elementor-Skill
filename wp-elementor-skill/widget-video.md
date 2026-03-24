# Widget Boilerplate — Video

> **When to use this file:** Load whenever building a widget that embeds a video (YouTube, Vimeo, self-hosted).
> Verified against `elementor/includes/widgets/video.php` (Elementor 3.35+).

---

```php
protected function register_controls(): void {

    // ── CONTENT ───────────────────────────────────────────────
    $this->start_controls_section( 'section_video', [
        'label' => esc_html__( 'Video', 'myplugin' ),
        'tab'   => \Elementor\Controls_Manager::TAB_CONTENT,
    ] );

    $this->add_control( 'video_type', [
        'label'   => esc_html__( 'Source', 'myplugin' ),
        'type'    => \Elementor\Controls_Manager::SELECT,
        'default' => 'youtube',
        'options' => [
            'youtube'     => esc_html__( 'YouTube',      'myplugin' ),
            'vimeo'       => esc_html__( 'Vimeo',        'myplugin' ),
            'dailymotion' => esc_html__( 'Dailymotion',  'myplugin' ),
            'hosted'      => esc_html__( 'Self Hosted',  'myplugin' ),
        ],
    ] );

    $this->add_control( 'youtube_url', [
        'label'       => esc_html__( 'Link', 'myplugin' ),
        'type'        => \Elementor\Controls_Manager::TEXT,
        'default'     => 'https://www.youtube.com/watch?v=XHOmBV4js_E',
        'placeholder' => esc_html__( 'Enter your YouTube URL', 'myplugin' ),
        'label_block' => true,
        'condition'   => [ 'video_type' => 'youtube' ],
        'dynamic'     => [ 'active' => true ],
    ] );

    $this->add_control( 'vimeo_url', [
        'label'       => esc_html__( 'Link', 'myplugin' ),
        'type'        => \Elementor\Controls_Manager::TEXT,
        'default'     => 'https://vimeo.com/235215203',
        'placeholder' => esc_html__( 'Enter your Vimeo URL', 'myplugin' ),
        'label_block' => true,
        'condition'   => [ 'video_type' => 'vimeo' ],
        'dynamic'     => [ 'active' => true ],
    ] );

    $this->add_control( 'hosted_url', [
        'label'      => esc_html__( 'Choose File', 'myplugin' ),
        'type'       => \Elementor\Controls_Manager::MEDIA,
        'media_type' => 'video',
        'condition'  => [ 'video_type' => 'hosted' ],
        'dynamic'    => [ 'active' => true ],
    ] );

    $this->add_control( 'start', [
        'label'       => esc_html__( 'Start Time', 'myplugin' ),
        'type'        => \Elementor\Controls_Manager::NUMBER,
        'description' => esc_html__( 'Specify a start time (in seconds)', 'myplugin' ),
        'condition'   => [ 'video_type' => [ 'youtube', 'hosted' ] ],
    ] );

    $this->add_control( 'end', [
        'label'       => esc_html__( 'End Time', 'myplugin' ),
        'type'        => \Elementor\Controls_Manager::NUMBER,
        'description' => esc_html__( 'Specify an end time (in seconds)', 'myplugin' ),
        'condition'   => [ 'video_type' => [ 'youtube', 'hosted' ] ],
    ] );

    $this->add_control( 'video_options', [
        'label'     => esc_html__( 'Video Options', 'myplugin' ),
        'type'      => \Elementor\Controls_Manager::HEADING,
        'separator' => 'before',
    ] );

    $this->add_control( 'autoplay', [
        'label'   => esc_html__( 'Autoplay', 'myplugin' ),
        'type'    => \Elementor\Controls_Manager::SWITCHER,
        'default' => '',
    ] );

    $this->add_control( 'mute', [
        'label'   => esc_html__( 'Mute', 'myplugin' ),
        'type'    => \Elementor\Controls_Manager::SWITCHER,
        'default' => '',
    ] );

    $this->add_control( 'loop', [
        'label'   => esc_html__( 'Loop', 'myplugin' ),
        'type'    => \Elementor\Controls_Manager::SWITCHER,
        'default' => '',
    ] );

    $this->add_control( 'controls', [
        'label'   => esc_html__( 'Player Controls', 'myplugin' ),
        'type'    => \Elementor\Controls_Manager::SWITCHER,
        'default' => 'yes',
    ] );

    $this->add_control( 'show_image_overlay', [
        'label'     => esc_html__( 'Privacy Mode', 'myplugin' ),
        'type'      => \Elementor\Controls_Manager::SWITCHER,
        'default'   => '',
        'separator' => 'before',
    ] );

    $this->add_control( 'image_overlay', [
        'label'     => esc_html__( 'Choose Image', 'myplugin' ),
        'type'      => \Elementor\Controls_Manager::MEDIA,
        'default'   => [ 'url' => \Elementor\Utils::get_placeholder_image_src() ],
        'condition' => [ 'show_image_overlay' => 'yes' ],
        'dynamic'   => [ 'active' => true ],
    ] );

    $this->add_control( 'show_play_icon', [
        'label'     => esc_html__( 'Play Icon', 'myplugin' ),
        'type'      => \Elementor\Controls_Manager::SWITCHER,
        'default'   => 'yes',
        'condition' => [ 'show_image_overlay' => 'yes' ],
    ] );

    $this->add_control( 'lightbox', [
        'label'     => esc_html__( 'Lightbox', 'myplugin' ),
        'type'      => \Elementor\Controls_Manager::SWITCHER,
        'default'   => '',
        'condition' => [ 'show_image_overlay' => 'yes' ],
    ] );

    $this->end_controls_section();

    // ── STYLE ─────────────────────────────────────────────────
    $this->start_controls_section( 'section_video_style', [
        'label' => esc_html__( 'Video', 'myplugin' ),
        'tab'   => \Elementor\Controls_Manager::TAB_STYLE,
    ] );

    $this->add_control( 'aspect_ratio', [
        'label'        => esc_html__( 'Aspect Ratio', 'myplugin' ),
        'type'         => \Elementor\Controls_Manager::SELECT,
        'options'      => [
            '169' => '16:9',
            '219' => '21:9',
            '43'  => '4:3',
            '32'  => '3:2',
            '11'  => '1:1',
            '916' => '9:16',
        ],
        'default'      => '169',
        // ✅ prefix_class ONLY — writes e.g. 'elementor-aspect-ratio-169' on the wrapper.
        // Elementor's own CSS uses this class with padding-top trick or aspect-ratio property.
        // DO NOT add a 'selectors' entry here — '{{VALUE}}' would output '169' which is invalid CSS.
        'prefix_class' => 'elementor-aspect-ratio-',
    ] );

    $this->add_group_control(
        \Elementor\Group_Control_Css_Filter::get_type(),
        [
            'name'     => 'css_filters',
            'selector' => '{{WRAPPER}} .elementor-wrapper',
        ]
    );

    $this->add_control( 'play_icon_color', [
        'label'     => esc_html__( 'Play Icon Color', 'myplugin' ),
        'type'      => \Elementor\Controls_Manager::COLOR,
        'selectors' => [ '{{WRAPPER}} .elementor-custom-embed-play i' => 'color: {{VALUE}};' ],
        'condition' => [ 'show_image_overlay' => 'yes', 'show_play_icon' => 'yes' ],
    ] );

    $this->end_controls_section();
}
```

> **Remove checklist:**
> - `start` + `end` → remove if no time-scrubbing needed
> - `show_image_overlay` + `image_overlay` + `show_play_icon` + `lightbox` → remove if no overlay/privacy mode
> - `css_filters` → remove for simple embeds

**render() + content_template() skeleton:**

```php
public function has_widget_inner_wrapper(): bool {
    return false;
}

// ✅ Video embeds change based on URL — not user-specific, so static caching is safe.
protected function is_dynamic_content(): bool {
    return false;
}

protected function render(): void {
    $settings = $this->get_settings_for_display();

    $video_url = '';
    switch ( $settings['video_type'] ) {
        case 'youtube':
            $video_url = $settings['youtube_url'];
            break;
        case 'vimeo':
            $video_url = $settings['vimeo_url'];
            break;
        case 'hosted':
            $video_url = ! empty( $settings['hosted_url']['url'] ) ? $settings['hosted_url']['url'] : '';
            break;
    }

    if ( empty( $video_url ) ) {
        return;
    }

    // ✅ Use wp_oembed_get() for YouTube/Vimeo — handles oEmbed parameters correctly.
    // For self-hosted, output an HTML5 <video> tag directly.
    if ( 'hosted' === $settings['video_type'] ) {
        $has_autoplay = 'yes' === $settings['autoplay'];
        $has_muted    = 'yes' === $settings['mute'];

        // ✅ Browser Autoplay Policy: autoplay without muted is blocked in Chrome/Safari/Firefox.
        // Enforce muted when autoplay is requested to guarantee playback actually starts.
        // Do NOT silently strip autoplay — show it as requested but force muted alongside it.
        if ( $has_autoplay && ! $has_muted ) {
            $has_muted = true; // force muted so autoplay is honoured
        }

        // ✅ Build attributes array then implode — avoids raw HTML injection via esc_attr
        // on space-joined strings. Each boolean attribute is a plain ASCII word, safe to output.
        $attrs = array_filter( [
            'yes' === $settings['controls'] ? 'controls' : '',
            $has_autoplay ? 'autoplay' : '',
            $has_muted    ? 'muted'    : '',
            'yes' === $settings['loop'] ? 'loop' : '',
            'playsinline', // always — prevents iOS Safari fullscreen takeover
        ] );
        // phpcs:ignore WordPress.Security.EscapeOutput.OutputNotEscaped -- $attrs are all hardcoded ASCII attribute name strings; $video_url escaped via esc_url below
        echo sprintf(
            '<div class="myplugin-video elementor-wrapper"><video %s src="%s"></video></div>',
            implode( ' ', $attrs ),
            esc_url( $video_url )
        );
    } else {
        $oembed = wp_oembed_get( $video_url );
        if ( $oembed ) {
            // ✅ wp_kses_post() allows <iframe> (needed for YouTube/Vimeo oEmbed output).
            // Do NOT use esc_html() here — that would entity-encode the entire iframe tag.
            echo '<div class="myplugin-video elementor-wrapper">' . wp_kses_post( $oembed ) . '</div>'; // phpcs:ignore WordPress.Security.EscapeOutput.OutputNotEscaped -- kses-filtered above
        }
    }
}

protected function content_template(): void {
    ?>
    <#
    var url = '';
    if ( 'youtube' === settings.video_type ) { url = settings.youtube_url; }
    else if ( 'vimeo' === settings.video_type ) { url = settings.vimeo_url; }
    else if ( 'hosted' === settings.video_type && settings.hosted_url ) { url = settings.hosted_url.url; }
    if ( ! url ) { return; }
    #>
    <div class="myplugin-video elementor-wrapper">
        <div class="myplugin-video-placeholder">
            <i class="eicon-video-camera" aria-hidden="true"></i>
        </div>
    </div>
    <?php
}
```
