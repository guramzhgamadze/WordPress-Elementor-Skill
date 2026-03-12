# JavaScript & CSS Standards

## JavaScript Standards

```js
// ✅ IIFE + ES6+ — no jQuery unless forced by a WP core dependency
( () => {
  'use strict';

  const init = ( scope = document ) => {
    scope.querySelectorAll( '.myplugin__trigger' ).forEach( el => {
      el.addEventListener( 'click', handleClick );
    } );
  };

  const handleClick = ( e ) => {
    e.preventDefault();
    const trigger = e.currentTarget;
    const target  = document.getElementById( trigger.getAttribute( 'aria-controls' ) );
    const isOpen  = trigger.getAttribute( 'aria-expanded' ) === 'true';

    trigger.setAttribute( 'aria-expanded', String( ! isOpen ) );
    // ✅ CSS class controls visibility — NOT aria-hidden (see offcanvas-ui.md for full pattern)
    target?.classList.toggle( 'myplugin__panel--open', ! isOpen );
    target?.classList.toggle( 'myplugin__panel--hidden', isOpen );
  };

  // ✅ ONLY correct pattern per official Elementor Developer docs:
  // Wait for 'elementor/frontend/init' before registering addAction.
  // DO NOT use if(window.elementorFrontend) — if elementorFrontend already
  // exists, all element_ready events have already fired and addAction silently
  // misses every widget already on the page.
  //
  // ⚠️ CRITICAL — this script must NOT use strategy:'defer' if it relies on this event.
  // 'elementor/frontend/init' is a CustomEvent fired synchronously during footer execution.
  // A deferred script may execute AFTER this event fires, causing a silent miss.
  // Use in_footer:true WITHOUT defer, and declare 'elementor-frontend' as a dependency.
  window.addEventListener( 'elementor/frontend/init', () => {
    window.elementorFrontend.hooks.addAction(
      'frontend/element_ready/myplugin-widget.default',
      ( $scope ) => init( $scope[0] )
    );
  } );
} )();
```

---

## Enqueue API — WP 6.3+ defer/async

```php
add_action( 'wp_enqueue_scripts', 'myplugin_enqueue_assets' );
function myplugin_enqueue_assets(): void {
    wp_enqueue_style(
        'myplugin-css',
        MYPLUGIN_URL . 'assets/css/myplugin.css',
        [],
        MYPLUGIN_VERSION
    );

    wp_enqueue_script(
        'myplugin-js',
        MYPLUGIN_URL . 'assets/js/myplugin.js',
        [],                    // no jQuery unless genuinely required
        MYPLUGIN_VERSION,
        // Both keys are required: 'strategy' controls HOW the script loads (defer/async);
        // 'in_footer' controls WHERE it is placed (head vs footer). They are independent.
        // Omitting 'in_footer' => true places the script in <head> even with defer.
        // See: make.wordpress.org/core/2023/07/14/registering-scripts-with-async-and-defer-attributes-in-wordpress-6-3/
        [ 'strategy' => 'defer', 'in_footer' => true ]
    );

    // ✅ Pass PHP data to JS — use wp_add_inline_script (best practice since WP 4.5+).
    // wp_localize_script() was designed for i18n string localisation only.
    // wp_add_inline_script() is the correct tool for passing arbitrary data to JS.
    //
    // ⚠️ wp_json_encode() returns false|string — guard against encoding failure.
    // "const myPluginData = ;" is a JS syntax error that breaks the whole page.
    $script_data = wp_json_encode( [
        'ajaxUrl' => admin_url( 'admin-ajax.php' ),
        'nonce'   => wp_create_nonce( 'myplugin_ajax' ),
        'i18n'    => [ 'error' => __( 'Something went wrong.', 'myplugin' ) ],
    ] );
    if ( $script_data ) {
        wp_add_inline_script(
            'myplugin-js',
            'const myPluginData = ' . $script_data . ';',
            'before'  // inject BEFORE the script so it's available on load
        );
    }
}
```

---

## CSS Standards (BEM + Scoped Design Tokens)

```css
/* ─── BEM Naming ──────────────────────────────────────────── */
.myplugin {}                          /* Block */
.myplugin__header {}                  /* Element */
.myplugin__trigger {}
.myplugin__trigger--active {}         /* Modifier */

/* ─── Scoped design tokens — never pollute :root ─────────── */
.myplugin {
  --mp-color-primary:   #1a1a2e;
  --mp-color-accent:    #e94560;
  --mp-spacing-sm:      0.5rem;   /* 8px  */
  --mp-spacing-md:      1rem;     /* 16px */
  --mp-spacing-lg:      2rem;     /* 32px */
  --mp-radius:          8px;
  --mp-transition:      0.2s ease;
}

/* ─── Fluid typography — replaces breakpoint font-size overrides ── */
.myplugin__heading {
  font-size: clamp(1.25rem, 2vw + 1rem, 2.5rem);
  line-height: 1.2;
}

/* ─── Accessible focus styles (never use outline: none without replacement) */
.myplugin__trigger:focus-visible {
  outline: 3px solid var(--mp-color-accent);
  outline-offset: 2px;
}

/* ─── Safe Elementor overrides — CRITICAL selector rules ─────────────────
   ⚠️ NEVER target .elementor-widget-container directly.
   This inner wrapper div is REMOVED when Elementor's "Optimized Markup"
   experiment is active (introduced 3.25 alpha; still opt-in as of 3.35.7
   but users can enable it). Does NOT exist at all in Elementor V4 elements.

   ✅ CORRECT patterns — always use the widget root or your own BEM classes: */
.elementor-widget-myplugin-widget { /* targets the widget root — always present */ }
.myplugin-widget { /* your own BEM root class — always safe */ }

/* ✅ {{WRAPPER}} in Elementor controls maps to .elementor-widget-myplugin-widget
   in CSS — use this for control selectors in register_controls(). */

/* ─── 8pt spacing system ───────────────────────────────────── */
.myplugin__card {
  padding:          var(--mp-spacing-md);
  margin-block-end: var(--mp-spacing-lg);
  border-radius:    var(--mp-radius);
  transition:       transform var(--mp-transition);
}
```
