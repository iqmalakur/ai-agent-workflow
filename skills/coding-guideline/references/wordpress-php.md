# WordPress PHP Formatting Examples

Read [common rules](common.md) and [PHP](php.md), then apply the project's WordPress coding standards and WPCS configuration.
Add [HTML](html.md), [CSS](css.md), or [JavaScript](javascript-typescript.md) guidance when changing those languages.
Use WordPress formatting for WordPress source and follow the project's supported PHP version.

## Production code

The example registers a REST endpoint that creates draft articles for users who can edit posts.
It prepares input, inserts the draft, handles `WP_Error`, and returns the created identifier.
WordPress supplies the global functions and `WP_REST_Request`, `WP_REST_Server`, `WP_REST_Response`, and `WP_Error` classes.
The `example_` prefix represents the plugin's own function prefix.
Both layouts preserve permissions, validation, slashing, draft status, and author assignment.

### Incorrect

Functions and logical groups have no empty-line separation.
The permission callback occupies one line between larger functions.

```php
<?php
function example_register_draft_routes() {
	register_rest_route(
		'example/v1',
		'/drafts',
		array(
			'methods'             => WP_REST_Server::CREATABLE,
			'callback'            => 'example_create_draft',
			'permission_callback' => 'example_can_create_draft',
			'args'                => array(
				'title'   => array(
					'required' => true,
					'type'     => 'string',
				),
				'content' => array(
					'default' => '',
					'type'    => 'string',
				),
			),
		)
	);
}
function example_can_create_draft() { return current_user_can( 'edit_posts' ); }
function example_create_draft( WP_REST_Request $request ) {
	$title   = sanitize_text_field( $request->get_param( 'title' ) );
	$content = wp_kses_post( $request->get_param( 'content' ) );
	if ( '' === $title ) {
		return new WP_Error(
			'invalid_title',
			'A title is required.',
			array( 'status' => 400 )
		);
	}
	$post_data = array(
		'post_title'   => $title,
		'post_content' => $content,
		'post_status'  => 'draft',
		'post_author'  => get_current_user_id(),
	);
	$post_id   = wp_insert_post( wp_slash( $post_data ), true );
	if ( is_wp_error( $post_id ) ) {
		return $post_id;
	}
	return new WP_REST_Response( array( 'id' => $post_id ), 201 );
}
add_action( 'rest_api_init', 'example_register_draft_routes' );
```

### Correct

Use WordPress spacing and tabs, and separate route registration, permissions, and draft creation.
Keep input preparation, persistence, and response construction in clear logical groups.
Use `wp_insert_post` with error returns enabled and handle its `WP_Error` result directly.

The plugin bootstrap loads this route file before `rest_api_init`.

```php
<?php

function example_register_draft_routes() {
	register_rest_route(
		'example/v1',
		'/drafts',
		array(
			'methods'             => WP_REST_Server::CREATABLE,
			'callback'            => 'example_create_draft',
			'permission_callback' => 'example_can_create_draft',
			'args'                => array(
				'title'   => array(
					'required' => true,
					'type'     => 'string',
				),
				'content' => array(
					'default' => '',
					'type'    => 'string',
				),
			),
		)
	);
}

function example_can_create_draft() {
	return current_user_can( 'edit_posts' );
}

function example_create_draft( WP_REST_Request $request ) {
	$title   = sanitize_text_field( $request->get_param( 'title' ) );
	$content = wp_kses_post( $request->get_param( 'content' ) );
	if ( '' === $title ) {
		return new WP_Error(
			'invalid_title',
			'A title is required.',
			array( 'status' => 400 )
		);
	}

	$post_data = array(
		'post_title'   => $title,
		'post_content' => $content,
		'post_status'  => 'draft',
		'post_author'  => get_current_user_id(),
	);
	$post_id   = wp_insert_post( wp_slash( $post_data ), true );
	if ( is_wp_error( $post_id ) ) {
		return $post_id;
	}

	return new WP_REST_Response( array( 'id' => $post_id ), 201 );
}

add_action( 'rest_api_init', 'example_register_draft_routes' );
```

## REST tests

The WordPress test bootstrap loads the plugin before these tests.
The suite supplies `WP_UnitTestCase`, its user factory, and database fixture cleanup.
Tests dispatch through the REST server so permission callbacks and argument handling are exercised.

### Incorrect

Every test repeats server setup, user creation, request construction, and restoration of global state.
Setup, action, assertions, and cleanup lack logical-group spacing.

