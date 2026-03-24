# Widget Boilerplate — Toggle

> **When to use this file:** Load whenever building a collapsible content widget where multiple panels can be open simultaneously.
> Verified against `elementor/includes/widgets/toggle.php` (Elementor 3.35+).
> ⚠️ Toggle is structurally identical to Accordion — the only differences are noted below.

---

```php
protected function register_controls(): void {

    // ── CONTENT ───────────────────────────────────────────────
    $this->start_controls_section( 'section_toggle', [
        'label' => esc_html__( 'Toggle', 'myplugin' ),
        'tab'   => \Elementor\Controls_Manager::TAB_CONTENT,
    ] );

    $repeater = new \Elementor\Repeater();

    $repeater->add_control( 'tab_title', [
        'label'       => esc_html__( 'Title', 'myplugin' ),
        'type'        => \Elementor\Controls_Manager::TEXT,
        'default'     => esc_html__( 'Toggle Title', 'myplugin' ),
        'label_block' => true,
        'dynamic'     => [ 'active' => true ],
    ] );

    $repeater->add_control( 'tab_content', [
        'label'      => esc_html__( 'Content', 'myplugin' ),
        'type'       => \Elementor\Controls_Manager::WYSIWYG,
        'default'    => esc_html__( 'Toggle Content', 'myplugin' ),
        'show_label' => false,
        'dynamic'    => [ 'active' => true ],
    ] );

    $this->add_control( 'tabs', [
        'label'       => esc_html__( 'Toggle Items', 'myplugin' ),
        'type'        => \Elementor\Controls_Manager::REPEATER,
        'fields'      => $repeater->get_controls(),
        'default'     => [
            [ 'tab_title' => esc_html__( 'Toggle #1', 'myplugin' ), 'tab_content' => esc_html__( 'Content here.', 'myplugin' ) ],
            [ 'tab_title' => esc_html__( 'Toggle #2', 'myplugin' ), 'tab_content' => esc_html__( 'Content here.', 'myplugin' ) ],
            [ 'tab_title' => esc_html__( 'Toggle #3', 'myplugin' ), 'tab_content' => esc_html__( 'Content here.', 'myplugin' ) ],
        ],
        'title_field' => '{{{ tab_title }}}',
    ] );

    $this->add_control( 'selected_icon', [
        'label'            => esc_html__( 'Icon', 'myplugin' ),
        'type'             => \Elementor\Controls_Manager::ICONS,
        'separator'        => 'before',
        'fa4compatibility' => 'icon',
        'default'          => [ 'value' => 'fas fa-plus',  'library' => 'fa-solid' ],
        'label_block'      => false,
        'skin'             => 'inline',
    ] );

    $this->add_control( 'selected_active_icon', [
        'label'            => esc_html__( 'Active Icon', 'myplugin' ),
        'type'             => \Elementor\Controls_Manager::ICONS,
        'fa4compatibility' => 'icon_active',
        'default'          => [ 'value' => 'fas fa-minus', 'library' => 'fa-solid' ],
        'label_block'      => false,
        'skin'             => 'inline',
        'condition'        => [ 'selected_icon[value]!' => '' ],
    ] );

    $this->add_control( 'title_html_tag', [
        'label'     => esc_html__( 'Title HTML Tag', 'myplugin' ),
        'type'      => \Elementor\Controls_Manager::SELECT,
        'options'   => [ 'p' => 'p', 'div' => 'div', 'span' => 'span', 'h1' => 'H1', 'h2' => 'H2', 'h3' => 'H3', 'h4' => 'H4', 'h5' => 'H5', 'h6' => 'H6' ],
        'default'   => 'p',
        'separator' => 'before',
    ] );

    $this->end_controls_section();

    // ── STYLE ─────────────────────────────────────────────────
    $this->start_controls_section( 'section_toggle_style', [
        'label' => esc_html__( 'Toggle', 'myplugin' ),
        'tab'   => \Elementor\Controls_Manager::TAB_STYLE,
    ] );

    $this->add_control( 'border_width', [
        'label'     => esc_html__( 'Border Width', 'myplugin' ),
        'type'      => \Elementor\Controls_Manager::SLIDER,
        'range'     => [ 'px' => [ 'min' => 0, 'max' => 10 ] ],
        'default'   => [ 'size' => 1, 'unit' => 'px' ],
        'selectors' => [
            '{{WRAPPER}} .myplugin-toggle-item, {{WRAPPER}} .myplugin-toggle-title, {{WRAPPER}} .myplugin-toggle-content' => 'border-width: {{SIZE}}{{UNIT}};',
        ],
    ] );

    $this->add_control( 'border_color', [
        'label'     => esc_html__( 'Border Color', 'myplugin' ),
        'type'      => \Elementor\Controls_Manager::COLOR,
        'selectors' => [
            '{{WRAPPER}} .myplugin-toggle-item, {{WRAPPER}} .myplugin-toggle-title, {{WRAPPER}} .myplugin-toggle-content' => 'border-color: {{VALUE}};',
        ],
    ] );

    $this->end_controls_section();

    $this->start_controls_section( 'section_title_style', [
        'label' => esc_html__( 'Title', 'myplugin' ),
        'tab'   => \Elementor\Controls_Manager::TAB_STYLE,
    ] );

    $this->add_control( 'title_background', [
        'label'     => esc_html__( 'Background', 'myplugin' ),
        'type'      => \Elementor\Controls_Manager::COLOR,
        'selectors' => [ '{{WRAPPER}} .myplugin-toggle-title' => 'background-color: {{VALUE}};' ],
    ] );

    $this->add_control( 'title_color', [
        'label'     => esc_html__( 'Color', 'myplugin' ),
        'type'      => \Elementor\Controls_Manager::COLOR,
        'global'    => [ 'default' => \Elementor\Core\Kits\Documents\Tabs\Global_Colors::COLOR_PRIMARY ],
        'selectors' => [ '{{WRAPPER}} .myplugin-toggle-title' => 'color: {{VALUE}};' ],
    ] );

    $this->add_control( 'title_active_color', [
        'label'     => esc_html__( 'Active Color', 'myplugin' ),
        'type'      => \Elementor\Controls_Manager::COLOR,
        'global'    => [ 'default' => \Elementor\Core\Kits\Documents\Tabs\Global_Colors::COLOR_ACCENT ],
        'selectors' => [ '{{WRAPPER}} .myplugin-toggle-title.myplugin-active' => 'color: {{VALUE}};' ],
    ] );

    $this->add_group_control(
        \Elementor\Group_Control_Typography::get_type(),
        [
            'name'     => 'title_typography',
            'selector' => '{{WRAPPER}} .myplugin-toggle-title',
            'global'   => [ 'default' => \Elementor\Core\Kits\Documents\Tabs\Global_Typography::TYPOGRAPHY_PRIMARY ],
        ]
    );

    $this->end_controls_section();

    $this->start_controls_section( 'section_content_style', [
        'label' => esc_html__( 'Content', 'myplugin' ),
        'tab'   => \Elementor\Controls_Manager::TAB_STYLE,
    ] );

    $this->add_control( 'content_background_color', [
        'label'     => esc_html__( 'Background', 'myplugin' ),
        'type'      => \Elementor\Controls_Manager::COLOR,
        'selectors' => [ '{{WRAPPER}} .myplugin-toggle-content' => 'background-color: {{VALUE}};' ],
    ] );

    $this->add_control( 'content_color', [
        'label'     => esc_html__( 'Color', 'myplugin' ),
        'type'      => \Elementor\Controls_Manager::COLOR,
        'global'    => [ 'default' => \Elementor\Core\Kits\Documents\Tabs\Global_Colors::COLOR_TEXT ],
        'selectors' => [ '{{WRAPPER}} .myplugin-toggle-content' => 'color: {{VALUE}};' ],
    ] );

    $this->add_group_control(
        \Elementor\Group_Control_Typography::get_type(),
        [
            'name'     => 'content_typography',
            'selector' => '{{WRAPPER}} .myplugin-toggle-content',
            'global'   => [ 'default' => \Elementor\Core\Kits\Documents\Tabs\Global_Typography::TYPOGRAPHY_TEXT ],
        ]
    );

    $this->end_controls_section();
}
```

