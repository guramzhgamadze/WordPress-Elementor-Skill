---
name: wordpress-elementor-dev
description: >
  Expert WordPress and Elementor Pro development skill. Use this whenever the user
  asks about WordPress theme development, Elementor Pro customization, plugin architecture,
  custom post types, hooks/filters, Loop Grids, Dynamic Tags, Theme Builder templates,
  off-canvas UIs, custom CSS/JS injection, performance optimization, or any PHP/JS/CSS
  code targeting a WordPress or Elementor environment. Trigger even for general questions
  like "how do I add a custom field in Elementor", "what hook should I use for X in
  WordPress", "how do I override a WooCommerce template", or "how do I register a
  custom widget" — do not wait for the user to explicitly say "use the skill."
  Also trigger for ACF integration, REST API endpoints, child theme setup, WooCommerce
  customization, WPML/multilingual setups, and any debugging of WordPress/Elementor issues.
---

# WordPress & Elementor Pro — Skill Router

A complete procedural guide for producing production-grade WordPress and Elementor Pro
code. This file is the **router** — read it first, then load the relevant sub-file(s)
for the task at hand.

---

## Sub-file Map — Read the Right File for Each Task

| Task type | Read this file |
|---|---|
| Plugin scaffold, child theme, code placement, CPT, AJAX handler | **scaffolding.md** |
| PHP standards, sanitization, escaping, nonces, WP_Error, transients, password hashing | **php-standards.md** |
| JavaScript standards, enqueue API, defer/async, wp_add_inline_script | **js-css-standards.md** |
| CSS standards, BEM, design tokens, Elementor CSS selectors | **js-css-standards.md** |
| Elementor custom widget, Dynamic Tags, Loop Grid, Form actions, Theme Builder conditions | **elementor-patterns.md** |
| WooCommerce HPOS, order API, template overrides, Loop Grid for products | **woocommerce.md** |
| REST API endpoints, schema, permission callbacks | **rest-api.md** |
| Off-canvas UI, off-canvas accessibility, focus trap | **offcanvas-ui.md** |
| Performance checklists (frontend + backend), speculative loading, IE conditional comments | **performance.md** |
| Accessibility checklist, WCAG 2.2 AA, ARIA patterns | **performance.md** |

### Widget Boilerplates — Load when building a widget of that type

| Widget type | Read this file |
|---|---|
| Button / CTA with icon and link | **widget-button.md** |
| Container / layout wrapper / section / card shell | **widget-container.md** |
| Image with caption, link, lightbox | **widget-image.md** |
| Heading / title / HTML tag selector | **widget-heading.md** |
| Rich text / WYSIWYG body content | **widget-text-editor.md** |
| Video embed (YouTube, Vimeo, self-hosted) | **widget-video.md** |
| Widget that renders a saved Elementor template by ID | **widget-elementor-template.md** (concise) · **widget-template-elementor.md** (extended — SELECT2, Dynamic Tags, CSS timing) |
| Widget whose markup lives in a separate PHP template file | **widget-php-template.md** (concise) · **widget-template-php.md** (extended — Strategies A/B/C, ob_start, path traversal) |
| Divider / horizontal rule with optional text or icon | **widget-divider.md** |
| Spacer / vertical gap | **widget-spacer.md** |
| Single standalone icon with optional link | **widget-icon.md** |
| Icon + title + description box | **widget-icon-box.md** |
| Image + title + description box | **widget-image-box.md** |
| Image grid / gallery | **widget-image-gallery.md** |
| Image slider / carousel | **widget-image-carousel.md** |
| Bullet list with icons per item | **widget-icon-list.md** |
| Animated number counter | **widget-counter.md** |
| Percentage progress bar | **widget-progress.md** |
| Customer quote / testimonial | **widget-testimonial.md** |
| Tabbed content panels | **widget-tabs.md** |
| Accordion — one panel open at a time | **widget-accordion.md** |
| Toggle — multiple panels open simultaneously | **widget-toggle.md** |
| Social media icon links row | **widget-social-icons.md** |
| Colored alert / notice box | **widget-alert.md** |
| Audio player (SoundCloud / self-hosted) | **widget-audio.md** |
| WordPress shortcode output | **widget-shortcode.md** |
| Raw custom HTML / JS / CSS embed | **widget-html.md** |
| Named anchor for in-page navigation | **widget-menu-anchor.md** |
| WordPress registered sidebar output | **widget-sidebar.md** |
| WordPress <!--more--> read more tag | **widget-read-more.md** |
| Google Maps embed | **widget-google-maps.md** |
| Decorative star rating display | **widget-star-rating.md** |
| Schema-ready structured rating | **widget-rating.md** |

