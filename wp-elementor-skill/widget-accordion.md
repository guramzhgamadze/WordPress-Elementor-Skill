# Widget Boilerplate — Accordion

> **When to use this file:** Load whenever building a collapsible content widget where one panel opens at a time.
> Verified against `elementor/includes/widgets/accordion.php` (Elementor 3.35+).
> Toggle widget is identical except `multiple_active` defaults to 'yes' — note at bottom.

---

```php
protected function register_controls(): void {

    // ── CONTENT ───────────────────────────────────────────────
    $this->start_controls_section( 'section_title', [
        'label' => esc_html__( 'Accordion', 'myplugin' ),
        'tab'   => \Elementor\Controls_Manager::TAB_CONTENT,
    ] );

    $repeater = new \Elementor\Repeater();

    $repeater->add_control( 'tab_title', [
        'label'       => esc_html__( 'Title', 'myplugin' ),
        'type'        => \Elementor\Controls_Manager::TEXT,
        'default'     => esc_html__( 'Accordion Title', 'myplugin' ),
        'label_block' => true,
        'dynamic'     => [ 'active' => true ],
    ] );

    $repeater->add_control( 'tab_content', [
        'label'      => esc_html__( 'Content', 'myplugin' ),
        'type'       => \Elementor\Controls_Manager::WYSIWYG,
        'default'    => esc_html__( 'Accordion Content', 'myplugin' ),
        'show_label' => false,
        'dynamic'    => [ 'active' => true ],
    ] );

    $this->add_control( 'tabs', [
        'label'       => esc_html__( 'Items', 'myplugin' ),
        'type'        => \Elementor\Controls_Manager::REPEATER,
        'fields'      => $repeater->get_controls(),
        'default'     => [
            [ 'tab_title' => esc_html__( 'Accordion #1', 'myplugin' ), 'tab_content' => esc_html__( 'Content here.', 'myplugin' ) ],
            [ 'tab_title' => esc_html__( 'Accordion #2', 'myplugin' ), 'tab_content' => esc_html__( 'Content here.', 'myplugin' ) ],
            [ 'tab_title' => esc_html__( 'Accordion #3', 'myplugin' ), 'tab_content' => esc_html__( 'Content here.', 'myplugin' ) ],
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
    $this->start_controls_section( 'section_title_style', [
        'label' => esc_html__( 'Accordion', 'myplugin' ),
        'tab'   => \Elementor\Controls_Manager::TAB_STYLE,
    ] );

    $this->add_control( 'border_width', [
        'label'     => esc_html__( 'Border Width', 'myplugin' ),
        'type'      => \Elementor\Controls_Manager::SLIDER,
        'range'     => [ 'px' => [ 'min' => 0, 'max' => 10 ] ],
        'default'   => [ 'size' => 1, 'unit' => 'px' ],
        'selectors' => [
            '{{WRAPPER}} .myplugin-accordion-item, {{WRAPPER}} .myplugin-tab-title, {{WRAPPER}} .myplugin-tab-content' => 'border-width: {{SIZE}}{{UNIT}};',
        ],
    ] );

    $this->add_control( 'border_color', [
        'label'     => esc_html__( 'Border Color', 'myplugin' ),
        'type'      => \Elementor\Controls_Manager::COLOR,
        'selectors' => [
            '{{WRAPPER}} .myplugin-accordion-item, {{WRAPPER}} .myplugin-tab-title, {{WRAPPER}} .myplugin-tab-content' => 'border-color: {{VALUE}};',
        ],
    ] );

    $this->end_controls_section();

    $this->start_controls_section( 'section_toggle_style_title', [
        'label' => esc_html__( 'Title', 'myplugin' ),
        'tab'   => \Elementor\Controls_Manager::TAB_STYLE,
    ] );

    $this->add_control( 'title_background', [
        'label'     => esc_html__( 'Background', 'myplugin' ),
        'type'      => \Elementor\Controls_Manager::COLOR,
        'selectors' => [ '{{WRAPPER}} .myplugin-tab-title' => 'background-color: {{VALUE}};' ],
    ] );

    $this->add_control( 'title_color', [
        'label'     => esc_html__( 'Color', 'myplugin' ),
        'type'      => \Elementor\Controls_Manager::COLOR,
        'selectors' => [ '{{WRAPPER}} .myplugin-tab-title' => 'color: {{VALUE}};' ],
        'global'    => [ 'default' => \Elementor\Core\Kits\Documents\Tabs\Global_Colors::COLOR_PRIMARY ],
    ] );

    $this->add_control( 'title_active_color', [
        'label'     => esc_html__( 'Active Color', 'myplugin' ),
        'type'      => \Elementor\Controls_Manager::COLOR,
        'selectors' => [ '{{WRAPPER}} .myplugin-tab-title.myplugin-active' => 'color: {{VALUE}};' ],
        'global'    => [ 'default' => \Elementor\Core\Kits\Documents\Tabs\Global_Colors::COLOR_ACCENT ],
    ] );

    $this->add_group_control(
        \Elementor\Group_Control_Typography::get_type(),
        [
            'name'     => 'title_typography',
            'selector' => '{{WRAPPER}} .myplugin-tab-title',
            'global'   => [ 'default' => \Elementor\Core\Kits\Documents\Tabs\Global_Typography::TYPOGRAPHY_PRIMARY ],
        ]
    );

    $this->end_controls_section();

    $this->start_controls_section( 'section_toggle_style_content', [
        'label' => esc_html__( 'Content', 'myplugin' ),
        'tab'   => \Elementor\Controls_Manager::TAB_STYLE,
    ] );

    $this->add_control( 'content_background_color', [
        'label'     => esc_html__( 'Background', 'myplugin' ),
        'type'      => \Elementor\Controls_Manager::COLOR,
        'selectors' => [ '{{WRAPPER}} .myplugin-tab-content' => 'background-color: {{VALUE}};' ],
    ] );

    $this->add_control( 'content_color', [
        'label'     => esc_html__( 'Color', 'myplugin' ),
        'type'      => \Elementor\Controls_Manager::COLOR,
        'selectors' => [ '{{WRAPPER}} .myplugin-tab-content' => 'color: {{VALUE}};' ],
        'global'    => [ 'default' => \Elementor\Core\Kits\Documents\Tabs\Global_Colors::COLOR_TEXT ],
    ] );

    $this->add_group_control(
        \Elementor\Group_Control_Typography::get_type(),
        [
            'name'     => 'content_typography',
            'selector' => '{{WRAPPER}} .myplugin-tab-content',
            'global'   => [ 'default' => \Elementor\Core\Kits\Documents\Tabs\Global_Typography::TYPOGRAPHY_TEXT ],
        ]
    );

    $this->end_controls_section();
}
```