> **Key difference from Accordion:**
> Toggle allows multiple panels open at the same time.
> In JS, do NOT close other panels when one opens — each item manages its own open/closed state independently.

> ⚠️ **Keyboard interaction requirement:** Elements with `role="button"` on non-native button
> tags require explicit `keydown` handlers for `Enter` and `Space`. See `widget-accordion.md`
> for the full keyboard handler pattern — it applies identically to toggle.

> **Remove checklist:**
> - `selected_icon` + `selected_active_icon` → remove if no expand/collapse icons needed
> - `title_html_tag` → remove if title tag is always fixed

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
    $tabs     = $settings['tabs'];

    if ( empty( $tabs ) ) {
        return;
    }

    // ✅ get_id() replaces deprecated get_id_int() (deprecated since Elementor 3.1.0)
    // Source: developers.elementor.com/v3-5-planned-deprecations/
    $id_int = substr( $this->get_id(), 0, 3 );
    ?>
    <div class="myplugin-toggle">
        <?php foreach ( $tabs as $index => $tab ) :
            $tab_count = $index + 1;
            $tab_id    = 'myplugin-toggle-' . $id_int . $tab_count;
            $tag       = \Elementor\Utils::validate_html_tag( $settings['title_html_tag'] );
        ?>
        <?php $content_id = 'myplugin-toggle-content-' . $id_int . $tab_count; ?>
        <div class="myplugin-toggle-item elementor-repeater-item-<?php echo esc_attr( $tab['_id'] ); ?>">
            <<?php echo esc_attr( $tag ); ?>
                id="<?php echo esc_attr( $tab_id ); ?>"
                class="myplugin-toggle-title"
                aria-expanded="false"
                aria-controls="<?php echo esc_attr( $content_id ); ?>"
                role="button"
                tabindex="0">
                <?php if ( ! empty( $tab['selected_icon']['value'] ) ) : ?>
                    <span class="myplugin-toggle-icon">
                        <span class="myplugin-toggle-icon-opened">
                            <?php \Elementor\Icons_Manager::render_icon( $settings['selected_active_icon'], [ 'aria-hidden' => 'true' ] ); ?>
                        </span>
                        <span class="myplugin-toggle-icon-closed">
                            <?php \Elementor\Icons_Manager::render_icon( $settings['selected_icon'], [ 'aria-hidden' => 'true' ] ); ?>
                        </span>
                    </span>
                <?php endif; ?>
                <span class="myplugin-toggle-title-text"><?php echo esc_html( $tab['tab_title'] ); ?></span>
            </<?php echo esc_attr( $tag ); ?>>
            <div class="myplugin-toggle-content"
                 id="<?php echo esc_attr( $content_id ); ?>"
                 role="region"
                 aria-labelledby="<?php echo esc_attr( $tab_id ); ?>"
                 style="display:none;">
                <?php echo wp_kses_post( $tab['tab_content'] ); ?>
            </div>
        </div>
        <?php endforeach; ?>
    </div>
    <?php
}

