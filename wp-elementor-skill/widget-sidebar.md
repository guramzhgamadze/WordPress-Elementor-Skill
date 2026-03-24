# Widget Boilerplate — Sidebar

> **When to use this file:** Load whenever building a widget that outputs a registered WordPress sidebar.
> Verified against `elementor/includes/widgets/sidebar.php` (Elementor 3.35+).

---

```php
protected function register_controls(): void {

    $this->start_controls_section( 'section_sidebar', [
        'label' => esc_html__( 'Sidebar', 'myplugin' ),
        'tab'   => \Elementor\Controls_Manager::TAB_CONTENT,
    ] );

    // ✅ Build options list from registered sidebars at registration time
    $sidebars_options = [];
    if ( isset( $GLOBALS['wp_registered_sidebars'] ) ) {
        foreach ( $GLOBALS['wp_registered_sidebars'] as $sidebar_id => $sidebar ) {
            $sidebars_options[ $sidebar_id ] = $sidebar['name'];
        }
    }

    $this->add_control( 'sidebar', [
        'label'   => esc_html__( 'Choose Sidebar', 'myplugin' ),
        'type'    => \Elementor\Controls_Manager::SELECT,
        'default' => array_key_first( $sidebars_options ) ?? '',
        'options' => $sidebars_options,
    ] );

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

// ✅ MUST be true — dynamic_sidebar() outputs registered sidebar widgets which may
// include user-specific content (login forms, cart widgets, recent posts etc.).
// Returning false would cache one user's sidebar output and serve it to all visitors.
// Source: developers.elementor.com/docs/widgets/widget-output-caching/
protected function is_dynamic_content(): bool {
    return true;
}

protected function render(): void {
    $sidebar = $this->get_settings_for_display()['sidebar'];
    if ( ! empty( $sidebar ) && is_active_sidebar( $sidebar ) ) {
        dynamic_sidebar( $sidebar );
    }
}
```

---



**content_template() skeleton:**

```php
protected function content_template(): void {
    // Sidebar cannot be previewed in the editor — dynamic PHP output only.
}
```
