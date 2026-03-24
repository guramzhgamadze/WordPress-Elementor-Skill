# Off-Canvas Filter UI — Full Pattern

```php
// Placement: Elementor Custom Code → wp_body_open
// CSS must hide the panel by default via class, NOT aria-hidden on the element itself.
// aria-hidden="true" on a role="dialog" with aria-modal conflicts with ARIA spec
// and causes inconsistent screen reader behaviour. Use CSS visibility instead.
?>
<div id="myplugin-offcanvas"
     class="myplugin-offcanvas myplugin-offcanvas--hidden"
     role="dialog"
     aria-modal="true"
     aria-labelledby="myplugin-offcanvas-title">

    <div class="myplugin-offcanvas__header">
        <h2 id="myplugin-offcanvas-title" class="myplugin-offcanvas__title">
            <?php esc_html_e( 'Filter Results', 'myplugin' ); ?>
        </h2>
        <button class="myplugin-offcanvas__close"
                aria-label="<?php esc_attr_e( 'Close filters', 'myplugin' ); ?>">
            &#10005;
        </button>
    </div>

    <div class="myplugin-offcanvas__body">
        <!-- filter controls here -->
    </div>
</div>

<button class="myplugin-offcanvas__trigger"
        aria-controls="myplugin-offcanvas"
        aria-expanded="false">
    <?php esc_html_e( 'Open Filters', 'myplugin' ); ?>
</button>

<!--
    ✅ Backdrop overlay — click to close the panel.
    Placed AFTER the panel (not inside it) so it sits behind the panel in stacking context.
    Controlled by the same JS open/close functions via the 'myplugin-offcanvas--hidden' class.
-->
<div class="myplugin-offcanvas__backdrop" aria-hidden="true"></div>
```

```css
/* ⚠️ visibility:hidden alone does NOT block keyboard Tab into focusable children.
   The `inert` attribute (toggled by JS below) handles that — baseline supported
   in all major browsers since April 2023 (95%+ coverage). */

/* ✅ transition MUST be on the BASE element — not only on the --hidden modifier.
   If transition is placed only on --hidden, the open animation (class removal)
   runs without a transition because the rule disappears with the class. */
.myplugin-offcanvas {
  position: fixed;   /* viewport-relative overlay — removed from document flow */
  top: 0;
  left: 0;
  width: 320px;

  /* ✅ Use 100dvh (dynamic viewport height) NOT 100vh.
     On iOS Safari, 100vh is the layout viewport height which does NOT account for
     the browser chrome. 100dvh reflects the actual visible viewport height.
     Supported: Chrome 108+, Safari 15.4+, Firefox 116+. */
  height: 100vh;     /* fallback for browsers without dvh support */
  height: 100dvh;    /* ✅ correct — accounts for mobile browser chrome */

  z-index: 9999;
  overflow-y: auto;

  /* ⚠️ ELEMENTOR TRANSFORM TRAP: position:fixed is broken when a parent has a
     non-none CSS transform (Elementor applies transforms to sections/columns).
     The wp_body_open placement ensures this element is a direct child of <body>,
     which has no transform. Never move this markup inside an Elementor widget. */

  visibility: visible;
  pointer-events: auto;
  transform: translateX(0);
  transition: transform 0.3s ease, visibility 0.3s ease; /* ← must be here, not on --hidden */
}
.myplugin-offcanvas--hidden {
  visibility: hidden;
  pointer-events: none;
  transform: translateX(-100%);
  /* Do NOT repeat transition here — the base class handles both directions */
}

/* ── Backdrop overlay ───────────────────────────────────── */
.myplugin-offcanvas__backdrop {
  position: fixed;
  inset: 0;                     /* covers full viewport */
  background: rgba( 0, 0, 0, 0.5 );
  z-index: 9998;                /* one below the panel (9999) */
  opacity: 1;
  transition: opacity 0.3s ease, visibility 0.3s ease;
  visibility: visible;
}
/* Hidden state — toggled by JS alongside the panel */
.myplugin-offcanvas__backdrop.myplugin-offcanvas__backdrop--hidden {
  opacity: 0;
  visibility: hidden;
  pointer-events: none;
}
```