> ⚠️ **Keyboard interaction requirement (ARIA APG §3.1):** Elements with `role="button"` that
> are NOT native `<button>` elements do NOT receive keyboard events for free. Your widget JS
> **MUST** handle both `click` and `keydown` (Enter + Space) to activate panels, and `keydown`
> (Up/Down arrows) to move focus between panel titles. Without this, keyboard-only users cannot
> operate the accordion.
> ```js
> // Minimal keyboard handler — add inside your elementor/frontend/init addAction callback
> scope.querySelectorAll( '.myplugin-tab-title' ).forEach( el => {
>   el.addEventListener( 'keydown', e => {
>     if ( e.key === 'Enter' || e.key === ' ' ) { e.preventDefault(); el.click(); }
>   } );
> } );
> ```
> Source: w3.org/WAI/ARIA/apg/patterns/accordion/

> **Toggle widget note:** Toggle is identical to Accordion with two differences:
> 1. `get_name()` returns `'toggle'`, `get_title()` returns `'Toggle'`
> 2. All panels can be open simultaneously (no mutual exclusion in JS)

> **Remove checklist:**
> - `selected_icon` + `selected_active_icon` → remove if no expand/collapse icons needed
> - `title_html_tag` → remove if title tag is always fixed

**render() + content_template() skeleton:**