protected function content_template(): void {
    ?>
    <#
    if ( ! settings.tabs || ! settings.tabs.length ) { return; }
    var tag = elementor.helpers.validateHTMLTag( settings.title_html_tag );
    #>
    <div class="myplugin-toggle">
        <# _.each( settings.tabs, function( tab, index ) {
            var iconHTML = tab.selected_icon && tab.selected_icon.value
                ? elementor.helpers.renderIcon( view, tab.selected_icon, { 'aria-hidden': true }, 'i', 'object' )
                : null;
        #>
        <# var contentId = 'myplugin-toggle-content-' + index; #>
        <div class="myplugin-toggle-item">
            <{{{ tag }}} class="myplugin-toggle-title" aria-expanded="false" aria-controls="{{ contentId }}" role="button" tabindex="0">
                <# if ( iconHTML ) { #>
                    <span class="myplugin-toggle-icon">
                        <span class="myplugin-toggle-icon-opened">{{{ elementor.helpers.renderIcon( view, settings.selected_active_icon, { 'aria-hidden': true }, 'i', 'object' ).value }}}</span>
                        <span class="myplugin-toggle-icon-closed">{{{ iconHTML.value }}}</span>
                    </span>
                <# } #>
                <span class="myplugin-toggle-title-text">{{{ tab.tab_title }}}</span>
            </{{{ tag }}}>
            <div class="myplugin-toggle-content" style="display:none;">
                {{{ tab.tab_content }}}
            </div>
        </div>
        <# } ); #>
    </div>
    <?php
}
```