**Always read the relevant sub-file before writing code.** For tasks that span multiple
areas (e.g. a WooCommerce widget with custom REST endpoint), read all relevant sub-files.
For a widget task, read BOTH the widget boilerplate file AND **elementor-patterns.md**.

---

## 0. Golden Rules (Never Violate)

These override everything in all sub-files:

1. **Native APIs first** — WordPress core hook before plugin; Elementor API before template override.
2. **Sanitize in, escape out** — Every input sanitized. Every output escaped. No exceptions.
3. **Prefix everything** — All functions, classes, constants, hooks, and CSS classes use a project-specific prefix.
4. **State your placement** — Every code response must declare exactly where the code lives.
5. **No over-clarifying** — Only ask a clarification question if the missing info would materially change the code output. Otherwise, state your assumption and proceed.
6. **NEVER hardcode visual settings in widgets** — Every visual property (colors, fonts, sizes,
   spacing, backgrounds, borders, shadows, alignment) MUST be exposed as a standard Elementor
   control in the editor panel. Users control appearance via the toolbar — not by editing code.
   See §5 "Mandatory Widget Controls" below for the required controls checklist.

---

## 1. Default Assumptions

Quickly assess — **only ask if the answer would change the code**:

| Info needed | Ask only if... |
|---|---|
| Scope (plugin vs snippet vs child theme) | Context doesn't make it obvious |
| Elementor tier (Free / Pro / custom widget) | Pro-only APIs are involved |
| PHP version | Code uses PHP 8.3+ features like typed class constants, or 8.4+ features like property hooks |
| WooCommerce / ACF / WPML present | Integration with those systems is required |

**Default assumptions when not stated:** WordPress 6.9.4+ (WP 7.0 scheduled April 9, 2026),
PHP 8.3+ (officially recommended by wordpress.org/about/requirements/; 8.4 and 8.5 = beta
support label; 8.2 is fully compatible but no longer the recommended default),
Elementor 4.0.0+ (free and Pro both version 4.0.0, released March 30, 2026). V3 Widget_Base API remains fully supported.
**Note:** Elementor core and Elementor Pro are separate plugins with independent version numbers
**Note:** Elementor free and Pro both released **4.0.0 on March 30, 2026**.
History: 3.35.8 (March 23) was a security release; 3.35.9 (March 25) fixed an
AI-generated image insertion bug; 4.0.0 (March 30) is the current stable release.
Always check both free and Pro when diagnosing compatibility issues. No multisite assumed.

> 🔜 **WordPress 7.0 (April 9, 2026):** PHP 7.2 and 7.3 will no longer be supported. The new
> minimum PHP version is 7.4.0. Sites still on PHP 7.2/7.3 will remain pinned to WP 6.9 and
> will not receive the 7.0 update. The minimum **recommended** version remains PHP 8.3.
> **Database:** No new minimum is enforced in WP 7.0 — the DB minimum was last bumped in
> WP 6.5 (MySQL 5.5.5+). The official `wordpress.org/about/requirements/` page now lists
> **MariaDB 10.6+ or MySQL 8.0+** as the primary recommended versions, and PHP 8.3+ as
> the recommended PHP version.
> Source: make.wordpress.org/core/2026/01/09/dropping-support-for-php-7-2-and-7-3/,
> wordpress.org/about/requirements/

