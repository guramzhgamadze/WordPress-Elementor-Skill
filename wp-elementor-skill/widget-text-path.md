# Widget Boilerplate — Text Path

> **When to use this file:** Load whenever building a widget that renders text following a
> curved or custom SVG path (wave, arc, circle, spiral, or a user-supplied SVG).
> Mirrors Elementor's native **Text Path** widget (free, since ~3.3). Built on V3 `Widget_Base`
> (Elementor 3.x–4.x; current through 4.2).
>
> **How it works:** an inline `<svg>` contains a `<path>` plus a `<text><textPath href="#id">`
> that flows the text along that path. The path needs a **unique id per widget instance**
> (derived from `get_id()`) so multiple Text Path widgets on one page don't collide.

---

## ⚠️ Two ways to supply the path — security note

| Source | Safe? | Notes |
|---|---|---|
| **Preset shapes** (hardcoded `d` attributes below) | ✅ Yes | Recommended default — the geometry is developer-defined, never user input. |
| **Custom SVG upload** (`MEDIA`, `svg`) | ⚠️ Conditional | SVG is an XSS vector. You MUST sanitize uploaded SVG before inlining it. Elementor ships an SVG sanitizer (`\Elementor\Core\Files\File_Types\Svg`) and only allows SVG uploads when the **"SVG Uploads"** option is enabled. Never `echo` raw uploaded SVG markup. The simplest safe approach is to read only the first `<path d="...">` value and re-emit it through `esc_attr()`. |

This boilerplate uses **presets** as the safe default and shows the custom path as an opt-in.

---

## register_controls() skeleton

```php
protected function register_controls(): void {

    // ── CONTENT ───────────────────────────────────────────────
    $this->start_controls_section( 'section_text_path', [
        'label' => esc_html__( 'Text Path', 'myplugin' ),
        'tab'   => \Elementor\Controls_Manager::TAB_CONTENT,
    ] );

    $this->add_control( 'text', [
        'label'       => esc_html__( 'Text', 'myplugin' ),
        'type'        => \Elementor\Controls_Manager::TEXTAREA,
        'default'     => esc_html__( 'Add Your Curved Text Here', 'myplugin' ),
        'placeholder' => esc_html__( 'Enter your text', 'myplugin' ),
        'dynamic'     => [ 'active' => true ],
    ] );

    $this->add_control( 'path_type', [
        'label'   => esc_html__( 'Path', 'myplugin' ),
        'type'    => \Elementor\Controls_Manager::SELECT,
        'default' => 'wave',
        'options' => [
            'wave'   => esc_html__( 'Wave',   'myplugin' ),
            'arc'    => esc_html__( 'Arc',    'myplugin' ),
            'line'   => esc_html__( 'Line',   'myplugin' ),
            'circle' => esc_html__( 'Circle', 'myplugin' ),
            'custom' => esc_html__( 'Custom SVG', 'myplugin' ),
        ],
    ] );

    // Custom SVG — only when path_type = custom (see security note above).
    $this->add_control( 'custom_svg', [
        'label'       => esc_html__( 'Custom SVG', 'myplugin' ),
        'type'        => \Elementor\Controls_Manager::MEDIA,
        'media_types' => [ 'svg' ],
        'condition'   => [ 'path_type' => 'custom' ],
        'description' => esc_html__( 'Requires SVG uploads enabled and a sanitized SVG. The first <path> is used.', 'myplugin' ),
    ] );

    $this->add_control( 'link', [
        'label'   => esc_html__( 'Link', 'myplugin' ),
        'type'    => \Elementor\Controls_Manager::URL,
        'dynamic' => [ 'active' => true ],
    ] );

    // ✅ startOffset along the path (0–100%). Set as an SVG attribute in render(), NOT CSS.
    $this->add_control( 'start_point', [
        'label'      => esc_html__( 'Starting Point', 'myplugin' ),
        'type'       => \Elementor\Controls_Manager::SLIDER,
        'default'    => [ 'unit' => '%', 'size' => 0 ],
        'size_units' => [ '%' ],
        'range'      => [ '%' => [ 'min' => 0, 'max' => 100 ] ],
    ] );

    $this->add_control( 'show_path', [
        'label'   => esc_html__( 'Show Path', 'myplugin' ),
        'type'    => \Elementor\Controls_Manager::SWITCHER,
        'default' => '',
    ] );

    $this->end_controls_section();

    // ── STYLE: Text ───────────────────────────────────────────
    $this->start_controls_section( 'section_text_style', [
        'label' => esc_html__( 'Text', 'myplugin' ),
        'tab'   => \Elementor\Controls_Manager::TAB_STYLE,
    ] );

    // ✅ Typography group control — font-family/size/weight/spacing all apply to SVG <text>.
    $this->add_group_control(
        \Elementor\Group_Control_Typography::get_type(),
        [
            'name'     => 'text_typography',
            'selector' => '{{WRAPPER}} .myplugin-text-path text',
        ]
    );

    // ✅ SVG text color is the `fill` property — NOT `color`.
    $this->add_control( 'text_color', [
        'label'     => esc_html__( 'Text Color', 'myplugin' ),
        'type'      => \Elementor\Controls_Manager::COLOR,
        'default'   => '',
        'selectors' => [ '{{WRAPPER}} .myplugin-text-path text' => 'fill: {{VALUE}};' ],
    ] );

    $this->add_control( 'text_hover_color', [
        'label'     => esc_html__( 'Hover Text Color', 'myplugin' ),
        'type'      => \Elementor\Controls_Manager::COLOR,
        'selectors' => [ '{{WRAPPER}} .myplugin-text-path:hover text' => 'fill: {{VALUE}};' ],
    ] );

    $this->end_controls_section();

    // ── STYLE: Path ───────────────────────────────────────────
    $this->start_controls_section( 'section_path_style', [
        'label'     => esc_html__( 'Path', 'myplugin' ),
        'tab'       => \Elementor\Controls_Manager::TAB_STYLE,
        'condition' => [ 'show_path' => 'yes' ],
    ] );

    $this->add_control( 'path_color', [
        'label'     => esc_html__( 'Path Color', 'myplugin' ),
        'type'      => \Elementor\Controls_Manager::COLOR,
        'default'   => '#d0d0d0',
        'selectors' => [ '{{WRAPPER}} .myplugin-text-path path' => 'stroke: {{VALUE}};' ],
    ] );

    $this->add_control( 'path_width', [
        'label'     => esc_html__( 'Path Width', 'myplugin' ),
        'type'      => \Elementor\Controls_Manager::SLIDER,
        'range'     => [ 'px' => [ 'min' => 0, 'max' => 20 ] ],
        'default'   => [ 'size' => 1, 'unit' => 'px' ],
        'selectors' => [ '{{WRAPPER}} .myplugin-text-path path' => 'stroke-width: {{SIZE}}{{UNIT}};' ],
    ] );

    $this->end_controls_section();
}
```

