# REST API Endpoints

```php
// Always in a plugin — never in functions.php
add_action( 'rest_api_init', function() {
    register_rest_route( 'myplugin/v1', '/items/(?P<id>\d+)', [
        'methods'             => \WP_REST_Server::READABLE,
        'callback'            => 'myplugin_rest_get_item',
        'permission_callback' => 'myplugin_rest_permissions',
        'args'                => [
            'id' => [
                // ✅ Use WP's built-in schema validation — NOT a custom is_numeric() closure.
                // is_numeric() accepts floats ('1.5'), negatives ('-1'), and scientific notation
                // ('1e5'), none of which are valid post IDs. The 'type' + 'minimum' +
                // 'rest_validate_request_arg' pattern is the canonical WP REST API approach.
                'type'              => 'integer',
                'minimum'           => 1,
                'sanitize_callback' => 'absint',
                'validate_callback' => 'rest_validate_request_arg',
                'required'          => true,
                'description'       => 'The ID of the item to retrieve.',
            ],
        ],
        // ✅ Always include a schema callback — required for REST API schema discovery,
        // client code generation, and WP-CLI route inspection.
        'schema' => 'myplugin_get_item_schema',
    ] );
} );

/**
 * JSON Schema for the /items/{id} endpoint.
 */
function myplugin_get_item_schema(): array {
    return [
        // ✅ WordPress REST API uses JSON Schema draft-04 — NOT draft-07.
        // WP core validates against draft-04 via its own schema validator. Using
        // draft-07 keywords (e.g. boolean 'exclusiveMinimum', 'if/then/else', '$ref' resolution)
        // will be silently ignored or cause validation errors on WP 6.x+.
        // Source: developer.wordpress.org/rest-api/extending-the-rest-api/schema/
        '$schema'    => 'http://json-schema.org/draft-04/schema#',
        'title'      => 'myplugin_item',
        'type'       => 'object',
        'properties' => [
            'id'      => [ 'type' => 'integer', 'description' => 'Unique post ID.' ],
            'title'   => [ 'type' => 'string',  'description' => 'Post title.' ],
            'content' => [ 'type' => 'string',  'description' => 'Post content (filtered HTML).' ],
        ],
    ];
}

function myplugin_rest_permissions( WP_REST_Request $request ): bool|WP_Error {
    // Return WP_Error (not false) on denial — gives client a proper JSON error body.
    // Returning false sends a generic 403 with no context.
    if ( ! current_user_can( 'read' ) ) {
        return new WP_Error(
            'rest_forbidden',
            __( 'You do not have permission to access this resource.', 'myplugin' ),
            [ 'status' => 403 ]
        );
    }
    return true;
}

// ─── REST API Nonce — Authenticating JS Fetch Calls ─────────────────────────
//
// WordPress REST API uses a SEPARATE nonce from AJAX (wp_ajax_*).
// The REST nonce is generated with: wp_create_nonce( 'wp_rest' )
// It must be sent as the X-WP-Nonce request header (not a POST body param).
//
// ✅ CORRECT — pass nonce to JS via wp_add_inline_script():
// $data = wp_json_encode( [ 'nonce' => wp_create_nonce( 'wp_rest' ), 'restUrl' => rest_url() ] );
// wp_add_inline_script( 'myplugin-js', 'const mypluginRest = ' . $data . ';', 'before' );
//
// ✅ CORRECT — JS fetch with nonce header:
// const res = await fetch( mypluginRest.restUrl + 'myplugin/v1/items/1', {
//   headers: { 'X-WP-Nonce': mypluginRest.nonce, 'Content-Type': 'application/json' },
// } );
//
// ⚠️ REST nonces expire after 24 hours. If your SPA/page stays open overnight, handle
// 403 (rest_cookie_invalid_nonce) responses by fetching a fresh nonce and retrying:
//
// ✅ CORRECT nonce refresh — WordPress core registers wp_ajax_rest_nonce() on the
// 'wp_ajax_rest_nonce' action (note: underscore, not hyphen). Calling this endpoint
// with logged-in credentials returns a fresh wp_rest nonce as plain text:
// Source: developer.wordpress.org/reference/functions/wp_ajax_rest_nonce/
//
// const refreshed  = await fetch( ajaxUrl + '?action=rest_nonce', { credentials: 'include' } );
// mypluginRest.nonce = ( await refreshed.text() ).trim();
// // then retry the original request with the new nonce header
//
// ⚠️ The action is 'rest_nonce' (underscore) not 'rest-nonce' (hyphen).
// ⚠️ User must be logged in — this endpoint returns a nonce for cookie-based auth only.
//
// ⚠️ Public endpoints (no authentication required): use '__return_true' as permission_callback.
// Never use '__return_false' — that permanently blocks all access including admins.
// register_rest_route( 'myplugin/v1', '/public-data', [
//     'methods'             => WP_REST_Server::READABLE,
//     'callback'            => 'myplugin_public_callback',
//     'permission_callback' => '__return_true',
// ] );

function myplugin_rest_get_item( WP_REST_Request $request ): WP_REST_Response|WP_Error {
    $id   = $request->get_param( 'id' );
    $post = get_post( $id );

    if ( ! $post || 'publish' !== $post->post_status ) {
        return new WP_Error( 'not_found', __( 'Item not found.', 'myplugin' ), [ 'status' => 404 ] );
    }

    // ✅ Set up global post context before apply_filters('the_content').
    // Without this, shortcodes, Gutenberg blocks, embeds, and any plugin that
    // reads the global $post will fail silently or produce wrong output.
    // wp_reset_postdata() restores the previous global $post state after rendering.
    $GLOBALS['post'] = $post;
    setup_postdata( $post );

    $response = rest_ensure_response( [
        'id'      => $post->ID,
        'title'   => get_the_title( $post ),
        'content' => apply_filters( 'the_content', $post->post_content ),
    ] );

    wp_reset_postdata();

    return $response;
}
```
