# PHP Standards

## Sanitization, Escaping, and Nonces

```php
// INPUT — always sanitize immediately on receipt
// ✅ wp_unslash() BEFORE sanitize_text_field / sanitize_email / wp_kses_post / sanitize_url.
// WordPress magic-quotes all $_POST/$_GET superglobals. Without wp_unslash() first,
// a value like "O'Reilly" is sanitized as "O\'Reilly" and stored with corrupt backslashes.
// Exception: absint() — integers cannot contain slashes, so wp_unslash() is not needed.
// Exception: sanitize_key() — strips slashes implicitly (see WPCS wiki).
$name    = sanitize_text_field( wp_unslash( $_POST['name'] ?? '' ) );
$email   = sanitize_email( wp_unslash( $_POST['email'] ?? '' ) );
$id      = absint( $_GET['post_id'] ?? 0 );
$content = wp_kses_post( wp_unslash( $_POST['content'] ?? '' ) );

// ✅ sanitize_url() is the canonical function for URL input sanitization.
// History: existed since WP 2.3.1, deprecated in WP 2.8.0 in favour of esc_url_raw(),
// then UN-DEPRECATED and restored as canonical in WP 5.9.0 to honour the naming convention
// "use sanitize_ to sanitize, esc_ to escape". esc_url_raw() is now an alias for sanitize_url().
// Use sanitize_url() — it is correct, un-deprecated, and WPCS-clean on WP 5.9+.
// Source: make.wordpress.org/plugins/2022/05/25/rejoice-to-sanitize_url/
//         developer.wordpress.org/reference/functions/sanitize_url/ (restored since 5.9)
//
// ⚠️ WP 6.9 BEHAVIOUR CHANGE — esc_url(), esc_url_raw(), and sanitize_url():
// In WP 6.8 and earlier, a protocol-less URL (e.g. "example.com/path") was prepended
// with http:// before processing. Starting in WP 6.9, if 'https' is the first item in
// the $protocols argument (which it IS by default), the functions prepend https:// instead.
// Test any code that passes bare URLs (no scheme) through these functions after upgrading.
// Source: make.wordpress.org/core/2025/11/19/url-escaping-functions-can-support-https-as-the-default-protocol-in-wordpress-6-9/
$url     = sanitize_url( wp_unslash( $_POST['redirect'] ?? '' ) );

// OUTPUT — always escape at the point of output
echo esc_html( $name );
echo esc_attr( $name );          // inside HTML attributes
echo esc_url( $url );            // href, src
echo wp_kses_post( $content );   // rich HTML content

// NONCES — on every form and AJAX action
// Render:
wp_nonce_field( 'myplugin_save_action', 'myplugin_nonce' );

// Verify:
// ✅ sanitize_key() is the WPCS-canonical sanitizer for nonces — it strips slashes
// automatically so wp_unslash() is not required (unlike sanitize_text_field).
if ( ! isset( $_POST['myplugin_nonce'] )
    || ! wp_verify_nonce( sanitize_key( $_POST['myplugin_nonce'] ), 'myplugin_save_action' )
) {
    wp_die( esc_html__( 'Security check failed.', 'myplugin' ) );
}
```

---

## WP_Error and External API Calls

```php
// Always use WP_Error for communicable failures
function myplugin_fetch_data( int $post_id ): array|WP_Error {
    if ( $post_id <= 0 ) {
        return new WP_Error( 'invalid_id', __( 'Invalid post ID.', 'myplugin' ) );
    }

    $response = wp_remote_get(
        'https://api.example.com/data/' . $post_id,
        [ 'timeout' => 10, 'sslverify' => true ]
    );

    if ( is_wp_error( $response ) ) {
        return $response;
    }

    $code = wp_remote_retrieve_response_code( $response );
    if ( 200 !== $code ) {
        return new WP_Error( 'api_error', sprintf(
            /* translators: %d: HTTP response code */
            __( 'API returned status %d.', 'myplugin' ),
            $code
        ), [ 'status' => $code ] );
    }

    $body    = wp_remote_retrieve_body( $response );
    $decoded = json_decode( $body, true );
    return is_array( $decoded ) ? $decoded : [];
}

// Caller always checks for errors
$data = myplugin_fetch_data( $post_id );
if ( is_wp_error( $data ) ) {
    // ✅ Guard error_log() with WP_DEBUG — raw error messages can leak sensitive
    // data (API keys, user data) into server logs on hosts where WP_DEBUG_LOG is enabled.
    if ( defined( 'WP_DEBUG' ) && WP_DEBUG ) {
        error_log( 'myplugin: ' . $data->get_error_message() );
    }
}
```

---

## Transient Caching

```php
// ⚠️ TRANSIENT KEY LIMIT: 172 characters maximum (WordPress enforces this silently —
// keys over the limit appear to save but are never retrieved).
// For dynamic keys built from multiple segments, use md5():

// ✅ Simple key — safe
$cache_key = 'myplugin_data_' . $post_id;

// ✅ Dynamic key — use md5() to guarantee safe length
$cache_key = 'myplugin_' . md5( $post_id . '_' . $user_id . '_' . $locale );

function myplugin_get_cached_data( int $post_id ): array {
    $cache_key = 'myplugin_data_' . $post_id;
    $cached    = get_transient( $cache_key );

    if ( false !== $cached ) {
        return $cached;
    }

    $data = myplugin_fetch_data( $post_id );

    if ( ! is_wp_error( $data ) ) {
        set_transient( $cache_key, $data, HOUR_IN_SECONDS );
    }

    return is_wp_error( $data ) ? [] : $data;
}
```