```php
// ✅ REQUIRED on every widget — removes the redundant inner wrapper div.
// Return true ONLY if your render() output physically requires the inner wrapper.
// Source: developers.elementor.com/docs/widgets/widget-inner-wrapper/
public function has_widget_inner_wrapper(): bool {
    return false;
}

// ✅ Return false to enable output caching for static widgets.
// Source: developers.elementor.com/docs/widgets/widget-output-caching/
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
    <div class="myplugin-accordion">
        <?php foreach ( $tabs as $index => $tab ) :
            $tab_count  = $index + 1;
            $tab_id     = 'myplugin-accordion-' . $id_int . $tab_count;
            $tag        = \Elementor\Utils::validate_html_tag( $settings['title_html_tag'] );
            $is_active  = 1 === $tab_count;
        ?>
        <?php
            // ✅ aria-controls must point to the content panel ID — required by ARIA APG Accordion pattern.
            // Without it, screen readers announce expand/collapse state but cannot navigate to the region.
            // Source: w3.org/WAI/ARIA/apg/patterns/accordion/
            $content_id = 'myplugin-content-' . $id_int . $tab_count;
        ?>
        <div class="myplugin-accordion-item elementor-repeater-item-<?php echo esc_attr( $tab['_id'] ); ?>">
            <<?php echo esc_attr( $tag ); ?>
                id="<?php echo esc_attr( $tab_id ); ?>"
                class="myplugin-tab-title<?php echo $is_active ? ' myplugin-active' : ''; ?>"
                aria-expanded="<?php echo $is_active ? 'true' : 'false'; ?>"
                aria-controls="<?php echo esc_attr( $content_id ); ?>"
                role="button"
                tabindex="0">
                <?php
                // ✅ BUG FIX: selected_icon and selected_active_icon are WIDGET-LEVEL controls,
                // not per-item repeater controls. Check $settings, not $tab.
                // Using $tab['selected_icon'] would always be empty — it's not in the repeater.
                if ( ! empty( $settings['selected_icon']['value'] ) ) : ?>
                    <span class="myplugin-accordion-icon myplugin-accordion-icon--<?php echo $is_active ? 'opened' : 'closed'; ?>">
                        <span class="myplugin-accordion-icon-opened">
                            <?php \Elementor\Icons_Manager::render_icon( $settings['selected_active_icon'], [ 'aria-hidden' => 'true' ] ); ?>
                        </span>
                        <span class="myplugin-accordion-icon-closed">
                            <?php \Elementor\Icons_Manager::render_icon( $settings['selected_icon'], [ 'aria-hidden' => 'true' ] ); ?>
                        </span>
                    </span>
                <?php endif; ?>
                <span class="myplugin-accordion-title"><?php echo esc_html( $tab['tab_title'] ); ?></span>
            </<?php echo esc_attr( $tag ); ?>>
            <div id="<?php echo esc_attr( $content_id ); ?>"
                 class="myplugin-tab-content<?php echo $is_active ? ' myplugin-active' : ''; ?>"
                 role="region"
                 aria-labelledby="<?php echo esc_attr( $tab_id ); ?>">
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
    <div class="myplugin-accordion">
        <# _.each( settings.tabs, function( tab, index ) {
            var isActive = 0 === index;
            var tabId    = 'myplugin-accordion-' + index;
            var contentId = 'myplugin-content-' + index;
            // ✅ BUG FIX: Both icons are WIDGET-LEVEL controls (settings.*), not per-item (tab.*).
            // tab.selected_icon does not exist — it is not in the repeater fields.
            // Read settings.selected_icon for the closed icon, settings.selected_active_icon for open.
            var iconHTML = settings.selected_icon && settings.selected_icon.value
                ? elementor.helpers.renderIcon( view, settings.selected_icon, { 'aria-hidden': true }, 'i', 'object' )
                : null;
            var activeIconHTML = settings.selected_active_icon && settings.selected_active_icon.value
                ? elementor.helpers.renderIcon( view, settings.selected_active_icon, { 'aria-hidden': true }, 'i', 'object' )
                : null;
        #>
        <div class="myplugin-accordion-item">
            <{{{ tag }}} id="{{ tabId }}"
                class="myplugin-tab-title{{ isActive ? ' myplugin-active' : '' }}"
                aria-expanded="{{ isActive ? 'true' : 'false' }}"
                aria-controls="{{ contentId }}"
                role="button" tabindex="0">
                <# if ( iconHTML ) { #>
                    <span class="myplugin-accordion-icon">
                        <span class="myplugin-accordion-icon-opened">{{{ activeIconHTML ? activeIconHTML.value : '' }}}</span>
                        <span class="myplugin-accordion-icon-closed">{{{ iconHTML.value }}}</span>
                    </span>
                <# } #>
                <span class="myplugin-accordion-title">{{{ tab.tab_title }}}</span>
            </{{{ tag }}}>
            <div id="{{ contentId }}"
                 class="myplugin-tab-content{{ isActive ? ' myplugin-active' : '' }}"
                 role="region"
                 aria-labelledby="{{ tabId }}">
                {{{ tab.tab_content }}}
            </div>
        </div>
        <# } ); #>
    </div>
    <?php
}
```
