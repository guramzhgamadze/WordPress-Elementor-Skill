# Widget Boilerplate — Audio

> **When to use this file:** Load whenever building a widget that embeds SoundCloud or self-hosted audio.
> Verified against `elementor/includes/widgets/audio.php` (Elementor 3.35+).

---

```php
protected function register_controls(): void {

    $this->start_controls_section( 'section_audio', [
        'label' => esc_html__( 'Audio', 'myplugin' ),
        'tab'   => \Elementor\Controls_Manager::TAB_CONTENT,
    ] );

    $this->add_control( 'link', [
        'label'       => esc_html__( 'Link', 'myplugin' ),
        'type'        => \Elementor\Controls_Manager::TEXT,
        'placeholder' => esc_html__( 'Enter your SoundCloud URL', 'myplugin' ),
        'default'     => 'https://soundcloud.com/shawn-wasabi/maple-syrup',
        'label_block' => true,
        'dynamic'     => [ 'active' => true ],
    ] );

    $this->add_control( 'visual', [
        'label'   => esc_html__( 'Visual Player', 'myplugin' ),
        'type'    => \Elementor\Controls_Manager::SWITCHER,
        'default' => 'yes',
    ] );

    $this->add_control( 'sc_auto_play', [
        'label'   => esc_html__( 'Autoplay', 'myplugin' ),
        'type'    => \Elementor\Controls_Manager::SWITCHER,
        'default' => '',
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

    if ( empty( $settings['link'] ) ) {
        return;
    }

    // ✅ Use wp_oembed_get() to convert the SoundCloud URL into an embed iframe.
    // Falls back gracefully — returns false if oEmbed fails (e.g. private track).
    // ⚠️ Pass the RAW URL — do NOT wrap in esc_url() here. esc_url() HTML-encodes
    // ampersands (&→&amp;) which corrupts the URL before wp_oembed_get() can parse it.
    // Sanitize with sanitize_url() instead, which strips unsafe characters without encoding.
    $oembed_html = wp_oembed_get( sanitize_url( $settings['link'] ) );

    if ( ! $oembed_html ) {
        if ( \Elementor\Plugin::$instance->editor && \Elementor\Plugin::$instance->editor->is_edit_mode() ) {
            echo '<p>' . esc_html__( 'Could not load audio. Check the URL.', 'myplugin' ) . '</p>';
        }
        return;
    }

    // ✅ DO NOT use wp_kses_post() on oEmbed HTML.
    // wp_kses_post() uses wp_kses_allowed_html('post') which does NOT include <iframe>.
    // SoundCloud oEmbed returns an <iframe> — wp_kses_post() would strip it entirely,
    // producing empty output with no error. wp_oembed_get() returns WordPress-generated
    // HTML that has already been processed through WordPress's own oEmbed stack (trusted).
    // This matches Elementor's native Audio widget (elementor/includes/widgets/audio.php).
    // Source: developer.wordpress.org/reference/functions/wp_kses_allowed_html/ ('post' context)
    echo '<div class="myplugin-audio">' . $oembed_html . '</div>'; // phpcs:ignore WordPress.Security.EscapeOutput.OutputNotEscaped -- trusted WordPress-generated oEmbed; wp_kses_post strips <iframe>
}

protected function content_template(): void {
    ?>
    <#
    if ( ! settings.link ) { return; }
    #>
    <div class="myplugin-audio myplugin-audio--placeholder">
        <p><?php echo esc_html__( 'Audio preview available on the frontend.', 'myplugin' ); ?></p>
    </div>
    <?php
}
```
