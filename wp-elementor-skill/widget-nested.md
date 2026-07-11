# Widget Guide — Nested Tabs & Nested Accordion

> **When to use this file:** Load when building a **Nested** Tabs or Accordion widget — where
> each panel is a real **Container** the user fills with any elements (not a single
> rich-text/repeater field). This mirrors Elementor's native **Nested Tabs** (free, 3.10/3.15+)
> and **Nested Accordion** (free, 3.19+).
>
> **Use the classic boilerplates instead** (`widget-tabs.md`, `widget-accordion.md`,
> `widget-toggle.md`) when each panel only needs simple text/WYSIWYG content. Those are simpler,
> fully self-contained `Widget_Base` widgets. Reach for **this** guide only when panels must hold
> arbitrary nested widgets/containers.

---

## ⚠️ Read this first — Nested is a different, partly-internal API

Nested widgets do **not** extend `\Elementor\Widget_Base`. They extend
`\Elementor\Modules\NestedElements\Base\Widget_Nested_Base`, and each "panel" is a child
**Container** managed by Elementor's Nested Elements editor module. That module's editor-side
behaviour (keeping the title repeater in sync with the child containers when items are
added/removed/reordered) is provided by Elementor's own JS handlers — the **third-party-facing
parts of this API are still only partially documented.**

What this means in practice:
- The **PHP class structure below is stable and correct** for 3.x–4.x.
- The **editor add/remove/reorder sync** relies on Elementor's `nested-elements` module. If you
  need fully custom editor behaviour, you are in semi-internal territory — pin your Elementor
  version and test on upgrades.
- **Requirements:** the **Container** layout is the default in Elementor 4.x, and Nested Elements
  is stable. On older sites both must be enabled under Elementor → Settings → Features.

Source: developers.elementor.com/docs/widgets/ · `elementor/modules/nested-tabs/` ·
`elementor/modules/nested-accordion/` (Elementor GitHub).

---

## Nested Tabs — class skeleton