```js
// Off-canvas JS — CSS class toggle + inert + ARIA expanded + focus trap + scroll lock
( () => {
  'use strict';

  const panel   = document.getElementById( 'myplugin-offcanvas' );
  const trigger = document.querySelector( '.myplugin-offcanvas__trigger' );
  const close   = panel?.querySelector( '.myplugin-offcanvas__close' );

  if ( ! panel || ! trigger ) return;

  const HIDDEN_CLASS = 'myplugin-offcanvas--hidden';
  // ✅ a[href] not [href] — only <a> elements with an href are natively keyboard focusable
  const focusable = 'a[href], button, input, select, textarea, [tabindex]:not([tabindex="-1"])';

  const backdrop = document.querySelector( '.myplugin-offcanvas__backdrop' );

  const open = () => {
    panel.classList.remove( HIDDEN_CLASS );
    panel.inert = false;                        // ✅ re-enable keyboard + AT access
    trigger.setAttribute( 'aria-expanded', 'true' );
    backdrop?.classList.remove( 'myplugin-offcanvas__backdrop--hidden' );

    // ✅ iOS Safari scroll lock — overflow:hidden on body does NOT prevent background
    // scroll on iOS Safari. Store scroll position, fix body in place, restore on close.
    const scrollY = window.scrollY;
    document.body.style.position   = 'fixed';
    document.body.style.top        = `-${ scrollY }px`;
    document.body.style.width      = '100%';
    document.body.dataset.scrollY  = scrollY;

    // ✅ Respect prefers-reduced-motion — when animations are disabled, don't wait 300ms
    const delay = window.matchMedia( '(prefers-reduced-motion: reduce)' ).matches ? 0 : 300;
    setTimeout( () => panel.querySelector( focusable )?.focus(), delay );
  };

  const closePanel = () => {
    panel.classList.add( HIDDEN_CLASS );
    panel.inert = true;                         // ✅ block Tab focus into hidden panel
    trigger.setAttribute( 'aria-expanded', 'false' );

    const scrollY = parseInt( document.body.dataset.scrollY || '0', 10 );
    document.body.style.position  = '';
    document.body.style.top       = '';
    document.body.style.width     = '';
    // ✅ Restore scroll BEFORE deleting dataset.scrollY — prevents data loss on rapid
    // open/close cycles where a second panel.open() could read a deleted value.
    window.scrollTo( 0, scrollY );
    delete document.body.dataset.scrollY;

    trigger.focus();
    backdrop?.classList.add( 'myplugin-offcanvas__backdrop--hidden' );
  };

  // Initialize panel and backdrop as hidden on load
  panel.inert = true;
  backdrop?.classList.add( 'myplugin-offcanvas__backdrop--hidden' );

  // Focus trap — Tab cycles within open panel
  // ✅ querySelectorAll runs fresh on each keydown so dynamically injected controls
  // (AJAX-loaded filter options) are always included in the trap
  panel.addEventListener( 'keydown', ( e ) => {
    if ( e.key !== 'Tab' ) return;
    const els = [ ...panel.querySelectorAll( focusable ) ];
    if ( els.length === 0 ) return;
    const first = els[0];
    const last  = els[ els.length - 1 ];
    if ( e.shiftKey && document.activeElement === first ) {
      e.preventDefault(); last.focus();
    } else if ( ! e.shiftKey && document.activeElement === last ) {
      e.preventDefault(); first.focus();
    }
  } );

  panel.addEventListener( 'keydown', ( e ) => { if ( e.key === 'Escape' ) closePanel(); } );
  trigger.addEventListener( 'click', open );
  close?.addEventListener( 'click', closePanel );
  // ✅ Backdrop click closes the panel — matches native dialog behaviour
  backdrop?.addEventListener( 'click', closePanel );
} )();
```