> 🤖 **WordPress 7.0 — New Developer APIs (WP AI Client + Connectors UI + Abilities API):**
> All opt-in, no breaking changes. See quick summary below — these do not affect most
> plugin/Elementor work in WP 7.0.
>
> - **WP AI Client:** `wp_ai_client_prompt($prompt)->generate_text()` — provider-agnostic
>   PHP + JS AI API. Use `function_exists('wp_ai_client_prompt')` to guard.
> - **Connectors UI:** Settings → Connectors admin page for managing AI provider credentials.
>   Hook: `connections-wp-admin-init`. ⚠️ **These names are PRE-STABLE and subject to change**
>   before the April 9, 2026 final release — guard all usage with `function_exists()` /
>   `did_action()` checks and do not ship production code depending on them until WP 7.0 stable.
> - **Abilities API:** PHP side in WP 6.9 (`wp_register_ability()`); JS counterpart in WP 7.0.
>   `'meta' => ['show_in_rest' => true]` to expose via REST.
> - **Iframed Editor (PUNTED to WP 7.1):** No action required for WP 7.0. Prepare for 7.1
>   with `"apiVersion": 3` in block.json.
> - **Real-Time Collaboration (RTC):** WP 7.0 introduces simultaneous multi-author block
>   editing. Technically: HTTP polling sync provider (not WebRTC), CRDT update data stored
>   persistently in `wp_post_meta` on a new internal post type **`wp_sync_storage`**. Updates
>   are batched and periodically compacted. RTC default (opt-in vs opt-out) finalized around
>   RC2 — per official March 2026 developer news. The `WP_ALLOW_COLLABORATION` wp-config
>   constant lets hosts swap the transport provider. Expected to become opt-out in a future
>   release once broader plugin coverage is confirmed.
>   **Client-side Media Processing was reverted from 7.0 in Beta 6** (package size ~13 MB,
>   Chromium-only, OOM crashes) — punted to WP 7.1. No action needed for 7.0.
>   **Plugin impact:** If your plugin queries `wp_post_meta` by post type or iterates all
>   posts/meta, add `'post_type__not_in' => ['wp_sync_storage']` (or equivalent exclusion)
>   to avoid unexpected hits on internal sync data.
>   Source: developer.wordpress.org/news/2026/03/whats-new-for-developers-march-2026/,
>   make.wordpress.org/core/2026/03/19/wordpress-7-0-release-candidate-1-delayed/
>   Source (RC schedule): make.wordpress.org/core/2026/01/09/wordpress-7-0-call-for-volunteers/
> - **WP 7.0 Beta cycle:** Beta 4 released **March 10, 2026** (emergency security fast-follow,
>   same day as WordPress 6.9.2 and 6.9.3). **WordPress 6.9.4** was then released March 11, 2026
>   after the Security Team found not all fixes in 6.9.2 were fully applied — 6.9.4 is the
>   current stable release. Beta 5 released March 12, 2026. **Beta 6 released March 20, 2026**
>   (reverts Client-side Media, includes RTC performance improvements, 4× polling interval).
>   **RC1 released March 24, 2026** (was scheduled March 19 — delayed due to RTC performance
>   concerns and package bloat). **RC2 released March 26, 2026. RC3 scheduled April 2, 2026. Final release: April 9, 2026.**
>   Source: wordpress.org/news/2026/03/wordpress-6-9-3-and-7-0-beta-4/,
>   wordpress.org/news/2026/03/wordpress-7-0-beta-5/,
>   wordpress.org/news/2026/03/wordpress-6-9-4-release/,
>   make.wordpress.org/core/2026/02/12/wordpress-7-0-release-party-schedule/,
>   make.wordpress.org/core/2026/03/19/wordpress-7-0-release-candidate-1-delayed/
>   Source (RC schedule): make.wordpress.org/core/2026/01/09/wordpress-7-0-call-for-volunteers/

> 📌 **PHP support labels (WP 6.9):**
> - PHP 8.0–8.3 = fully compatible. PHP 7.4 = fully compatible.
> - **PHP 8.3 = officially recommended** (raised from beta in July 2025).
> - PHP 8.4 = beta support (WP 6.7+). PHP 8.5 = beta support (WP 6.9+).
> - "Beta support" = actively working toward full compatibility; possible deprecation notices.
> Source: make.wordpress.org/core/handbook/references/php-compatibility-and-wordpress-versions/

