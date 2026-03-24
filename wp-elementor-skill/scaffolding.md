# Scaffolding — Plugin, Child Theme, CPT, AJAX

## Code Placement Rules

**Always declare placement at the top of every code response.**

| Destination | When to use |
|---|---|
| Child theme `functions.php` | Lightweight theme-scoped hooks, no reusability needed |
| Elementor → Site Settings → Custom Code | JS/CSS snippets injected at `wp_head`, `wp_footer`, `wp_body_open` |
| Dedicated plugin `includes/` class | Reusable logic, CPTs, REST endpoints, widget registration |
| `wp-content/mu-plugins/` | Must-load logic, network-wide on multisite, security-critical code |
| Elementor Widget PHP file | Custom `\Elementor\Widget_Base` extension, registered via hook |
| `wp-content/themes/mytheme-child/woocommerce/` | WooCommerce template overrides — **last resort only** |

---

## Plugin Scaffolding

Use when logic is reusable, complex, or must survive theme changes.

### File structure
```
/wp-content/plugins/myplugin/
├── myplugin.php                        ← Main bootstrap file
├── includes/
│   ├── class-myplugin.php              ← Singleton init class
│   ├── class-myplugin-hooks.php        ← All add_action / add_filter
│   ├── class-myplugin-assets.php       ← Enqueue logic
│   ├── class-myplugin-rest.php         ← REST API endpoints (if needed)
│   └── class-myplugin-widget.php       ← Elementor widget (if needed)
├── assets/
│   ├── css/myplugin.css
│   └── js/myplugin.js
└── README.md
```

### Main plugin file (always use this header):
```php
<?php
/**
 * Plugin Name:  My Plugin
 * Description:  Short description.
 * Version:      1.0.0
 * Requires at least: 6.9
 * Requires PHP: 8.3
 * Requires Plugins: elementor
 * Author:       Your Name
 * License:      GPL-2.0-or-later
 * Text Domain:  myplugin
 */
// ↑ "Requires Plugins" is the WP 6.5+ native plugin dependency declaration.
// WordPress.org plugin slugs only — do NOT add inline comments on that line;
// get_file_data() reads everything after the colon as the value, including comments.
// NOTE: WordPress 7.0 (April 2026) raises the minimum PHP to 7.4. Update "Requires PHP"
// to at least 7.4 in your plugin header once you target WP 7.0+ sites exclusively.

defined( 'ABSPATH' ) || exit;

define( 'MYPLUGIN_VERSION', '1.0.0' );
define( 'MYPLUGIN_PATH',    plugin_dir_path( __FILE__ ) );
define( 'MYPLUGIN_URL',     plugin_dir_url( __FILE__ ) );

// Bail early if Elementor is not active (when Elementor features are used)
add_action( 'plugins_loaded', function() {
    if ( ! did_action( 'elementor/loaded' ) ) {
        add_action( 'admin_notices', function() {
            echo '<div class="notice notice-warning"><p>'
                . esc_html__( 'My Plugin requires Elementor to be active.', 'myplugin' )
                . '</p></div>';
        } );
        return;
    }
    require MYPLUGIN_PATH . 'includes/class-myplugin.php';
    MyPlugin::instance()->init();
} );

// ✅ WooCommerce HPOS compatibility declaration.
// HPOS is enabled by default for new WooCommerce stores since v8.2 (October 2023).
// Without this declaration, WooCommerce shows a blocking admin warning.
// Remove this block if your plugin does not interact with WooCommerce at all.
add_action( 'before_woocommerce_init', function() {
    if ( class_exists( \Automattic\WooCommerce\Utilities\FeaturesUtil::class ) ) {
        \Automattic\WooCommerce\Utilities\FeaturesUtil::declare_compatibility(
            'custom_order_tables', // HPOS feature identifier — do not change this string
            __FILE__,
            true  // true = compatible; false = not compatible
        );
    }
} );
```

