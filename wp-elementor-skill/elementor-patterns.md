# Elementor-Specific Patterns

## V4 Compatibility Rules (Apply Now)

> ⚠️ **Apply these rules to all new code — before V4 becomes the default:**
> 1. **NEVER target `.elementor-widget-container` in CSS or JS** — deprecated since 3.26.
>    As of 3.35.7 the Optimized Markup experiment is still opt-in, but users may enable it.
>    Write all new code as if this wrapper is absent.
> 2. **ALWAYS include `has_widget_inner_wrapper(): false`** on every new widget.
> 3. **Do NOT use `strategy: 'defer'`** on scripts that listen for `elementor/frontend/init`.
> 4. V4 is production-ready as of Elementor 3.35 (February 2, 2026), but the **PHP extension
>    API for custom Atomic Elements is not yet publicly documented**. Use V3 `Widget_Base` only.
> 5. **Elementor 3.34.2+ Unified Admin Menu (January 20, 2026):** Register all admin menus
>    using standard WordPress `add_submenu_page('elementor', ...)` only. Non-standard injection
>    methods are silently dropped.

---

## Custom Widget — Complete Pattern

**Step 1 — Plugin header must declare Elementor dependency (WP 6.5+):**
```php
/**
 * Plugin Name:  My Plugin
 * Requires Plugins: elementor
 * Requires PHP: 8.3
 * Requires at least: 6.9
 */
```
> ⚠️ Do NOT add inline comments on the `Requires Plugins:` line — `get_file_data()` reads
> everything after the colon as the value. Keep the `did_action('elementor/loaded')` runtime
> check (see scaffolding.md) as a fallback for pre-6.5 sites.

**Step 2 — Register widget assets on `wp_enqueue_scripts` (register only — do NOT enqueue):**
```php
// ⚠️ DO NOT add strategy:'defer' to widget scripts that use elementor/frontend/init.
// Use ['in_footer' => true] WITHOUT strategy:defer. The 'elementor-frontend' dependency
// guarantees document-order execution AFTER Elementor's script.
add_action( 'wp_enqueue_scripts', function() {
    wp_register_script(
        'myplugin-widget-js',
        MYPLUGIN_URL . 'assets/js/myplugin-widget.js',
        [ 'elementor-frontend' ],  // ✅ dependency guarantees load order
        MYPLUGIN_VERSION,
        [ 'in_footer' => true ]    // ✅ no defer — see note above
    );
    wp_register_style(
        'myplugin-widget-css',
        MYPLUGIN_URL . 'assets/css/myplugin-widget.css',
        [],
        MYPLUGIN_VERSION
    );
} );
```