> ✅ **Elementor 4.0.0 RELEASED March 30, 2026 (free + Pro simultaneously):**
> - **Elementor free 4.0.0** (March 30, 2026): Atomic Editor status set to **Stable**;
>   enabled by default for all new site installations; Global Style sync (Variables/Classes
>   → legacy Global Styles); self-hosted Video Atomic Element; new onboarding flow.
>   Source: github.com/elementor/elementor/blob/main/changelog.txt (= 4.0.0 - 2026-03-30)
>         · wordpress.org/plugins/elementor/ (version 4.0.0, last updated Mar 30, 2026)
> - **Elementor Pro 4.0.0** (March 30, 2026): Atomic Forms (composable Label/Input/Textarea/
>   Submit atoms); Pro Interactions (scroll-triggered, hover, click; Custom Effect with
>   Scale/Move/Rotate/Skew; breakpoint controls); Component creation & detach for Pro users;
>   custom fonts in typography controls.
>   Source: elementor.com/pro/changelog/ (= 4.0.0 - 2026-03-30)
> - **What 4.0 changes for new sites:** The Atomic Editor is now the default experience.
>   Atomic Elements, Variables, Classes, and Components are enabled on fresh installs.
> - **What 4.0 does NOT change for existing sites:** Updating to 4.0 leaves current sites
>   fully untouched. V3 widgets and V4 Atomic Elements coexist on the same page. Atomic
>   features can be enabled manually: WP Admin → Elementor → Editor → Settings → Atomic Editor.
>   Source: elementor.com/products/website-builder/v4-faq/
> - **V3 `Widget_Base` remains fully supported in 4.0** — all skill code targets V3 and is
>   production-safe on any site. The V4 Atomic Element PHP extension API is now **Stable**
>   in 4.0, but the third-party extension documentation is still being finalized. Third-party
>   plugins should continue using V3 `Widget_Base` until official V4 PHP extension docs ship.
>   Source: developers.elementor.com/elementor-editor-4-0-developers-update/
---

## 2. Architecture Decision Tree

Run through this mentally before writing a single line:

```
Does a WordPress core hook (add_action/add_filter) solve it?
  YES → Use the hook. No plugin needed. Place in child theme functions.php
        or Elementor Custom Code.
  NO  → Does Elementor's PHP/JS API solve it?
          YES → Extend via Elementor hooks, Dynamic Tags, or Widget_Base.
                Register via elementor/widgets/register or elementor/dynamic_tags/register.
          NO  → Is this logic reusable across themes or sites?
                  YES → Scaffold a dedicated plugin (see scaffolding.md).
                  NO  → Child theme functions.php or Elementor Custom Code block.

Is a WooCommerce override needed?
  → Use Elementor Loop Grid + custom query filter BEFORE touching template files.
  → Only override woocommerce/ templates as an absolute last resort.

Is this a REST API endpoint?
  → Always register via register_rest_route() inside a plugin, never in functions.php.
```

---

## 3. Mandatory Output Format

**Every single code response must follow this structure — no exceptions:**

```
📍 PLACEMENT
Exact file path or Elementor hook location.
e.g. /wp-content/plugins/myplugin/includes/class-myplugin-hooks.php
     Elementor → Site Settings → Custom Code → wp_footer

⚙️ REQUIRES
WordPress X.X+ | PHP X.X+ | Elementor Pro X.X+ | ACF X.X+ | WooCommerce X.X+
(list only what the code actually depends on)

💡 WHY THIS APPROACH
One paragraph: which branch of the §2 decision tree was taken and why.

📋 CODE
Complete, commented, deployment-ready code block — no truncation, no omissions.

🔧 INTEGRATION NOTES (include when relevant)
Any manual steps required: flush rewrite rules, set Query ID in Elementor editor,
activate plugin, clear Elementor cache, etc.
```

---

## 4. Quick Reference — Pattern Index