---

## General PHP Rules

```php
// ✅ Bail early — avoid deeply nested conditionals
function myplugin_process(): void {
    if ( ! is_user_logged_in() ) return;
    if ( ! current_user_can( 'edit_posts' ) ) return;
    // ... proceed
}

// ✅ Prefix ALL global symbols
function myplugin_my_function(): void {}
class MyPlugin_Utility {}
define( 'MYPLUGIN_DEBUG', false );
do_action( 'myplugin/after_save', $post_id );

// ✅ OOP for anything beyond ~20 lines
// ✅ Use WP_Query / get_posts() — avoid raw $wpdb unless unavoidable
// ✅ Never suppress errors with @ — fix the root cause

// ✅ PHP 8.2+ readonly classes — ideal for immutable plugin config/value objects
readonly class MyPlugin_Config {
    public function __construct(
        public string $api_key,
        public int    $cache_ttl,
        public string $endpoint,
    ) {}
}

// ✅ PHP 8.3+ typed class constants — prefer over untyped define() in OOP contexts
class MyPlugin_Constants {
    const string VERSION   = '1.0.0';
    const int    CACHE_TTL = 3600;
}
```

> ⚠️ **`declare(strict_types=1)` placement rule:** Must be the **very first statement** in
> its file, immediately after `<?php` — before any output, namespace, use, require, or other
> statements. Every PHP file that uses it must open like this:
> ```php
> <?php
> declare( strict_types=1 );
> // ... rest of file
> ```

---

## WP 6.8+ Password Hashing (bcrypt / BLAKE2b)

> ⚠️ **Breaking change in WP 6.8 (April 2025):** WordPress replaced the old phpass
> MD5-based scheme with **bcrypt** for user passwords and **BLAKE2b** for application
> passwords and security keys.
>
> **Hash prefix reference (WP 6.8+):**
> - `$wp$2y$` — bcrypt (user passwords via `wp_hash_password()`). The `$wp` prefix
>   distinguishes WordPress's pre-hashed bcrypt from vanilla bcrypt used by third-party plugins.
> - `$generic$` — BLAKE2b via Sodium (application passwords, password reset keys, personal
>   data request keys, recovery mode key), used via `wp_fast_hash()`.
> - `$P$` — Legacy phpass hash (still valid; rehashed opportunistically on next login).
> - **Post passwords** still use phpass in WP 6.8.
>
> Source: make.wordpress.org/core/2025/02/17/wordpress-6-8-will-use-bcrypt-for-password-hashing/

```php
// ✅ ALWAYS use wp_check_password() — never raw string comparison against user_pass.
if ( wp_check_password( $plain_password, $stored_hash, $user->ID ) ) {

    // ✅ WP 6.8+: check whether the stored hash needs upgrading to bcrypt.
    if ( wp_password_needs_rehash( $stored_hash ) ) {
        // ✅ wp_set_password() takes the PLAINTEXT password and hashes internally.
        // Do NOT pre-hash with wp_hash_password() — double-hashing permanently locks users out.
        //
        // ✅ wp_set_password() does NOT send any password-change email and does NOT fire
        // 'after_password_reset'. It only fires the 'wp_set_password' action (WP 6.2+).
        // 'after_password_reset' is fired by WordPress's higher-level reset_password()
        // function, which calls wp_set_password() internally. Calling wp_set_password()
        // directly (as in this rehash flow) does not trigger that hook at all.
        // Source: developer.wordpress.org/reference/functions/wp_set_password/
        //         developer.wordpress.org/reference/functions/reset_password/
        //
        // ⚠️ SESSION INVALIDATION: wp_set_password() invalidates ALL existing auth cookies
        // for this user immediately. Call wp_set_auth_cookie() afterward to re-issue a valid
        // cookie for the current session if you want the user to stay logged in.
        wp_set_password( $plain_password, $user->ID );
    }
}

// ✅ To hash a password: always wp_hash_password() — never password_hash() directly.
$hash = wp_hash_password( $plain_password );

// ❌ NEVER inspect hash prefixes directly — they changed in WP 6.8 and may change again.

// ✅ wp_fast_hash() / wp_verify_fast_hash() — BLAKE2b via Sodium.
// ⚠️ IMPORTANT: HIGH-ENTROPY RANDOM INPUT ONLY (>128 bits).
// MUST NOT be used for user passwords (use wp_hash_password()).
// MUST NOT be used for low-entropy input such as user IDs, emails, short tokens
// (use wp_hash() for those). Intended only for randomly-generated high-entropy secrets
// such as application passwords and security keys.
$hash  = wp_fast_hash( $random_secret );
$valid = wp_verify_fast_hash( $input, $hash ); // returns bool
```