**Step 3 — Widget class with all required methods:**
```php
// includes/class-myplugin-widget.php
defined( 'ABSPATH' ) || exit;

class MyPlugin_Widget extends \Elementor\Widget_Base {

    public function get_name(): string        { return 'myplugin-widget'; }
    public function get_title(): string       { return esc_html__( 'My Widget', 'myplugin' ); }
    public function get_icon(): string        { return 'eicon-code'; }
    public function get_categories(): array   { return [ 'general' ]; }
    public function get_keywords(): array     { return [ 'custom', 'myplugin' ]; }

    // ✅ REQUIRED: Declare JS dependencies — Elementor loads these only on pages using this widget
    public function get_script_depends(): array {
        return [ 'myplugin-widget-js' ];
    }

    // ✅ REQUIRED: Declare CSS dependencies
    public function get_style_depends(): array {
        return [ 'myplugin-widget-css' ];
    }

    // ✅ REQUIRED on ALL new widgets — removes the redundant inner wrapper div
    // (.elementor-widget-container). Return true ONLY if your render() output
    // physically requires that inner wrapper.
    public function has_widget_inner_wrapper(): bool {
        return false;
    }

    // ✅ Return false if widget renders static content — enables Elementor output caching.
    // Return true if output depends on current user, session, time, or other dynamic data.
    protected function is_dynamic_content(): bool {
        return false;
    }

    protected function register_controls(): void {

        $this->start_controls_section( 'section_content', [
            'label' => esc_html__( 'Content', 'myplugin' ),
            'tab'   => \Elementor\Controls_Manager::TAB_CONTENT,
        ] );

        $this->add_control( 'title', [
            'label'              => esc_html__( 'Title', 'myplugin' ),
            'type'               => \Elementor\Controls_Manager::TEXT,
            'default'            => esc_html__( 'Default Title', 'myplugin' ),
            'label_block'        => true,
            'frontend_available' => true,
        ] );

        $this->end_controls_section();

        $this->start_controls_section( 'section_style', [
            'label' => esc_html__( 'Title Style', 'myplugin' ),
            'tab'   => \Elementor\Controls_Manager::TAB_STYLE,
        ] );

        $this->add_control( 'title_color', [
            'label'     => esc_html__( 'Color', 'myplugin' ),
            'type'      => \Elementor\Controls_Manager::COLOR,
            'selectors' => [
                // ✅ {{WRAPPER}} resolves to .elementor-widget-myplugin-widget (widget root)
                // NEVER use .elementor-widget-container here — see V4 rules above
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

        $this->end_controls_section();
    }

    protected function render(): void {
        $settings = $this->get_settings_for_display();

        // ✅ Null-safe check — Plugin::$instance->editor is null on the frontend
        $is_editor = \Elementor\Plugin::$instance->editor
                     && \Elementor\Plugin::$instance->editor->is_edit_mode();
        ?>
        <div class="myplugin-widget">
            <h2 class="myplugin-widget__title">
                <?php echo esc_html( $settings['title'] ); ?>
            </h2>
            <?php if ( $is_editor ) : ?>
                <span class="myplugin-widget__editor-hint" style="opacity:0.5;font-size:0.75rem;">
                    <?php esc_html_e( '[Editor preview]', 'myplugin' ); ?>
                </span>
            <?php endif; ?>
        </div>
        <?php
    }

    // ✅ REQUIRED: JS/Backbone template for live editor preview.
    // {{{ }}} for TEXT control values — Elementor processes values via get_settings_for_display()
    // before they reach content_template(). {{{ }}} renders the processed value without
    // double-encoding. {{ }} would HTML-encode already-safe values, corrupting apostrophes etc.
    protected function content_template(): void {
        ?>
        <div class="myplugin-widget">
            <h2 class="myplugin-widget__title">{{{ settings.title }}}</h2>
        </div>
        <?php
    }
}

// ✅ REQUIRED registration hook
add_action( 'elementor/widgets/register', function( \Elementor\Widgets_Manager $manager ) {
    require_once MYPLUGIN_PATH . 'includes/class-myplugin-widget.php';
    $manager->register( new MyPlugin_Widget() );
} );
```

---

## Dynamic Tag Registration

> **Dynamic Tag Category Reference (as of Elementor 3.35):**
>
> | Constant | Value | Available in | Use for |
> |---|---|---|---|
> | `Module::TEXT_CATEGORY` | `'text'` | **Free** | Single-line text fields |
> | `Module::URL_CATEGORY` | `'url'` | **Free** | URL/link controls |
> | `Module::IMAGE_CATEGORY` | `'image'` | **Free** | Media/image controls |
> | `Module::MEDIA_CATEGORY` | `'media'` | **Free** | Alias for IMAGE_CATEGORY |
> | `Module::NUMBER_CATEGORY` | `'number'` | **Free** | Slider/number controls |
> | `Module::COLOR_CATEGORY` | `'color'` | **Free** | Color controls |
> | `Module::POST_GROUP` | `'post'` | **Pro only** | ⚠️ Fatal error without Pro |
> | `Module::SITE_GROUP` | `'site'` | **Pro only** | ⚠️ Fatal error without Pro |
>
> Always use the class constant (`Module::TEXT_CATEGORY`) not the raw string — the string
> values are internal and could change. Source: `elementor/modules/dynamic-tags/module.php`

```php
// ✅ Type hint \Elementor\Core\DynamicTags\Manager (different from Widgets_Manager)
add_action( 'elementor/dynamic_tags/register', function( \Elementor\Core\DynamicTags\Manager $manager ) {
    // ✅ Register the custom group BEFORE registering the tag that references it.
    // ⚠️ Do NOT use \Elementor\Modules\DynamicTags\Module::POST_GROUP here.
    // POST_GROUP is Elementor Pro-only — causes PHP fatal error if Pro is not active.
    $manager->register_group( 'myplugin', [
        'title' => esc_html__( 'My Plugin', 'myplugin' ),
    ] );
    require_once MYPLUGIN_PATH . 'includes/class-myplugin-dynamic-tag.php';
    $manager->register( new MyPlugin_Dynamic_Tag() );
} );

class MyPlugin_Dynamic_Tag extends \Elementor\Core\DynamicTags\Tag {
    public function get_name(): string     { return 'myplugin-tag'; }
    public function get_title(): string    { return esc_html__( 'My Custom Tag', 'myplugin' ); }
    // ✅ get_group() MUST return array — returning a string causes PHP TypeError
    public function get_group(): array     { return [ 'myplugin' ]; }
    public function get_categories(): array {
        // ✅ TEXT_CATEGORY IS defined in free Elementor — safe to use without Pro
        return [ \Elementor\Modules\DynamicTags\Module::TEXT_CATEGORY ];
    }

    public function render(): void {
        $value = get_post_meta( get_the_ID(), '_my_custom_field', true );
        echo esc_html( (string) $value );
    }
}
```