---

## Preset path helper

```php
/**
 * Developer-defined SVG path presets. Each preset is a [ 'd', 'viewBox' ] pair.
 * Because these `d` strings are hardcoded (never user input), they are safe to output.
 *
 * @return array<string, array{d:string, viewBox:string}>
 */
private function get_path_presets(): array {
    return [
        'wave'   => [ 'd' => 'M0,50 Q100,0 200,50 T400,50',                       'viewBox' => '0 0 400 100' ],
        'arc'    => [ 'd' => 'M10,90 A190,190 0 0 1 390,90',                      'viewBox' => '0 0 400 110' ],
        'line'   => [ 'd' => 'M0,50 L400,50',                                     'viewBox' => '0 0 400 100' ],
        'circle' => [ 'd' => 'M200,40 a160,160 0 1,1 -0.1,0',                     'viewBox' => '0 0 400 400' ],
    ];
}
```

---

## has_widget_inner_wrapper() + is_dynamic_content() + render() skeleton

```php
public function has_widget_inner_wrapper(): bool {
    return false;
}

// ✅ Static unless the text comes from a per-request Dynamic Tag — false is the safe default.
protected function is_dynamic_content(): bool {
    return false;
}

protected function render(): void {
    $settings = $this->get_settings_for_display();

    $text = $settings['text'] ?? '';
    if ( '' === $text ) {
        return;
    }

    // ✅ Unique path id per instance — prevents <textPath href="#id"> collisions when
    // multiple Text Path widgets share a page. get_id() is the Elementor element id.
    $path_id = 'myplugin-text-path-' . $this->get_id();

    $presets = $this->get_path_presets();
    $type    = $settings['path_type'] ?? 'wave';

    if ( 'custom' === $type && ! empty( $settings['custom_svg']['url'] ) ) {
        // ⚠️ Custom SVG path. Resolve the uploaded file, sanitize, and extract ONLY the
        // first `d` attribute — never inline the whole uploaded SVG (XSS risk).
        $d       = $this->get_custom_path_d( (int) ( $settings['custom_svg']['id'] ?? 0 ) );
        $view_box = '0 0 400 100'; // fall back; adjust to your SVGs or read it the same way
        if ( '' === $d ) {
            return;
        }
    } else {
        $preset   = $presets[ $type ] ?? $presets['wave'];
        $d        = $preset['d'];
        $view_box = $preset['viewBox'];
    }

    // startOffset is an SVG attribute (percentage), not CSS.
    $start_offset = (int) ( $settings['start_point']['size'] ?? 0 );

    // Path stroke visibility: 'none' hides it, controls re-enable it via the Path style section.
    $path_fill   = 'none';
    $path_stroke = 'yes' === ( $settings['show_path'] ?? '' ) ? 'currentColor' : 'transparent';

    $this->add_render_attribute( 'wrapper', 'class', 'myplugin-text-path' );

    $open_link  = '';
    $close_link = '';
    if ( ! empty( $settings['link']['url'] ) ) {
        $this->add_link_attributes( 'link', $settings['link'] );
        $open_link  = '<a ' . $this->get_render_attribute_string( 'link' ) . '>';
        $close_link = '</a>';
    }
    ?>
    <div <?php $this->print_render_attribute_string( 'wrapper' ); ?>>
        <?php echo $open_link; // phpcs:ignore WordPress.Security.EscapeOutput.OutputNotEscaped -- built via add_link_attributes(), pre-sanitized ?>
        <svg viewBox="<?php echo esc_attr( $view_box ); ?>" preserveAspectRatio="xMidYMid meet" xmlns="http://www.w3.org/2000/svg">
            <path id="<?php echo esc_attr( $path_id ); ?>"
                  d="<?php echo esc_attr( $d ); ?>"
                  fill="<?php echo esc_attr( $path_fill ); ?>"
                  stroke="<?php echo esc_attr( $path_stroke ); ?>"></path>
            <text>
                <textPath href="#<?php echo esc_attr( $path_id ); ?>"
                          xlink:href="#<?php echo esc_attr( $path_id ); ?>"
                          startOffset="<?php echo esc_attr( $start_offset ); ?>%">
                    <?php echo esc_html( $text ); ?>
                </textPath>
            </text>
        </svg>
        <?php echo $close_link; // phpcs:ignore WordPress.Security.EscapeOutput.OutputNotEscaped -- closing tag literal ?>
    </div>
    <?php
}

/**
 * Read the first `d` attribute from a sanitized uploaded SVG.
 * Returns '' if the attachment is missing, not an SVG, or contains no <path>.
 */
private function get_custom_path_d( int $attachment_id ): string {
    if ( $attachment_id <= 0 ) {
        return '';
    }
    $file = get_attached_file( $attachment_id );
    if ( ! $file || ! is_readable( $file ) || 'image/svg+xml' !== get_post_mime_type( $attachment_id ) ) {
        return '';
    }
    $svg = file_get_contents( $file ); // phpcs:ignore WordPress.WP.AlternativeFunctions.file_get_contents_file_get_contents
    if ( ! $svg ) {
        return '';
    }
    // ✅ Extract ONLY the first path `d` value — never inline the full SVG.
    if ( preg_match( '/<path[^>]*\sd="([^"]+)"/i', $svg, $m ) ) {
        return $m[1]; // esc_attr() is applied at output in render()
    }
    return '';
}
```

