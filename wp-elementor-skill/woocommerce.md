# WooCommerce Integration

## ⚠️ HPOS Compatibility — Required for All WooCommerce Plugins (WC 8.2+)

WooCommerce High-Performance Order Storage (HPOS) uses dedicated order tables instead
of `wp_posts`/`wp_postmeta`. It is **enabled by default for all new stores since WC 8.2
(October 2023)**. Any plugin that declares `WC tested up to` in its header and touches
orders MUST declare HPOS compatibility or WooCommerce will block the feature with admin
warnings.

> ⚠️ **WooCommerce 10.7 (scheduled April 14, 2026) — HPOS "Sync on Read" disabled by default:**
> Starting in WooCommerce 10.7, the HPOS **"sync on read"** mechanism is disabled by default.
> **What this means:** "Sync on read" was a safety net that detected when order data was written
> directly to the legacy `wp_posts`/`wp_postmeta` tables (bypassing WooCommerce's API) and
> pulled those changes back into HPOS on the next read. With this disabled, any plugin or custom
> code that writes order data via `wp_update_post()`, `update_post_meta()`, or direct SQL will
> **silently produce stale data** in HPOS.
>
> **Action required:** Audit all code that touches order data. Migrate to HPOS-native API.
> ```php
> // Test the new 10.7 default (sync-on-read disabled) before upgrading:
> add_filter( 'woocommerce_hpos_enable_sync_on_read', '__return_false' );
>
> // Re-enable sync-on-read after upgrading to 10.7 (temporary bridge only):
> add_filter( 'woocommerce_hpos_enable_sync_on_read', '__return_true' );
> ```
> Re-enabling should be treated as a temporary measure only — sync-on-read may be fully
> removed in a future WooCommerce version.
> Source: developer.woocommerce.com/2026/02/16/hpos-sync-on-read-to-be-disabled-by-default-in-woocommerce-10-7/

```php
// HPOS compatibility declaration — add to your main plugin file (see scaffolding.md)
add_action( 'before_woocommerce_init', function() {
    if ( class_exists( \Automattic\WooCommerce\Utilities\FeaturesUtil::class ) ) {
        \Automattic\WooCommerce\Utilities\FeaturesUtil::declare_compatibility(
            'custom_order_tables',
            __FILE__,
            true
        );
    }
} );
```

---

## HPOS Order API Rules

**Always use these — never `get_post()` / `update_post_meta()` for orders:**

```php
// ✅ Retrieve an order
$order = wc_get_order( $order_id );   // returns WC_Order, works with both legacy and HPOS

// ✅ Read order meta
$value = $order->get_meta( '_my_meta_key' );

// ✅ Write order meta
$order->update_meta_data( '_my_meta_key', $value );
$order->save();

// ❌ NEVER use these for orders when HPOS is active:
// $order = get_post( $order_id );           // returns WP_Post — wrong type, may be empty
// update_post_meta( $order_id, ... );       // writes to postmeta, bypasses HPOS tables
// get_post_meta( $order_id, '_key', true ); // reads from postmeta, stale if HPOS is authoritative
```

---

## Preferred: Elementor Loop Grid (No Template Files Touched)

```php
// Use elementor-patterns.md Loop Grid pattern — set post_type to 'product'
// Apply tax_query for product_cat / product_tag as needed
// This is always safer than overriding template files
add_action( 'elementor/query/myplugin_products_query', function( \WP_Query $query ) {
    $query->set( 'post_type',      'product' );
    $query->set( 'posts_per_page', 6 );
    $query->set( 'no_found_rows',  true );
} );
```

---

## Hook-Based Additions (Safer Than Template Override)

```php
add_action( 'woocommerce_after_add_to_cart_button', 'myplugin_after_cart_button' );
function myplugin_after_cart_button(): void {
    // ✅ Use wc_get_product() — NOT global $product.
    // global $product works on standard single-product pages but returns null or the
    // wrong product inside Elementor popups, Quick View overlays, and AJAX-rendered
    // product templates. wc_get_product( get_the_ID() ) is reliable in all contexts.
    $product = wc_get_product( get_the_ID() );
    if ( ! $product instanceof WC_Product ) return;

    echo '<p class="myplugin-product__note">'
        . esc_html__( 'Free shipping on this item.', 'myplugin' )
        . '</p>';
}
```

---

## Template Override (Last Resort — Always Document Why)

```
Copy from: /wp-content/plugins/woocommerce/templates/
Place at:  /wp-content/themes/mytheme-child/woocommerce/
Maintain:  After every WooCommerce update, check WooCommerce → Status → System Status
           for "Outdated template files" warnings. WooCommerce compares the version
           declared in each template's @version comment against the bundled version.
           Outdated template overrides cause silently broken checkout flows.
```
