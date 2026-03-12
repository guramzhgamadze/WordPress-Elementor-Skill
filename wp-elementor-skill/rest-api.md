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