| Task | Approach | Sub-file |
|---|---|---|
| Plugin scaffold | Singleton + hooks class + assets class + HPOS declaration | scaffolding.md |
| Child theme setup | `wp_enqueue_style` parent + child in functions.php | scaffolding.md |
| Custom Post Type + Taxonomy | `register_post_type()` + `register_taxonomy()` in plugin | scaffolding.md |
| Secure AJAX handler | `wp_ajax_` hooks + nonce verify + `wp_send_json_*` | scaffolding.md |
| PHP sanitization / escaping | `wp_unslash()` + `sanitize_*` + `esc_*` patterns | php-standards.md |
| Transient caching | `get_transient` / `set_transient` | php-standards.md |
| External API call + WP_Error | `wp_remote_get()` + `WP_Error` pattern | php-standards.md |
| WP 6.8 password hashing | `wp_check_password()` + `wp_password_needs_rehash()` | php-standards.md |
| WP 6.8 app password / key hashing | `wp_fast_hash()` + `wp_verify_fast_hash()` (BLAKE2b) | php-standards.md |
| JS standards + enqueue defer/async | IIFE + WP 6.3+ enqueue API | js-css-standards.md |
| PHP → JS data passing | `wp_add_inline_script()` with `wp_json_encode()` | js-css-standards.md |
| CSS BEM + design tokens | Scoped tokens, 8pt spacing, fluid type | js-css-standards.md |
| Custom Elementor widget | `Widget_Base` + all required methods | elementor-patterns.md |
| Elementor Dynamic Tag | `Tag` class + `elementor/dynamic_tags/register` | elementor-patterns.md |
| Elementor Loop Grid query | `elementor/query/` filter | elementor-patterns.md |
| ACF field in Elementor | Dynamic Tag extending `\Elementor\Core\DynamicTags\Tag` | elementor-patterns.md |
| Elementor Pro Form action | `Action_Base` + field iteration | elementor-patterns.md |
| Theme Builder custom condition | `Condition_Base` + `elementor/theme/register_conditions` | elementor-patterns.md |
| WooCommerce HPOS compatibility | `FeaturesUtil::declare_compatibility()` + `wc_get_order()` | woocommerce.md |
| WooCommerce loop | Loop Grid + `elementor/query/` filter | woocommerce.md |
| Custom REST endpoint | `register_rest_route()` + schema callback in plugin | rest-api.md |
| Off-canvas filter panel | BEM CSS + ARIA JS + Custom Code block | offcanvas-ui.md |
| Performance checklist | Frontend + backend checklists | performance.md |
| Accessibility checklist | WCAG 2.2 AA patterns | performance.md |
| Elementor dependency check | `Requires Plugins: elementor` header + `did_action` fallback | scaffolding.md |
| Widget with button/CTA | Controls + render matching native Button widget | widget-button.md |
| Widget with container/layout | Controls + render matching native Container | widget-container.md |
| Widget with image | `Group_Control_Image_Size` + `get_attachment_image_html()` | widget-image.md |
| Widget with heading | TEXTAREA + header_size tag selector | widget-heading.md |
| Widget with rich text | WYSIWYG + wp_kses_post output | widget-text-editor.md |
| Widget with video embed | Source SELECT + overlay + aspect ratio | widget-video.md |
| Widget rendering saved Elementor template | get_builder_content_for_display() + SELECT2 | widget-elementor-template.md · widget-template-elementor.md |
| Widget with PHP template file | load_template() + locate_template() strategies A/B/C | widget-php-template.md · widget-template-php.md |
| Widget with divider line | Style + width + optional text/icon element | widget-divider.md |
| Widget with spacer gap | Single responsive SLIDER | widget-spacer.md |
| Widget with single icon | ICONS control + size + color tabs | widget-icon.md |
| Widget with icon + text box | Icon + title + description pattern | widget-icon-box.md |
| Widget with image + text box | Image + title + description pattern | widget-image-box.md |
| Widget with image grid | GALLERY control + Group_Control_Image_Size | widget-image-gallery.md |
| Widget with image slider | GALLERY + Swiper + navigation controls | widget-image-carousel.md |
| Widget with icon bullet list | REPEATER + icon + text + optional link | widget-icon-list.md |
| Widget with animated counter | Number + prefix/suffix + duration | widget-counter.md |
| Widget with progress bar | Percentage SLIDER + bar styling | widget-progress.md |
| Widget with testimonial quote | Content + image + name + job title | widget-testimonial.md |
| Widget with tabbed panels | REPEATER tabs + horizontal/vertical type | widget-tabs.md |
| Widget with accordion | REPEATER + single-open collapse pattern | widget-accordion.md |
| Widget with toggle panels | REPEATER + multi-open toggle pattern | widget-toggle.md |
| Widget with social icons | REPEATER + brand icons + links | widget-social-icons.md |
| Widget with alert/notice box | Type SELECT + title + description + dismiss | widget-alert.md |
| Widget with audio player | SoundCloud URL + autoplay options | widget-audio.md |
| Widget outputting shortcode | TEXTAREA + do_shortcode() | widget-shortcode.md |
| Widget with raw HTML embed | CODE control + unescaped output | widget-html.md |
| Widget as named anchor | TEXT ID + sanitize_html_class() | widget-menu-anchor.md |
| Widget outputting sidebar | Registered sidebar SELECT + dynamic_sidebar() | widget-sidebar.md |
| Widget with read more tag | No controls — WordPress $more global | widget-read-more.md |
| Widget with Google Maps | Address TEXT + zoom SLIDER + iframe | widget-google-maps.md |
| Widget with star rating display | Scale + rating number + icon style | widget-star-rating.md |
| Widget with schema rating | Icon count + fractional rating + gap | widget-rating.md |