---

## content_template() skeleton

```php
protected function content_template(): void {
    ?>
    <#
    if ( ! settings.text ) { return; }
    // Preset map mirrored from get_path_presets() — keep both in sync.
    var presets = {
        wave:   { d: 'M0,50 Q100,0 200,50 T400,50', viewBox: '0 0 400 100' },
        arc:    { d: 'M10,90 A190,190 0 0 1 390,90', viewBox: '0 0 400 110' },
        line:   { d: 'M0,50 L400,50',                viewBox: '0 0 400 100' },
        circle: { d: 'M200,40 a160,160 0 1,1 -0.1,0', viewBox: '0 0 400 400' }
    };
    var preset = presets[ settings.path_type ] || presets.wave;
    var pathId = 'myplugin-text-path-preview';
    var offset = settings.start_point && settings.start_point.size ? settings.start_point.size : 0;
    var showPath = 'yes' === settings.show_path;
    #>
    <div class="myplugin-text-path">
        <svg viewBox="{{ preset.viewBox }}" preserveAspectRatio="xMidYMid meet" xmlns="http://www.w3.org/2000/svg">
            <path id="{{ pathId }}" d="{{ preset.d }}" fill="none" stroke="{{ showPath ? 'currentColor' : 'transparent' }}"></path>
            <text><textPath href="#{{ pathId }}" startOffset="{{ offset }}%">{{ settings.text }}</textPath></text>
        </svg>
    </div>
    <?php
}
```

> **Note on the editor preview:** `content_template()` cannot read a custom uploaded SVG's path
> (that resolution is PHP-side), so the preview always uses a preset shape — the custom path
> renders on the frontend only. This matches how Elementor's own SVG-dependent widgets behave.

---

> **Remove checklist:**
> - `custom_svg` + `get_custom_path_d()` → remove if only preset paths are needed (drops the SVG security surface entirely).
> - `link` → remove if the curved text is never a link.
> - Path style section + `show_path` → remove if the path line is always hidden.
> - `text_hover_color` → remove if there is no hover state.