### Singleton init class pattern:
```php
<?php
// includes/class-myplugin.php
defined( 'ABSPATH' ) || exit;

class MyPlugin {
    private static ?self $instance = null;

    public static function instance(): self {
        if ( null === self::$instance ) {
            self::$instance = new self();
        }
        return self::$instance;
    }

    public function init(): void {
        require MYPLUGIN_PATH . 'includes/class-myplugin-hooks.php';
        require MYPLUGIN_PATH . 'includes/class-myplugin-assets.php';
        ( new MyPlugin_Hooks() )->register();
        ( new MyPlugin_Assets() )->register();
    }
}
```

---

## Child Theme Scaffolding

Use when customizing an existing theme without a dedicated plugin.

### File structure
```
/wp-content/themes/mytheme-child/
├── style.css          ← Required header; do NOT import parent here
├── functions.php      ← Enqueue parent + child styles; all hooks go here
└── screenshot.png     ← 1200×900px recommended
```

### style.css header:
```css
/*
 * Theme Name:   My Theme Child
 * Template:     parent-theme-folder-name
 * Version:      1.0.0
 * Text Domain:  mytheme-child
 */
```

### functions.php — correct enqueue pattern:
```php
<?php
defined( 'ABSPATH' ) || exit;

/**
 * Enqueue parent and child theme stylesheets.
 * Never use @import in style.css — always use wp_enqueue_style().
 */
add_action( 'wp_enqueue_scripts', 'mytheme_child_enqueue_styles' );
function mytheme_child_enqueue_styles(): void {
    $parent_style = 'parent-theme-style'; // match parent theme's registered handle

    wp_enqueue_style(
        $parent_style,
        get_template_directory_uri() . '/style.css',
        [],
        wp_get_theme( get_template() )->get( 'Version' )
    );

    wp_enqueue_style(
        'mytheme-child-style',
        get_stylesheet_uri(),
        [ $parent_style ],
        wp_get_theme()->get( 'Version' )
    );
}
```

---

## Custom Post Type Registration

```php
// Placement: dedicated plugin — never in functions.php
// Always register on 'init', never on 'after_setup_theme' or earlier

add_action( 'init', 'myplugin_register_post_types' );
function myplugin_register_post_types(): void {

    register_post_type( 'myplugin_item', [
        'labels' => [
            'name'               => _x( 'Items', 'post type general name', 'myplugin' ),
            'singular_name'      => _x( 'Item', 'post type singular name', 'myplugin' ),
            'add_new_item'       => __( 'Add New Item', 'myplugin' ),
            'edit_item'          => __( 'Edit Item', 'myplugin' ),
            'view_item'          => __( 'View Item', 'myplugin' ),
            'search_items'       => __( 'Search Items', 'myplugin' ),
            'not_found'          => __( 'No items found.', 'myplugin' ),
            'not_found_in_trash' => __( 'No items found in Trash.', 'myplugin' ),
        ],
        'public'             => true,
        'has_archive'        => true,
        'show_in_rest'       => true,   // required for Gutenberg + Elementor Loop Grid
        'supports'           => [ 'title', 'editor', 'thumbnail', 'custom-fields' ],
        'menu_icon'          => 'dashicons-portfolio',
        'rewrite'            => [ 'slug' => 'items', 'with_front' => false ],
    ] );

    // Register associated taxonomy
    register_taxonomy( 'myplugin_category', 'myplugin_item', [
        'labels'            => [
            'name'          => _x( 'Item Categories', 'taxonomy general name', 'myplugin' ),
            'singular_name' => _x( 'Item Category', 'taxonomy singular name', 'myplugin' ),
        ],
        'hierarchical'  => true,
        'public'        => true,
        'show_in_rest'  => true,        // required for Elementor Loop Grid filtering
        'rewrite'       => [ 'slug' => 'item-category' ],
    ] );
}

// ⚠️ FLUSH REWRITE RULES: only on activation — NEVER on 'init' (too expensive).
// register_activation_hook( __FILE__, function() {
//     myplugin_register_post_types();   // register CPTs so rules exist before flush
//     flush_rewrite_rules();
// } );
// Alternatively: Dashboard → Settings → Permalinks → Save Changes (manual one-time flush).
```