---

## 5. Mandatory Widget Controls — No Hardcoded Visuals (NEVER VIOLATE)

> **This section is MANDATORY for every custom Elementor widget.** Whenever you build a widget,
> every visual property must be an Elementor control — NEVER a hardcoded CSS value. Users
> control appearance from the editor panel/toolbar, not by editing source code.

### The Rule

**NEVER hardcode** any of the following in PHP `render()`, in static CSS, or in
`content_template()` output:

- Colors (text, background, border, shadow)
- Typography (font family, size, weight, line-height, letter-spacing, transform)
- Spacing (padding, margin, gap)
- Sizing (width, height, min/max values)
- Borders (style, width, color, radius)
- Shadows (box-shadow, text-shadow)
- Backgrounds (color, gradient, image)
- Alignment / positioning
- Opacity, transitions, hover effects

**ALL** of the above must use Elementor controls with `selectors` that inject CSS dynamically.
The only exceptions are structural CSS (display, position, overflow) required for the widget
layout to function at all — and even these should use controls when there is a user-facing
choice (e.g. flex-direction toggle).

### Required Controls Checklist — Apply to Every Widget

When building a widget, include ALL controls that apply to its visual elements.
Use this checklist as a mandatory gate:

| Visual property | Required Elementor control | Tab |
|---|---|---|
| **Text content** | `TEXT`, `TEXTAREA`, or `WYSIWYG` + `'dynamic' => ['active' => true]` | TAB_CONTENT |
| **Typography** (any text element) | `add_group_control( Group_Control_Typography::get_type() )` | TAB_STYLE |
| **Text color** | `COLOR` control with `selectors` | TAB_STYLE |
| **Text alignment** | `add_responsive_control()` with `CHOOSE` (left/center/right/justify) | TAB_STYLE or TAB_CONTENT |
| **Background** | `add_group_control( Group_Control_Background::get_type() )` | TAB_STYLE |
| **Border** | `add_group_control( Group_Control_Border::get_type() )` | TAB_STYLE |
| **Border radius** | `add_responsive_control()` with `DIMENSIONS` + `'selectors'` | TAB_STYLE |
| **Box shadow** | `add_group_control( Group_Control_Box_Shadow::get_type() )` | TAB_STYLE |
| **Text shadow** | `add_group_control( Group_Control_Text_Shadow::get_type() )` | TAB_STYLE |
| **Padding** | `add_responsive_control()` with `DIMENSIONS` | TAB_STYLE |
| **Margin** | `add_responsive_control()` with `DIMENSIONS` | TAB_STYLE |
| **Width / Height** | `add_responsive_control()` with `SLIDER` | TAB_STYLE |
| **Spacing / Gap** | `add_responsive_control()` with `SLIDER` | TAB_STYLE |
| **Image** | `MEDIA` + `add_group_control( Group_Control_Image_Size::get_type() )` | TAB_CONTENT |
| **CSS Filters** (if image/element) | `add_group_control( Group_Control_Css_Filter::get_type() )` | TAB_STYLE |
| **Hover state** | Duplicate color/background/shadow controls inside `'section_style_hover'` with `selectors` targeting `:hover` | TAB_STYLE |
| **Transition duration** | `SLIDER` (seconds) with `selectors => ['transition-duration']` | TAB_STYLE |
| **Link** | `URL` control with `'dynamic' => ['active' => true]` | TAB_CONTENT |
| **Icon** | `ICONS` control with `fa4compatibility` | TAB_CONTENT |
| **HTML tag** | `SELECT` (h1–h6, div, span, p) | TAB_CONTENT |

