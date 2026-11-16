# PHP Formatting Examples

Read [common rules](common.md) for length limits, logical groups, helper reuse, and test exemptions.
Follow the project's PHP formatter and conventions.
For WordPress code, also read [WordPress PHP](wordpress-php.md) and follow its platform conventions.

## Production code

The handler parses signup input, validates it, hashes a password, inserts a user, and returns a response.
The imports under `Example\Signup` are dummy project dependencies.
`SignupInput::fromJson` parses the request and checks field types.
It wraps JSON parsing and input-shape failures in `InvalidSignupInput`.
Signup-specific validation checks required fields and the bcrypt password length limit.
Expected input failures use `InvalidSignupInput`.
Hashing and repository failures retain their original exception types.
`UserRepository::insert` stores the user or throws an error.
The response exposes its HTTP status through `status`.
Both versions preserve these contracts.

### Incorrect

The handler spans multiple lines without separating logical groups.
The local hashing helper occupies one line immediately after the handler.

```php
<?php

namespace Example\Signup;

use Example\Signup\Database;
use Example\Signup\Http\Request;
use Example\Signup\Http\Response;
use Example\Signup\Model\SignupInput;
use Example\Signup\Model\User;
use Example\Signup\Repo\UserRepository;
use Example\Signup\Validation\InvalidSignupInput;
use Throwable;

use function Example\Signup\Validation\validateSignup;

function signup(Database $db, Request $request): Response
{
    try {
        $input = SignupInput::fromJson($request->body);
        validateSignup($input);
        $hash = hashPassword($input->password);
        $user = new User($input->name, $input->email, $hash);
        UserRepository::insert($db, $user);
        return new Response(201);
    } catch (InvalidSignupInput) {
        return new Response(400);
    } catch (Throwable) {
        return new Response(500);
    }
}
function hashPassword(string $password): string { return password_hash($password, PASSWORD_BCRYPT); }
```

### Correct

Group the signup flow in one `try` and handle expected input failures before the general `Throwable` catch.
Separate parsing and validation, hashing, persistence, and response construction with empty lines.
Import the reusable password policy from its shared module.

The handler lives in `Signup/signup.php`.

```php
<?php

namespace Example\Signup;

use Example\Signup\Database;
use Example\Signup\Http\Request;
use Example\Signup\Http\Response;
use Example\Signup\Model\SignupInput;
use Example\Signup\Model\User;
use Example\Signup\Repo\UserRepository;
use Example\Signup\Validation\InvalidSignupInput;
use Throwable;

use function Example\Signup\Password\hashPassword;
use function Example\Signup\Validation\validateSignup;

function signup(Database $db, Request $request): Response
{
    try {
        $input = SignupInput::fromJson($request->body);
        validateSignup($input);

        $hash = hashPassword($input->password);

        $user = new User($input->name, $input->email, $hash);
        UserRepository::insert($db, $user);

        return new Response(201);
    } catch (InvalidSignupInput) {
        return new Response(400);
    } catch (Throwable) {
        return new Response(500);
    }
}
```

The shared helper lives in `Password/hashPassword.php`.

```php
<?php

namespace Example\Signup\Password;

function hashPassword(string $password): string
{
    return password_hash($password, PASSWORD_BCRYPT);
}
```

## Test code

Both cases exercise the same production handler.
The dummy `TestDatabase` opens an isolated database, reads stored users, and closes resources.
Native PHP assertions illustrate the checks; use the project's test framework in application code.

### Incorrect

The cases repeat database setup, request construction, and cleanup.
Setup, action, and assertions lack logical-group spacing.

```php
<?php

namespace Example\Signup;

use Example\Signup\Database;
use Example\Signup\Http\Request;
use Example\Signup\TestUtil\TestDatabase;

function testSignup(): void
{
    $input = [
        'name' => 'Alex Morgan',
        'email' => 'alex@example.com',
        'password' => 'example-password',
    ];
    $db = TestDatabase::open();
    try {
        $request = new Request(json_encode($input, JSON_THROW_ON_ERROR));
        $response = signup($db, $request);
        assert($response->status === 201);
        $users = TestDatabase::users($db);
        assert(count($users) === 1);
        assert($users[0]->name === 'Alex Morgan');
        assert($users[0]->email === 'alex@example.com');
        assert(password_verify(
            'example-password',
            $users[0]->passwordHash,
        ));
    } finally {
        TestDatabase::close($db);
    }
}

function testSignupRejectsInvalidInput(): void
{
    $input = [
        'name' => '',
        'email' => 'alex@example.com',
        'password' => 'example-password',
    ];
    $db = TestDatabase::open();
    try {
        $request = new Request(json_encode($input, JSON_THROW_ON_ERROR));
        $response = signup($db, $request);
        assert($response->status === 400);
        assert(TestDatabase::users($db) === []);
    } finally {
        TestDatabase::close($db);
    }
}
```

### Correct

Reuse `withSignupTest` for shared setup and guaranteed teardown.
Keep each case's input and expected outcomes visible.
Test files and functions retain line-length and formatting rules while receiving size exemptions.

```php
<?php

namespace Example\Signup;

use Example\Signup\Database;
use Example\Signup\Http\Request;
use Example\Signup\TestUtil\TestDatabase;

function testSignup(): void
{
    $input = [
        'name' => 'Alex Morgan',
        'email' => 'alex@example.com',
        'password' => 'example-password',
    ];

    withSignupTest($input, function (Database $db, Request $request): void {
        $response = signup($db, $request);

        assert($response->status === 201);

        $users = TestDatabase::users($db);
        assert(count($users) === 1);
        assert($users[0]->name === 'Alex Morgan');
        assert($users[0]->email === 'alex@example.com');

        assert(password_verify(
            'example-password',
            $users[0]->passwordHash,
        ));
    });
}

function testSignupRejectsInvalidInput(): void
{
    $input = [
        'name' => '',
        'email' => 'alex@example.com',
        'password' => 'example-password',
    ];

    withSignupTest($input, function (Database $db, Request $request): void {
        $response = signup($db, $request);

        assert($response->status === 400);
        assert(TestDatabase::users($db) === []);
    });
}

function withSignupTest(array $input, callable $run): void
{
    $db = TestDatabase::open();
    try {
        $request = new Request(json_encode($input, JSON_THROW_ON_ERROR));

        $run($db, $request);
    } finally {
        TestDatabase::close($db);
    }
}
```

For query strings, also read [SQL formatting](sql.md) and preserve interpolation behavior in multiline strings.