---

## AJAX Handler Pattern

Complete PHP + JS pattern for secure WordPress AJAX:

```php
// Placement: includes/class-myplugin-hooks.php

// Register handlers — add wp_ajax_nopriv_ only if action is public-facing
add_action( 'wp_ajax_myplugin_action',        'myplugin_ajax_handler' );
add_action( 'wp_ajax_nopriv_myplugin_action', 'myplugin_ajax_handler' );

/**
 * All wp_send_json_* functions call wp_die() internally.
 * Use check_ajax_referer() (not wp_verify_nonce) in AJAX handlers —
 * it is the idiomatic WP pattern, fires the check_ajax_referer action
 * hook for security logging, and correctly dies on failure.
 */
function myplugin_ajax_handler(): void {
    // 1. Verify nonce — dies automatically with -1 output and 403 header on failure
    check_ajax_referer( 'myplugin_ajax', 'nonce' );

    // 2. Check capability — nonces confirm request origin, NOT permission
    if ( ! current_user_can( 'read' ) ) {
        wp_send_json_error( [ 'message' => __( 'Insufficient permissions.', 'myplugin' ) ], 403 );
        return;
    }

    // 3. Sanitize and validate input
    $item_id = absint( $_POST['item_id'] ?? 0 );
    if ( $item_id <= 0 ) {
        wp_send_json_error( [ 'message' => __( 'Invalid item ID.', 'myplugin' ) ], 400 );
        return;
    }

    // 4. Process and respond — wp_send_json_success calls wp_die() internally
    $data = myplugin_get_cached_data( $item_id );
    wp_send_json_success( $data );
}
```

```js
// JS side — pairs with the wp_add_inline_script data (see js-css-standards.md)
const fetchItem = async ( itemId ) => {
  const body = new FormData();
  body.append( 'action', 'myplugin_action' );
  body.append( 'nonce',   myPluginData.nonce );
  body.append( 'item_id', itemId );

  try {
    const res  = await fetch( myPluginData.ajaxUrl, { method: 'POST', body } );
    const json = await res.json();

    if ( ! json.success ) {
      console.error( 'myplugin AJAX error:', json.data?.message );
      return null;
    }

    return json.data;
  } catch ( err ) {
    console.error( 'myplugin fetch failed:', err );
    return null;
  }
};
```

---

## Elementor Widget Registration Reminder

When registering an Elementor widget from a plugin (via `elementor/widgets/register`),
**every widget class must implement these two methods** — they are required for V4
compatibility and output caching. Forgetting them in any widget (including those scaffolded
from this file) will cause rendering issues and PHP warnings in future Elementor versions.

```php
// ✅ REQUIRED on EVERY new widget — removes the redundant inner wrapper div.
// Elementor 3.26+. Without this, the deprecated .elementor-widget-container wrapper
// is still rendered, breaking Optimized Markup mode and V4 Atomic elements.
// Return true ONLY if your render() output physically requires that wrapper.
// Source: developers.elementor.com/docs/widgets/widget-inner-wrapper/
public function has_widget_inner_wrapper(): bool {
    return false;
}

// ✅ REQUIRED on EVERY widget — controls Elementor output caching.
// Return false  → Elementor may cache the rendered HTML (safe for static/non-user-specific output).
// Return true   → Elementor skips caching (required if output varies per user, session, or time).
// Source: developers.elementor.com/docs/widgets/widget-output-caching/
protected function is_dynamic_content(): bool {
    return false; // change to true if output is user/session/time-specific
}
```

> **See `elementor-patterns.md` for the complete widget boilerplate**, and the relevant
> `widget-*.md` file for control patterns matching Elementor's native widgets.