```php
// includes/class-myplugin-nested-tabs.php
defined( 'ABSPATH' ) || exit;

use Elementor\Controls_Manager;
use Elementor\Repeater;
use Elementor\Modules\NestedElements\Base\Widget_Nested_Base;
use Elementor\Modules\NestedElements\Controls\Control_Nested_Repeater;

class MyPlugin_Nested_Tabs extends Widget_Nested_Base {

    public function get_name(): string      { return 'myplugin-nested-tabs'; }
    public function get_title(): string     { return esc_html__( 'My Nested Tabs', 'myplugin' ); }
    public function get_icon(): string      { return 'eicon-tabs'; }
    public function get_categories(): array { return [ 'general' ]; }

    // ✅ The default child containers created when the widget is dropped in.
    // Each entry is a 'container' element the user then fills with any widgets.
    protected function get_default_children_elements(): array {
        return [
            [
                'elType'   => 'container',
                'settings' => [ '_title' => esc_html__( 'Tab #1', 'myplugin' ) ],
                'elements' => [],
            ],
            [
                'elType'   => 'container',
                'settings' => [ '_title' => esc_html__( 'Tab #2', 'myplugin' ) ],
                'elements' => [],
            ],
        ];
    }

    // ✅ Which repeater control holds each item's title — Elementor uses this to keep the
    // title repeater and the child containers in sync.
    protected function get_default_repeater_title_setting_key(): string {
        return 'tab_title';
    }

    // ✅ Title template for newly-added children (%d = index).
    protected function get_default_children_title(): string {
        return esc_html__( 'Tab #%d', 'myplugin' );
    }

    // ✅ Selectors the editor uses to place the empty-state "add element" placeholder.
    protected function get_default_children_placeholder_selector(): string {
        return '.myplugin-nested-tabs__content';
    }

    protected function get_default_children_container_placeholder_selector(): string {
        return '.e-con';
    }

    protected function get_html_wrapper_class(): string {
        return 'myplugin-nested-tabs';
    }

    protected function register_controls(): void {

        $this->start_controls_section( 'section_tabs', [
            'label' => esc_html__( 'Tabs', 'myplugin' ),
            'tab'   => Controls_Manager::TAB_CONTENT,
        ] );

        $repeater = new Repeater();

        $repeater->add_control( 'tab_title', [
            'label'       => esc_html__( 'Title', 'myplugin' ),
            'type'        => Controls_Manager::TEXT,
            'default'     => esc_html__( 'Tab Title', 'myplugin' ),
            'placeholder' => esc_html__( 'Tab Title', 'myplugin' ),
            'dynamic'     => [ 'active' => true ],
        ] );

        // ✅ Control_Nested_Repeater (NOT the plain REPEATER) — this is the repeater type that
        // syncs each row with a child container. 'tabs' is the items array; prevent_empty stops
        // the user deleting the last tab.
        $this->add_control( 'tabs', [
            'label'         => esc_html__( 'Tabs Items', 'myplugin' ),
            'type'          => Control_Nested_Repeater::CONTROL_TYPE,
            'fields'        => $repeater->get_controls(),
            'default'       => [
                [ 'tab_title' => esc_html__( 'Tab #1', 'myplugin' ) ],
                [ 'tab_title' => esc_html__( 'Tab #2', 'myplugin' ) ],
            ],
            'title_field'   => '{{{ tab_title }}}',
            'prevent_empty' => true,
        ] );

        $this->end_controls_section();

        // Add TAB_STYLE sections for title/active/content colours + typography exactly as in
        // widget-tabs.md — every visual property must still be a control (SKILL.md §5).
    }

    public function has_widget_inner_wrapper(): bool {
        return false;
    }

    // ✅ Nested content can contain anything (forms, dynamic widgets) — leave caching off
    // unless you are certain every child is static.
    protected function is_dynamic_content(): bool {
        return true;
    }

    protected function render(): void {
        $settings = $this->get_settings_for_display();
        $tabs     = $settings['tabs'] ?? [];
        if ( empty( $tabs ) ) {
            return;
        }

        // ✅ Per-instance id prefix prevents aria-controls collisions across multiple widgets.
        $id_int = substr( $this->get_id(), 0, 6 );
        ?>
        <div class="myplugin-nested-tabs">
            <div class="myplugin-nested-tabs__headings" role="tablist" aria-orientation="horizontal">
                <?php foreach ( $tabs as $index => $item ) :
                    $n          = $index + 1;
                    $is_active  = 1 === $n;
                    $title_id   = "myplugin-tab-title-{$id_int}{$n}";
                    $content_id = "myplugin-tab-content-{$id_int}{$n}";
                    ?>
                    <button type="button"
                            id="<?php echo esc_attr( $title_id ); ?>"
                            class="myplugin-nested-tabs__title<?php echo $is_active ? ' myplugin-active' : ''; ?>"
                            role="tab"
                            aria-controls="<?php echo esc_attr( $content_id ); ?>"
                            aria-selected="<?php echo $is_active ? 'true' : 'false'; ?>"
                            tabindex="<?php echo $is_active ? '0' : '-1'; ?>">
                        <?php echo esc_html( $item['tab_title'] ); ?>
                    </button>
                <?php endforeach; ?>
            </div>

            <div class="myplugin-nested-tabs__content">
                <?php foreach ( $tabs as $index => $item ) :
                    $n          = $index + 1;
                    $is_active  = 1 === $n;
                    $title_id   = "myplugin-tab-title-{$id_int}{$n}";
                    $content_id = "myplugin-tab-content-{$id_int}{$n}";
                    ?>
                    <div id="<?php echo esc_attr( $content_id ); ?>"
                         class="myplugin-nested-tabs__panel<?php echo $is_active ? ' myplugin-active' : ''; ?>"
                         role="tabpanel"
                         aria-labelledby="<?php echo esc_attr( $title_id ); ?>"
                         <?php echo $is_active ? '' : 'hidden'; ?>>
                        <?php
                        // ✅ THE KEY CALL: print the matching child Container by index.
                        // Widget_Nested_Base maps repeater item N to child container N.
                        $this->print_child( $index );
                        ?>
                    </div>
                <?php endforeach; ?>
            </div>
        </div>
        <?php
    }

    // ✅ Editor preview (Backbone). Render the title buttons; child containers are injected by
    // Elementor's nested-elements editor handler, so the panel markup is minimal here.
    protected function content_template(): void {
        ?>
        <div class="myplugin-nested-tabs">
            <div class="myplugin-nested-tabs__headings" role="tablist">
                <# _.each( settings.tabs, function( item, index ) { var isActive = 0 === index; #>
                    <button type="button"
                            class="myplugin-nested-tabs__title{{ isActive ? ' myplugin-active' : '' }}"
                            role="tab" aria-selected="{{ isActive ? 'true' : 'false' }}"
                            tabindex="{{ isActive ? '0' : '-1' }}">{{ item.tab_title }}</button>
                <# } ); #>
            </div>
            <div class="myplugin-nested-tabs__content"></div>
        </div>
        <?php
    }
}
```

