# Widget Boilerplate — Tabs

> **When to use this file:** Load whenever building a widget that shows multiple content panels via clickable tabs.
> Verified against `elementor/includes/widgets/tabs.php` (Elementor 3.35+).
> ✅ Uses REPEATER control — each tab is one repeater item with title + content.

---

```php
protected function register_controls(): void {

    // ── CONTENT ───────────────────────────────────────────────
    $this->start_controls_section( 'section_tabs', [
        'label' => esc_html__( 'Tabs', 'myplugin' ),
        'tab'   => \Elementor\Controls_Manager::TAB_CONTENT,
    ] );

    $repeater = new \Elementor\Repeater();

    $repeater->add_control( 'tab_title', [
        'label'       => esc_html__( 'Title', 'myplugin' ),
        'type'        => \Elementor\Controls_Manager::TEXT,
        'default'     => esc_html__( 'Tab Title', 'myplugin' ),
        'placeholder' => esc_html__( 'Tab Title', 'myplugin' ),
        'label_block' => true,
        'dynamic'     => [ 'active' => true ],
    ] );

    $repeater->add_control( 'tab_content', [
        'label'      => esc_html__( 'Content', 'myplugin' ),
        'type'       => \Elementor\Controls_Manager::WYSIWYG,
        'default'    => esc_html__( 'Tab Content', 'myplugin' ),
        'show_label' => false,
        'dynamic'    => [ 'active' => true ],
    ] );

    $this->add_control( 'tabs', [
        'label'       => esc_html__( 'Tabs Items', 'myplugin' ),
        'type'        => \Elementor\Controls_Manager::REPEATER,
        'fields'      => $repeater->get_controls(),
        'default'     => [
            [ 'tab_title' => esc_html__( 'Tab #1', 'myplugin' ), 'tab_content' => esc_html__( 'Tab content goes here.', 'myplugin' ) ],
            [ 'tab_title' => esc_html__( 'Tab #2', 'myplugin' ), 'tab_content' => esc_html__( 'Tab content goes here.', 'myplugin' ) ],
            [ 'tab_title' => esc_html__( 'Tab #3', 'myplugin' ), 'tab_content' => esc_html__( 'Tab content goes here.', 'myplugin' ) ],
        ],
        'title_field' => '{{{ tab_title }}}',
    ] );

    $this->add_control( 'type', [
        'label'        => esc_html__( 'Type', 'myplugin' ),
        'type'         => \Elementor\Controls_Manager::SELECT,
        'default'      => 'horizontal',
        'options'      => [
            'horizontal' => esc_html__( 'Horizontal', 'myplugin' ),
            'vertical'   => esc_html__( 'Vertical',   'myplugin' ),
        ],
        'prefix_class' => 'elementor-tabs-view-',
        'separator'    => 'before',
    ] );

    $this->add_control( 'tab_align_horizontal', [
        'label'        => esc_html__( 'Tabs Alignment', 'myplugin' ),
        'type'         => \Elementor\Controls_Manager::CHOOSE,
        'options'      => [
            'left'    => [ 'title' => esc_html__( 'Left',    'myplugin' ), 'icon' => 'eicon-text-align-left'    ],
            'center'  => [ 'title' => esc_html__( 'Center',  'myplugin' ), 'icon' => 'eicon-text-align-center'  ],
            'right'   => [ 'title' => esc_html__( 'Right',   'myplugin' ), 'icon' => 'eicon-text-align-right'   ],
            'justify' => [ 'title' => esc_html__( 'Justify', 'myplugin' ), 'icon' => 'eicon-text-align-justify' ],
        ],
        'prefix_class' => 'elementor-tabs-alignment-',
        'condition'    => [ 'type' => 'horizontal' ],
    ] );

    $this->end_controls_section();

    // ── STYLE ─────────────────────────────────────────────────
    $this->start_controls_section( 'section_tabs_style', [
        'label' => esc_html__( 'Tabs', 'myplugin' ),
        'tab'   => \Elementor\Controls_Manager::TAB_STYLE,
    ] );

    $this->add_control( 'navigation_color', [
        'label'     => esc_html__( 'Text Color', 'myplugin' ),
        'type'      => \Elementor\Controls_Manager::COLOR,
        'selectors' => [ '{{WRAPPER}} .myplugin-tab-title' => 'color: {{VALUE}};' ],
    ] );

    $this->add_control( 'tab_active_color', [
        'label'     => esc_html__( 'Active Color', 'myplugin' ),
        'type'      => \Elementor\Controls_Manager::COLOR,
        'global'    => [ 'default' => \Elementor\Core\Kits\Documents\Tabs\Global_Colors::COLOR_PRIMARY ],
        'selectors' => [ '{{WRAPPER}} .myplugin-tab-title.myplugin-active' => 'color: {{VALUE}};' ],
    ] );

    $this->add_group_control(
        \Elementor\Group_Control_Typography::get_type(),
        [
            'name'     => 'tab_typography',
            'global'   => [ 'default' => \Elementor\Core\Kits\Documents\Tabs\Global_Typography::TYPOGRAPHY_PRIMARY ],
            'selector' => '{{WRAPPER}} .myplugin-tab-title',
        ]
    );

    $this->end_controls_section();

    $this->start_controls_section( 'section_content_style', [
        'label' => esc_html__( 'Content', 'myplugin' ),
        'tab'   => \Elementor\Controls_Manager::TAB_STYLE,
    ] );

    $this->add_control( 'content_color', [
        'label'     => esc_html__( 'Color', 'myplugin' ),
        'type'      => \Elementor\Controls_Manager::COLOR,
        'global'    => [ 'default' => \Elementor\Core\Kits\Documents\Tabs\Global_Colors::COLOR_TEXT ],
        'selectors' => [ '{{WRAPPER}} .myplugin-tab-content' => 'color: {{VALUE}};' ],
    ] );

    $this->add_group_control(
        \Elementor\Group_Control_Typography::get_type(),
        [
            'name'     => 'content_typography',
            'global'   => [ 'default' => \Elementor\Core\Kits\Documents\Tabs\Global_Typography::TYPOGRAPHY_TEXT ],
            'selector' => '{{WRAPPER}} .myplugin-tab-content',
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
    $tabs     = $settings['tabs'];
    if ( empty( $tabs ) ) return;

    // ✅ CRITICAL: Include widget instance ID in all element IDs.
    // Using only $tab_count (1, 2, 3…) causes ID collisions when multiple Tab widgets
    // appear on the same page — aria-controls points to the WRONG panel in the second widget.
    // substr( $this->get_id(), 0, 3 ) gives a 3-char prefix unique to this widget instance.
    $id_int = substr( $this->get_id(), 0, 3 );
    ?>
    <div class="myplugin-tabs">
        <div class="myplugin-tabs-wrapper" role="tablist">
            <?php foreach ( $tabs as $index => $tab ) :
                $tab_count = $index + 1;
                $is_active = 1 === $tab_count;
            ?>
                <div class="myplugin-tab-title<?php echo $is_active ? ' myplugin-active' : ''; ?>"
                     id="myplugin-tab-title-<?php echo esc_attr( $id_int . $tab_count ); ?>"
                     role="tab"
                     aria-controls="myplugin-tab-content-<?php echo esc_attr( $id_int . $tab_count ); ?>"
                     aria-selected="<?php echo $is_active ? 'true' : 'false'; ?>"
                     tabindex="<?php echo $is_active ? '0' : '-1'; ?>">
                    <?php echo esc_html( $tab['tab_title'] ); ?>
                </div>
            <?php endforeach; ?>
        </div>
        <div class="myplugin-tabs-content-wrapper">
            <?php foreach ( $tabs as $index => $tab ) :
                $tab_count = $index + 1;
                $is_active = 1 === $tab_count;
            ?>
                <div class="myplugin-tab-content<?php echo $is_active ? ' myplugin-active' : ''; ?>"
                     id="myplugin-tab-content-<?php echo esc_attr( $id_int . $tab_count ); ?>"
                     role="tabpanel"
                     aria-labelledby="myplugin-tab-title-<?php echo esc_attr( $id_int . $tab_count ); ?>">
                    <?php echo wp_kses_post( $tab['tab_content'] ); ?>
                </div>
            <?php endforeach; ?>
        </div>
    </div>
    <?php
}
```

> **Remove checklist:**
> - `type` + `tab_align_horizontal` → remove if tabs are always horizontal


**content_template() skeleton:**

```php
protected function content_template(): void {
    ?>
    <# if ( ! settings.tabs || ! settings.tabs.length ) { return; } #>
    <div class="myplugin-tabs">
        <div class="myplugin-tabs-wrapper" role="tablist">
            <# _.each( settings.tabs, function( tab, index ) {
                var isActive = 0 === index;
            #>
            <div class="myplugin-tab-title{{ isActive ? ' myplugin-active' : '' }}"
                 role="tab"
                 aria-selected="{{ isActive ? 'true' : 'false' }}"
                 tabindex="{{ isActive ? '0' : '-1' }}">
                {{{ tab.tab_title }}}
            </div>
            <# } ); #>
        </div>
        <div class="myplugin-tabs-content-wrapper">
            <# _.each( settings.tabs, function( tab, index ) {
                var isActive = 0 === index;
            #>
            <div class="myplugin-tab-content{{ isActive ? ' myplugin-active' : '' }}" role="tabpanel">
                {{{ tab.tab_content }}}
            </div>
            <# } ); #>
        </div>
    </div>
    <?php
}
```