### Example — Correct vs Incorrect

```php
// ❌ WRONG — hardcoded color and font-size
protected function render(): void {
    $settings = $this->get_settings_for_display();
    echo '<h2 style="color: #e94560; font-size: 24px;">'
        . esc_html( $settings['title'] ) . '</h2>';
}

// ✅ CORRECT — all visuals controlled from the panel via selectors
protected function register_controls(): void {
    // ... Content section with title TEXT control ...

    $this->start_controls_section( 'section_title_style', [
        'label' => esc_html__( 'Title Style', 'myplugin' ),
        'tab'   => \Elementor\Controls_Manager::TAB_STYLE,
    ] );

    $this->add_control( 'title_color', [
        'label'     => esc_html__( 'Color', 'myplugin' ),
        'type'      => \Elementor\Controls_Manager::COLOR,
        'selectors' => [
            '{{WRAPPER}} .myplugin-widget__title' => 'color: {{VALUE}};',
        ],
    ] );

    $this->add_group_control(
        \Elementor\Group_Control_Typography::get_type(),
        [
            'name'     => 'title_typography',
            'selector' => '{{WRAPPER}} .myplugin-widget__title',
        ]
    );

    $this->add_responsive_control( 'title_align', [
        'label'   => esc_html__( 'Alignment', 'myplugin' ),
        'type'    => \Elementor\Controls_Manager::CHOOSE,
        'options' => [
            'left'   => [ 'title' => esc_html__( 'Left',   'myplugin' ), 'icon' => 'eicon-text-align-left' ],
            'center' => [ 'title' => esc_html__( 'Center', 'myplugin' ), 'icon' => 'eicon-text-align-center' ],
            'right'  => [ 'title' => esc_html__( 'Right',  'myplugin' ), 'icon' => 'eicon-text-align-right' ],
        ],
        'selectors' => [
            '{{WRAPPER}} .myplugin-widget__title' => 'text-align: {{VALUE}};',
        ],
    ] );

    $this->add_responsive_control( 'title_spacing', [
        'label'      => esc_html__( 'Bottom Spacing', 'myplugin' ),
        'type'       => \Elementor\Controls_Manager::SLIDER,
        'size_units' => [ 'px', 'em', 'rem' ],
        'range'      => [ 'px' => [ 'min' => 0, 'max' => 100 ] ],
        'selectors'  => [
            '{{WRAPPER}} .myplugin-widget__title' => 'margin-bottom: {{SIZE}}{{UNIT}};',
        ],
    ] );

    $this->end_controls_section();
}

protected function render(): void {
    $settings = $this->get_settings_for_display();
    // ✅ No inline styles — all visuals come from Elementor's selectors
    $this->add_render_attribute( 'title', 'class', 'myplugin-widget__title' );
    $this->add_inline_editing_attributes( 'title' );
    echo '<h2 ' . $this->get_render_attribute_string( 'title' ) . '>'
        . esc_html( $settings['title'] ) . '</h2>';
}
```

### `add_render_attribute()` and `add_inline_editing_attributes()` — Always Use

The official Elementor API for building HTML attributes is `$this->add_render_attribute()`.
**Always use it** instead of manually concatenating class/id/aria attributes in `render()`.
Pair with `$this->add_inline_editing_attributes()` for any text field that supports live
editing in the Elementor editor panel.

Source: developers.elementor.com/docs/widgets/rendering-html-attribute/
Source: developers.elementor.com/docs/widgets/rendering-inline-editing/