---

## Loop Grid — Custom Query Filter

```php
// In Elementor editor → Query ID field: set to "myplugin_loop_query"
add_action( 'elementor/query/myplugin_loop_query', function( \WP_Query $query ) {
    $query->set( 'post_type',      'product' );
    $query->set( 'posts_per_page', 6 );
    $query->set( 'no_found_rows',  true ); // skip COUNT(*) when no pagination needed
    $query->set( 'tax_query', [ [
        'taxonomy' => 'product_cat',
        'field'    => 'slug',
        'terms'    => [ 'featured' ],
    ] ] );
} );
```

---

## ACF + Elementor Dynamic Tag (Full Pattern)

```php
class MyPlugin_ACF_Tag extends \Elementor\Core\DynamicTags\Tag {
    public function get_name(): string     { return 'myplugin-acf-tag'; }
    public function get_title(): string    { return esc_html__( 'ACF Field', 'myplugin' ); }
    public function get_group(): array     { return [ 'myplugin' ]; }
    public function get_categories(): array {
        return [ \Elementor\Modules\DynamicTags\Module::TEXT_CATEGORY ];
    }

    protected function register_controls(): void {
        $this->add_control( 'field_key', [
            'label' => esc_html__( 'ACF Field Key or Name', 'myplugin' ),
            'type'  => \Elementor\Controls_Manager::TEXT,
        ] );
    }

    public function render(): void {
        $field_key = $this->get_settings( 'field_key' );
        if ( empty( $field_key ) || ! function_exists( 'get_field' ) ) return;
        // ✅ sanitize_text_field() NOT sanitize_key() — sanitize_key() lowercases and
        // strips non-[a-z0-9_-] characters, silently corrupting mixed-case ACF field names.
        $value = get_field( sanitize_text_field( $field_key ) );

        // get_field() can return arrays (repeaters) — guard against it
        if ( is_array( $value ) || is_object( $value ) ) return;

        echo esc_html( (string) $value );
    }
}

add_action( 'elementor/dynamic_tags/register', function( \Elementor\Core\DynamicTags\Manager $manager ) {
    // register_group() is idempotent — safe to call in multiple callbacks
    $manager->register_group( 'myplugin', [
        'title' => esc_html__( 'My Plugin', 'myplugin' ),
    ] );
    require_once MYPLUGIN_PATH . 'includes/class-myplugin-acf-tag.php';
    $manager->register( new MyPlugin_ACF_Tag() );
} );
```

---

## Elementor Pro Form — Custom Action (Elementor Pro 3.28+)

Elementor Pro 3.28 (March 2025) updated the Form **Field** API (`Field_Base`) so fields
can declare JS/CSS dependencies via methods (identical to widget `get_script_depends()`)
instead of the old `$depended_scripts` / `$depended_styles` properties (now deprecated
on `Field_Base`). Form Actions (`Action_Base`) are not part of this documented change.