```php
<?php
class Example_Draft_Test extends WP_UnitTestCase {
	public function test_creates_a_draft() {
		global $wp_rest_server;
		$previous_user   = get_current_user_id();
		$previous_server = $wp_rest_server;
		try {
			$wp_rest_server = new WP_REST_Server();
			do_action( 'rest_api_init' );
			$user_id = self::factory()->user->create( array( 'role' => 'author' ) );
			wp_set_current_user( $user_id );
			$request = new WP_REST_Request( 'POST', '/example/v1/drafts' );
			$request->set_param( 'title', 'Workshop notes' );
			$request->set_param( 'content', "<p>Bring the team's notes.</p>" );
			$response = rest_do_request( $request );
			$this->assertSame( 201, $response->get_status() );
			$data = $response->get_data();
			$this->assertSame( 'draft', get_post_status( $data['id'] ) );
			$this->assertSame(
				get_current_user_id(),
				(int) get_post_field( 'post_author', $data['id'], 'raw' )
			);
			$this->assertSame(
				'Workshop notes',
				get_post_field( 'post_title', $data['id'], 'raw' )
			);
			$this->assertSame(
				"<p>Bring the team's notes.</p>",
				get_post_field( 'post_content', $data['id'], 'raw' )
			);
		} finally {
			wp_set_current_user( $previous_user );
			$wp_rest_server = $previous_server;
		}
	}
	public function test_rejects_an_empty_title() {
		global $wp_rest_server;
		$previous_user   = get_current_user_id();
		$previous_server = $wp_rest_server;
		try {
			$wp_rest_server = new WP_REST_Server();
			do_action( 'rest_api_init' );
			$user_id = self::factory()->user->create( array( 'role' => 'author' ) );
			wp_set_current_user( $user_id );
			$request = new WP_REST_Request( 'POST', '/example/v1/drafts' );
			$request->set_param( 'title', '' );
			$request->set_param( 'content', "" );
			$response = rest_do_request( $request );
			$this->assertSame( 400, $response->get_status() );
			$this->assertSame( 'invalid_title', $response->get_data()['code'] );
		} finally {
			wp_set_current_user( $previous_user );
			$wp_rest_server = $previous_server;
		}
	}
	public function test_rejects_a_subscriber() {
		global $wp_rest_server;
		$previous_user   = get_current_user_id();
		$previous_server = $wp_rest_server;
		try {
			$wp_rest_server = new WP_REST_Server();
			do_action( 'rest_api_init' );
			$user_id = self::factory()->user->create( array( 'role' => 'subscriber' ) );
			wp_set_current_user( $user_id );
			$request = new WP_REST_Request( 'POST', '/example/v1/drafts' );
			$request->set_param( 'title', 'Workshop notes' );
			$request->set_param( 'content', "" );
			$response = rest_do_request( $request );
			$this->assertSame( 403, $response->get_status() );
			$this->assertSame( array(), get_posts( array( 'post_status' => 'draft' ) ) );
		} finally {
			wp_set_current_user( $previous_user );
			$wp_rest_server = $previous_server;
		}
	}
}
```

### Correct

Use shared setup and teardown to restore the previous user and REST server.
Reuse request construction while keeping roles, input, and expected responses visible.
The fixture reuses the plugin's existing registration hook.
Test file and function size exemptions retain the common formatting and reuse requirements.

```php
<?php

class Example_Draft_Test extends WP_UnitTestCase {
	private $previous_user;
	private $previous_server;

	public function set_up() {
		parent::set_up();
		global $wp_rest_server;
		$this->previous_user   = get_current_user_id();
		$this->previous_server = $wp_rest_server;

		$wp_rest_server = new WP_REST_Server();
		do_action( 'rest_api_init' );
	}

	public function tear_down() {
		global $wp_rest_server;
		wp_set_current_user( $this->previous_user );
		$wp_rest_server = $this->previous_server;

		parent::tear_down();
	}

	public function test_creates_a_draft() {
		$request = $this->prepare_request(
			'author',
			'Workshop notes',
			"<p>Bring the team's notes.</p>"
		);

		$response = rest_do_request( $request );

		$this->assertSame( 201, $response->get_status() );

		$data = $response->get_data();
		$this->assertSame( 'draft', get_post_status( $data['id'] ) );
		$author_id = get_post_field(
			'post_author',
			$data['id'],
			'raw'
		);
		$this->assertSame( get_current_user_id(), (int) $author_id );
		$this->assertSame(
			'Workshop notes',
			get_post_field( 'post_title', $data['id'], 'raw' )
		);
		$this->assertSame(
			"<p>Bring the team's notes.</p>",
			get_post_field( 'post_content', $data['id'], 'raw' )
		);
	}

	public function test_rejects_an_empty_title() {
		$request = $this->prepare_request(
			'author',
			'',
			""
		);

		$response = rest_do_request( $request );

		$this->assertSame( 400, $response->get_status() );
		$this->assertSame(
			'invalid_title',
			$response->get_data()['code']
		);
	}

	public function test_rejects_a_subscriber() {
		$request = $this->prepare_request(
			'subscriber',
			'Workshop notes',
			""
		);

		$response = rest_do_request( $request );

		$this->assertSame( 403, $response->get_status() );
		$this->assertSame(
			array(),
			get_posts( array( 'post_status' => 'draft' ) )
		);
	}

	private function prepare_request( $role, $title, $content ) {
		$user_id = self::factory()->user->create(
			array( 'role' => $role )
		);
		wp_set_current_user( $user_id );

		$request = new WP_REST_Request( 'POST', '/example/v1/drafts' );
		$request->set_param( 'title', $title );
		$request->set_param( 'content', $content );

		return $request;
	}
}
```

## Verification

Run PHP syntax checks and the project's available WPCS checks.
With a WordPress test environment, dispatch authorized, invalid, and denied requests through the registered route.
Check draft status, author assignment, content preservation, error responses, and test cleanup.
Also verify an insertion failure using the project's existing test seam or fixture.
Identify any local checks performed with temporary WordPress stubs.