---

## Registration

```php
// Nested widgets register through the normal hook.
add_action( 'elementor/widgets/register', function( \Elementor\Widgets_Manager $manager ) {
    require_once MYPLUGIN_PATH . 'includes/class-myplugin-nested-tabs.php';
    $manager->register( new MyPlugin_Nested_Tabs() );
} );
```

---

## Frontend interaction handler (required)

The markup above is static until JS wires up tab switching. Register a frontend handler under
`elementor/frontend/init` (same pattern as `js-css-standards.md`) — **without `strategy:defer`**:

```js
window.addEventListener( 'elementor/frontend/init', () => {
  window.elementorFrontend.hooks.addAction(
    'frontend/element_ready/myplugin-nested-tabs.default',
    ( $scope ) => {
      const root = $scope && $scope[0];
      if ( ! root ) return;

      const titles = [ ...root.querySelectorAll( '.myplugin-nested-tabs__title' ) ];
      const panels = [ ...root.querySelectorAll( '.myplugin-nested-tabs__panel' ) ];

      const activate = ( i ) => {
        titles.forEach( ( t, n ) => {
          const on = n === i;
          t.classList.toggle( 'myplugin-active', on );
          t.setAttribute( 'aria-selected', String( on ) );
          t.tabIndex = on ? 0 : -1;
        } );
        panels.forEach( ( p, n ) => {
          p.classList.toggle( 'myplugin-active', n === i );
          p.toggleAttribute( 'hidden', n !== i );
        } );
      };

      titles.forEach( ( title, i ) => {
        title.addEventListener( 'click', () => activate( i ) );
        // ✅ ARIA APG Tabs keyboard support — arrows move focus, Enter/Space activate.
        title.addEventListener( 'keydown', ( e ) => {
          if ( e.key === 'ArrowRight' || e.key === 'ArrowDown' ) {
            e.preventDefault(); titles[ ( i + 1 ) % titles.length ].focus();
          } else if ( e.key === 'ArrowLeft' || e.key === 'ArrowUp' ) {
            e.preventDefault(); titles[ ( i - 1 + titles.length ) % titles.length ].focus();
          } else if ( e.key === 'Enter' || e.key === ' ' ) {
            e.preventDefault(); activate( i );
          }
        } );
      } );
    }
  );
} );
```

---

## Nested Accordion — what changes

Nested Accordion shares the same `Widget_Nested_Base` structure. Differences from Nested Tabs:

1. **Markup:** use the native HTML disclosure pattern — each item is a `<details>`/`<summary>`
   pair (or a `<button aria-expanded>` + region). `<details>`/`<summary>` gives you
   keyboard + screen-reader support **for free**, so prefer it:
   ```php
   foreach ( $items as $index => $item ) : ?>
       <details class="myplugin-nested-accordion__item"<?php echo 0 === $index ? ' open' : ''; ?>>
           <summary class="myplugin-nested-accordion__title"><?php echo esc_html( $item['item_title'] ); ?></summary>
           <div class="myplugin-nested-accordion__panel"><?php $this->print_child( $index ); ?></div>
       </details>
   <?php endforeach;
   ```
2. **Open behaviour:** Accordion = multiple panels may stay open (like `widget-toggle.md`), or
   add a `max_items_expended = 'one'` control and a small JS handler that closes siblings when one
   `<details>` opens (single-open, like `widget-accordion.md`).
3. **Icons:** expose `selected_icon` / `selected_active_icon` as **widget-level** `ICONS` controls
   and read them from `$settings`, not the repeater item — see the bug note in
   `widget-toggle.md`.
4. `get_default_repeater_title_setting_key()` → `'item_title'`; `get_html_wrapper_class()` →
   `'myplugin-nested-accordion'`; icon `eicon-accordion`.

> If you use `<details>`/`<summary>`, you do **not** need the keyboard handler above — the browser
> provides it. Only add JS for the optional single-open behaviour.

---

> **Remove / adapt checklist:**
> - Drop the frontend keyboard handler if you build the Accordion with `<details>`/`<summary>`.
> - Add TAB_STYLE controls for every colour/spacing/typography value (do not hardcode visuals).
> - Pin your Elementor version and re-test on upgrade — the nested-elements editor sync is
>   semi-internal API.