```php
// Requires: Elementor Pro
class MyPlugin_Form_Action extends \ElementorPro\Modules\Forms\Classes\Action_Base {

    public function get_name(): string  { return 'myplugin-action'; }
    public function get_label(): string { return esc_html__( 'My Plugin Action', 'myplugin' ); }

    public function get_script_depends(): array {
        return [ 'myplugin-form-js' ];
    }

    public function get_style_depends(): array {
        return [ 'myplugin-form-css' ];
    }

    // ✅ Type hint omitted intentionally — Elementor does not expose a public interface/class
    // for the $widget parameter. Using \Elementor\Widget_Base is the closest match but
    // Elementor passes its internal form widget class. Omit type hint to avoid TypeError.
    public function register_settings_section( $widget ): void {
        $widget->start_controls_section( 'myplugin_action_section', [
            'label'     => esc_html__( 'My Plugin', 'myplugin' ),
            'condition' => [ 'submit_actions' => $this->get_name() ],
        ] );

        $widget->add_control( 'myplugin_endpoint', [
            'label' => esc_html__( 'Endpoint URL', 'myplugin' ),
            'type'  => \Elementor\Controls_Manager::TEXT,
        ] );

        $widget->end_controls_section();
    }

    public function on_export( array $element ): array {
        return $element;
    }

    // $record = \ElementorPro\Modules\Forms\Classes\Form_Record
    // $ajax_handler = \ElementorPro\Modules\Forms\Classes\Ajax_Handler
    // Both classes are Pro-only — do NOT type hint them directly here or PHP
    // will throw a fatal error on sites where Elementor Pro is inactive.
    public function run( $record, $ajax_handler ): void {
        $settings = $record->get( 'form_settings' );
        $endpoint = sanitize_url( $settings['myplugin_endpoint'] ?? '' );

        if ( empty( $endpoint ) ) return;

        // ✅ $record->get('fields') returns an associative array keyed by field ID.
        // Loop key $id IS the field ID — do NOT use $field['id'].
        $raw_fields = $record->get( 'fields' );
        $payload    = [];
        foreach ( $raw_fields as $id => $field ) {
            $payload[ sanitize_key( $id ) ] = sanitize_text_field( $field['value'] );
        }

        $response = wp_remote_post( $endpoint, [
            'body'    => wp_json_encode( $payload ),
            'headers' => [ 'Content-Type' => 'application/json' ],
            'timeout' => 15,
        ] );

        if ( is_wp_error( $response ) ) {
            $ajax_handler->add_error_message(
                esc_html__( 'Submission failed. Please try again.', 'myplugin' )
            );
        }
    }
}

add_action( 'elementor_pro/forms/actions/register', function( $form_actions_registrar ) {
    require_once MYPLUGIN_PATH . 'includes/class-myplugin-form-action.php';
    $form_actions_registrar->register( new MyPlugin_Form_Action() );
} );
```

---

## Elementor Theme Builder — Custom Display Conditions

> ⚠️ **API scope:** Only registering **new condition types** via
> `elementor/theme/register_conditions` is the public API. Programmatically
> **assigning** conditions to an existing template is undocumented internal operation.

```php
// Requires: Elementor Pro
class MyPlugin_Logged_In_Condition extends \ElementorPro\Modules\ThemeBuilder\Conditions\Condition_Base {

    // 'general' = site-wide; 'singular' = singular posts/pages; 'archive' = archive pages
    public static function get_type(): string { return 'general'; }
    public function get_name(): string        { return 'myplugin_logged_in'; }
    public function get_label(): string       { return esc_html__( 'Logged-In User', 'myplugin' ); }
    public function get_all_label(): string   { return esc_html__( 'All Logged-In Users', 'myplugin' ); }

    public function check( $args ): bool {
        return is_user_logged_in();
    }
}

add_action( 'elementor/theme/register_conditions', function( $conditions_manager ) {
    require_once MYPLUGIN_PATH . 'includes/class-myplugin-logged-in-condition.php';
    $conditions_manager
        ->get_condition( 'general' )
        ->register_sub_condition( new MyPlugin_Logged_In_Condition() );
} );
```

**Custom condition with role selector:**
```php
class MyPlugin_User_Role_Condition extends \ElementorPro\Modules\ThemeBuilder\Conditions\Condition_Base {

    public static function get_type(): string { return 'general'; }
    public function get_name(): string        { return 'myplugin_user_role'; }
    public function get_label(): string       { return esc_html__( 'User Role', 'myplugin' ); }
    public function get_all_label(): string   { return esc_html__( 'All Roles', 'myplugin' ); }

    protected function register_sub_conditions(): void {
        global $wp_roles;
        $roles = [];
        foreach ( $wp_roles->roles as $slug => $role ) {
            $roles[ $slug ] = translate_user_role( $role['name'] );
        }
        $this->add_control( 'myplugin_role_select', [
            'section' => 'settings',
            'label'   => esc_html__( 'Role', 'myplugin' ),
            'type'    => \Elementor\Controls_Manager::SELECT,
            'options' => $roles,
        ] );
    }

    public function check( $args ): bool {
        $required_role = $args['id'] ?? '';
        if ( ! $required_role || ! is_user_logged_in() ) return false;
        $user = wp_get_current_user();
        return in_array( $required_role, (array) $user->roles, true );
    }
}
```

> **Programmatic condition assignment (internal — use only for one-time migration scripts):**
> ```php
> update_post_meta( $template_id, '_elementor_conditions', [
>     [ 'name' => 'general', 'id' => '', 'sub_name' => '', 'sub_id' => '' ],
> ] );
> delete_transient( 'elementor_pro_theme_builder_conditions' );
> ```
