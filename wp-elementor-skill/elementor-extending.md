# Elementor — Extension Points Beyond Widgets

> **When to read this file:** Extending Elementor *past* the basics in `elementor-patterns.md`
> (widgets, dynamic tags, loop queries, form actions, theme conditions). This covers the rest of
> the documented **V3 extension surface**: custom **form fields**, **theme locations**, injecting
> controls into **native** widgets, **Finder** and **context-menu** items, plus a consolidated
> **hooks** reference and the **deprecations** to avoid.
>
> Canonical source — verify signatures here, the API evolves: developers.elementor.com/docs/
>
> ℹ️ **V4 / Atomic note (verified mid-2026):** everything below is the stable **V3** extension API
> (the skill's target). There is **still no published third-party PHP API for building custom
> Atomic Elements** — no `Atomic_Widget_Base`; the official docs still point to `Widget_Base` for
> custom widgets. Atomic Elements are documented only as a *data structure*
> (developers.elementor.com/docs/data-structure/atomic-elements/), not a creation API. **Keep
> building with V3 `Widget_Base`** — see SKILL.md §1. (Newer 4.x **control types** like
> `VISUAL_CHOICE` are usable from V3 widgets today — see §7 below.)

---

## 1. Custom Form Field (Elementor Pro)

The sibling of a Form **Action** (`elementor-patterns.md`). A custom **field** adds a new input
type to the Form widget. Extends `\ElementorPro\Modules\Forms\Fields\Field_Base`.

```php
// Requires: Elementor Pro
class MyPlugin_Range_Field extends \ElementorPro\Modules\Forms\Fields\Field_Base {

    public function get_type(): string  { return 'myplugin_range'; }
    public function get_name(): string  { return esc_html__( 'Range Slider', 'myplugin' ); }

    // ✅ Pro 3.28+: declare field assets via METHODS (the old $depended_scripts /
    // $depended_styles properties on Field_Base are deprecated).
    public function get_script_depends(): array { return [ 'myplugin-range-field' ]; }

    // $item = this field's settings; $form = the Form widget instance.
    public function render( $item, $item_index, $form ): void {
        $form->add_render_attribute( 'input' . $item_index, [
            'type'  => 'range',
            'min'   => $item['min'] ?? 0,
            'max'   => $item['max'] ?? 100,
            'class' => 'elementor-field-textual myplugin-range',
        ] );
        // get_render_attribute_string() returns pre-built, safe attribute markup.
        echo '<input ' . $form->get_render_attribute_string( 'input' . $item_index ) . '>'; // phpcs:ignore WordPress.Security.EscapeOutput.OutputNotEscaped -- Elementor-built attribute string
    }

    // Optional: add per-field settings controls to the Form's field repeater.
    public function update_controls( $widget ): void {
        $elementor = \ElementorPro\Plugin::elementor();
        $control   = $elementor->controls_manager->get_control_from_stack(
            $widget->get_unique_name(), 'form_fields'
        );
        if ( is_wp_error( $control ) ) { return; }
        $field_controls = [ /* … add 'min'/'max' controls with 'condition' => ['field_type'=>$this->get_type()] … */ ];
        $control['fields'] = \ElementorPro\Core\Utils::array_inject(
            $control['fields'], 'placeholder', $field_controls
        );
        $widget->update_control( 'form_fields', $control );
    }
}

// ✅ Register on the Pro fields registrar (parallels elementor_pro/forms/actions/register).
add_action( 'elementor_pro/forms/fields/register', function( $fields_registrar ) {
    require_once MYPLUGIN_PATH . 'includes/class-myplugin-range-field.php';
    $fields_registrar->register( new MyPlugin_Range_Field() );
} );
```

**Validation / sanitization** of submitted values happens on the form lifecycle hooks, not the
field class:
```php
add_action( 'elementor_pro/forms/validation', function( $record, $ajax_handler ) {
    foreach ( $record->get_field( [ 'type' => 'myplugin_range' ] ) as $id => $field ) {
        if ( ! is_numeric( $field['value'] ) ) {
            $ajax_handler->add_error( $id, esc_html__( 'Invalid value.', 'myplugin' ) );
        }
    }
}, 10, 2 );
```
Other useful form hooks: `elementor_pro/forms/process` (after validation), and
`elementor_pro/forms/new_record` (act on a successful submission).

---

## 2. Theme Locations (Elementor Pro Theme Builder)

The sibling of a Theme **Condition**. A **location** is a slot a Theme Builder template can fill
(your theme/plugin defines `header`, `footer`, `single`, or a custom one).

```php
// Requires: Elementor Pro. Register the location.
add_action( 'elementor/theme/register_locations', function( $manager ) {
    $manager->register_location( 'myplugin_before_content', [
        'label'           => esc_html__( 'Before Content', 'myplugin' ),
        'multiple'        => false,   // true = allow stacking multiple templates
        'edit_in_content' => true,    // edit inline in the content area
    ] );
} );
```
```php
// In the theme/plugin template, output the location with a graceful fallback:
if ( function_exists( 'elementor_theme_do_location' ) ) {
    if ( ! elementor_theme_do_location( 'myplugin_before_content' ) ) {
        // No template assigned → render default markup here.
    }
}
```
Themes that want Elementor to fully manage header/footer declare support:
`add_theme_support( 'elementor' );` and register their locations the same way.

---

## 3. Add controls to — or filter the output of — a NATIVE widget

You don't always build a new widget; often you just want to extend an existing one.

**Inject a control into a native widget's existing section** (no subclassing):
```php
// Hook shape: elementor/element/{element_name}/{section_id}/before_section_end (or after_section_start)
add_action( 'elementor/element/heading/section_title/before_section_end', function( $element, $args ) {
    $element->add_control( 'myplugin_badge', [
        'label' => esc_html__( 'Badge Text', 'myplugin' ),
        'type'  => \Elementor\Controls_Manager::TEXT,
    ] );
}, 10, 2 );
```
Find the `{element_name}` and `{section_id}` by inspecting the native widget's
`register_controls()` (e.g. `heading` / `section_title`, `button` / `section_button`).

**Filter a native widget's rendered HTML:**
```php
add_filter( 'elementor/widget/render_content', function( string $content, $widget ): string {
    if ( 'heading' !== $widget->get_name() ) { return $content; }
    $badge = $widget->get_settings_for_display( 'myplugin_badge' );
    if ( $badge ) {
        $content .= '<span class="myplugin-badge">' . esc_html( $badge ) . '</span>';
    }
    return $content;
}, 10, 2 );
```

---

## 4. Finder & Context Menu (editor UX — niche)

**Finder** (Ctrl/Cmd+E quick search) — add your plugin's admin destinations:
```php
add_action( 'elementor/finder/register', function( $categories_manager ) {
    $categories_manager->register( new MyPlugin_Finder_Category() );
} );

class MyPlugin_Finder_Category extends \Elementor\Core\Common\Modules\Finder\Base_Category {
    public function get_title(): string { return esc_html__( 'My Plugin', 'myplugin' ); }
    public function get_category_items( array $options = [] ): array {
        return [
            [
                'title'    => esc_html__( 'Settings', 'myplugin' ),
                'icon'     => 'settings',
                'url'      => admin_url( 'admin.php?page=myplugin' ),
                'keywords' => [ 'myplugin', 'settings', 'options' ],
            ],
        ];
    }
}
```
> ⚠️ The Finder registrar hook/signature has shifted across versions (older builds used the
> `elementor/finder/categories` filter). Verify against current docs before shipping.

**Context Menu** (editor right-click) is **JS-side**, via the editor hooks:
```js
elementor.hooks.addFilter( 'elements/widget/contextMenuGroups', ( groups, view ) => {
    groups.push( {
        name:    'myplugin',
        actions: [ { name: 'my_action', title: 'My Action', callback: () => { /* … */ } } ],
    } );
    return groups;
} );
```

---

## 5. Hooks quick-reference (the extension surface)

**PHP — registration & rendering:**

| Hook | Use |
|---|---|
| `elementor/widgets/register` | Register custom widgets (since 3.5; old: `widgets_registered`) |
| `elementor/elements/categories_registered` | Register a widget panel category |
| `elementor/dynamic_tags/register` | Register dynamic tags + tag groups |
| `elementor/query/{query_id}` | Filter a Loop Grid / Posts query by Query ID |
| `elementor/element/{el}/{section}/before_section_end` | Inject a control into a native widget |
| `elementor/widget/render_content` (filter) | Modify a widget's rendered HTML |
| `elementor/frontend/the_content` (filter) | Modify rendered Elementor page content |
| `elementor/theme/register_conditions` | Custom Theme Builder display conditions (Pro) |
| `elementor/theme/register_locations` | Custom Theme Builder locations (Pro) |
| `elementor_pro/forms/actions/register` | Custom form submit action (Pro) |
| `elementor_pro/forms/fields/register` | Custom form field type (Pro) |
| `elementor_pro/forms/validation` / `process` / `new_record` | Form submission lifecycle (Pro) |
| `elementor/finder/register` | Add Finder items |
| `elementor/frontend/after_register_scripts` / `after_enqueue_styles` | Frontend asset timing |
| `elementor/editor/after_enqueue_scripts` / `after_enqueue_styles` | Editor-only assets |
| `elementor/preview/enqueue_styles` | Preview-iframe-only assets |

**JS — editor & frontend:**

| Hook | Use |
|---|---|
| `elementor/frontend/init` (bind with **jQuery** — see `js-css-standards.md`) | Frontend boot |
| `frontend/element_ready/{widget}.default` | Per-widget frontend handler (incl. AJAX-loaded) |
| `panel/open_editor/widget/{widget}` | Editor panel opened for a widget |
| `elements/widget/contextMenuGroups` (filter) | Editor context-menu items |

---

## 6. Deprecations to avoid

These still "work" but are deprecated — using them invites breakage and Plugin Check / review
flags. Use the right-hand column.

| Deprecated | Use instead | Since |
|---|---|---|
| `get_id_int()` | `get_id()` | 3.1 |
| `_register_controls()` | `register_controls()` | 3.1 |
| `_content_template()` | `content_template()` | 3.1 |
| `$depended_scripts` / `$depended_styles` properties | `get_script_depends()` / `get_style_depends()` methods | 3.24 / Pro 3.28 |
| `Scheme_Color` / `Scheme_Typography`, `Group_Control_Scheme_*` | `Global_Colors` / `Global_Typography` globals | 3.0 |
| `elementor/widgets/widgets_registered` | `elementor/widgets/register` | 3.5 |
| Targeting `.elementor-widget-container` in CSS/JS | the widget root / your own BEM class (+ `has_widget_inner_wrapper(): false`) | 3.25+ / V4 |
| `window.addEventListener('elementor/frontend/init', …)` as the *only* binding | `jQuery(window).on('elementor/frontend/init', …)` | — |

> Elementor's own deprecation log: developers.elementor.com/docs/deprecations/ — check it each
> major release; deprecated APIs get a `_deprecated_*` notice under `WP_DEBUG`.

---

## 7. Newer control types (4.x) — `VISUAL_CHOICE`

A V3-compatible control added in the 4.x line: an **image-based** choice picker (each option is a
visual/SVG, not just an icon-glyph like `CHOOSE`). Ideal for layout / skin / structure pickers.

```php
$this->add_control( 'structure', [
    'label'       => esc_html__( 'Layout', 'myplugin' ),
    'type'        => \Elementor\Controls_Manager::VISUAL_CHOICE,
    'default'     => 'grid',
    'label_block' => true,
    'columns'     => 2,                       // grid columns in the panel
    'options'     => [
        'grid'    => [
            'title' => esc_attr__( 'Grid', 'myplugin' ),
            'image' => plugins_url( 'assets/img/layout-grid.svg', MYPLUGIN_FILE ),
        ],
        'masonry' => [
            'title' => esc_attr__( 'Masonry', 'myplugin' ),
            'image' => plugins_url( 'assets/img/layout-masonry.svg', MYPLUGIN_FILE ),
        ],
    ],
    // Like CHOOSE, you can drive CSS directly when the value maps to a class/value:
    'prefix_class' => 'myplugin-layout-',     // → 'myplugin-layout-grid' on the wrapper
] );

// render(): the value is the selected option KEY (a string).
$layout = $this->get_settings_for_display( 'structure' );  // 'grid' | 'masonry'
```

- Value is the **option key string** (read with `get_settings_for_display('structure')`).
- `image` is a URL — ship your own SVGs; don't `wp_kses()` them (see `field-notes.md` §6 / §2 on
  the `viewBox` lowercasing trap if you ever inline them).
- Compare with **`CHOOSE`** (icon-glyph toggle, e.g. alignment) and **`SELECT`/`SELECT2`** (plain
  dropdown) — pick `VISUAL_CHOICE` only when an image genuinely communicates the option better.
> Source: developers.elementor.com/docs/editor-controls/control-visual-choice/
