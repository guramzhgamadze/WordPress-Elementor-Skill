# Widget Boilerplate — Read More

> **When to use this file:** Load whenever building a widget that inserts WordPress's `<!--more-->` tag for archive excerpts.
> Verified against `elementor/includes/widgets/read-more.php` (Elementor 3.35+).
> This widget has **no controls** — it only renders the WordPress `more` tag.

---

```php
protected function register_controls(): void {
    // No controls — this widget only inserts the WordPress <!--more--> tag
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

// ✅ MUST be true — this widget mutates the global $more variable which controls how
// the_content() behaves for the current request. Output caching would freeze the $more=0
// side-effect at cache-build time and not re-apply it on cached responses, causing full
// post content to appear instead of truncating at the <!--more--> marker.
// Source: developers.elementor.com/docs/widgets/widget-output-caching/
protected function is_dynamic_content(): bool {
    return true;
}

protected function render(): void {
    // ✅ The <!--more--> tag splits post content on ARCHIVE/LOOP pages (index, category, tag).
    // Setting global $more = 0 tells the_content() to stop output at the <!--more--> marker.
    // On singular pages the_content() always shows full content — $more has no effect there.
    // Do NOT add an is_singular() check here: the widget must run on archive pages, which
    // are the pages where <!--more--> is actually respected.
    global $more;
    $more = 0; // phpcs:ignore WordPress.WP.GlobalVariablesOverride
}
```

---



**content_template() skeleton:**

```php
protected function content_template(): void {
    // Read More has no frontend preview — it only affects archive excerpt splitting.
}
```
