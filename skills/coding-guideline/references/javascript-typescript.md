# JavaScript and TypeScript Formatting Examples

Read [common rules](common.md) for length limits, logical groups, helper reuse, and test exemptions.
Follow the project's JavaScript or TypeScript formatter and conventions.

## Production code

The signup handler parses input, validates it, hashes a password, inserts a user, and returns an HTTP response.
The `@signup/...` imports are dummy application paths.
The model module supplies `Database`, `SignupInput`, and `User`.
The signup-specific `parseSignupInput(request)` helper parses JSON and validates the input, returning a promise containing the validated signup input.
Expected parsing and validation failures use the imported `InvalidSignupInput` error class.
The helper supplies safe client-facing messages for those expected failures.
The repository exports `insertUser(db, user)`, which rejects on failure.
The crypto module supplies a random salt, asynchronous password-key derivation, and password verification.
Both versions use the same contracts.

### Incorrect

The multiline handler lacks empty lines between logical groups.
The hashing helper occupies one line immediately after the handler.

```javascript
import { insertUser } from "@signup/repo";
import {
  InvalidSignupInput,
  parseSignupInput,
} from "@signup/signup-validation";
import { createSalt, deriveKey } from "@signup/crypto";

export async function signup(db, request) {
  try {
    const input = await parseSignupInput(request);
    const passwordHash = await hashPassword(input.password);
    const user = {
      name: input.name,
      email: input.email,
      passwordHash,
    };
    await insertUser(db, user);
    return new Response(null, { status: 201 });
  } catch (error) {
    if (error instanceof InvalidSignupInput) {
      return new Response(error.message, { status: 400 });
    }
    return new Response("signup failed", { status: 500 });
  }
}
async function hashPassword(password) { const salt = createSalt(); const key = await deriveKey(password, salt); return `${salt}:${key}`; }
```

### Correct

Use one `try` for signup and narrow the caught error with `instanceof` before mapping input failures to HTTP 400.
Other failures return HTTP 500.
Keep short signatures on one line and separate logical groups.
Import reusable hashing from the shared password module.

The shared handler lives in `signup.js`.

```javascript
import { insertUser } from "@signup/repo";
import {
  InvalidSignupInput,
  parseSignupInput,
} from "@signup/signup-validation";
import { hashPassword } from "@signup/password";

export async function signup(db, request) {
  try {
    const input = await parseSignupInput(request);

    const passwordHash = await hashPassword(input.password);

    const user = {
      name: input.name,
      email: input.email,
      passwordHash,
    };
    await insertUser(db, user);

    return new Response(null, { status: 201 });
  } catch (error) {
    if (error instanceof InvalidSignupInput) {
      return new Response(error.message, { status: 400 });
    }

    return new Response("signup failed", { status: 500 });
  }
}
```

The shared helper lives in `password.js`, exported through `@signup/password`.
The crypto module owns the password-key derivation settings.

```javascript
import { createSalt, deriveKey } from "@signup/crypto";

export async function hashPassword(password) {
  const salt = createSalt();
  const key = await deriveKey(password, salt);

  return `${salt}:${key}`;
}
```

## Test code

Both cases call the same production handler using the platform's `Request` and `Response` APIs.
The dummy database fixture exposes inserted users through `users` and provides explicit cleanup.
Use the project's existing test framework; this example uses `node:test`.

### Incorrect

The tests repeat database setup, cleanup registration, and request construction.
Setup, action, and assertion groups have no empty-line separation.

```javascript
import assert from "node:assert/strict";
import { test } from "node:test";

import { verifyPassword } from "@signup/crypto";
import { closeTestDatabase, openTestDatabase } from "@signup/testutil";
import { signup } from "./signup.js";

test("creates a user", async (t) => {
  const db = openTestDatabase();
  t.after(() => closeTestDatabase(db));
  const input = {
    name: "Alex Morgan",
    email: "alex@example.com",
    password: "example-password",
  };
  const request = new Request("https://example.test/signup", {
    method: "POST",
    body: JSON.stringify(input),
  });
  const response = await signup(db, request);
  assert.equal(response.status, 201);
  const users = db.users;
  assert.equal(users.length, 1);
  assert.equal(users[0].name, "Alex Morgan");
  assert.equal(users[0].email, "alex@example.com");
  assert.equal(
    await verifyPassword("example-password", users[0].passwordHash),
    true,
  );
});

test("rejects invalid input", async (t) => {
  const db = openTestDatabase();
  t.after(() => closeTestDatabase(db));
  const input = {
    name: "",
    email: "alex@example.com",
    password: "example-password",
  };
  const request = new Request("https://example.test/signup", {
    method: "POST",
    body: JSON.stringify(input),
  });
  const response = await signup(db, request);
  assert.equal(response.status, 400);
  assert.equal(db.users.length, 0);
});
```

### Correct

Use `newSignupTest` for repeated setup and preserve cleanup through `t.after`.
Each test keeps its input, action, and expected outcomes visible.
Test files and functions receive size exemptions while retaining formatting and line-length rules.

```javascript
import assert from "node:assert/strict";
import { test } from "node:test";

import { verifyPassword } from "@signup/crypto";
import { closeTestDatabase, openTestDatabase } from "@signup/testutil";
import { signup } from "./signup.js";

test("creates a user", async (t) => {
  const { db, request } = newSignupTest(t, {
    name: "Alex Morgan",
    email: "alex@example.com",
    password: "example-password",
  });

  const response = await signup(db, request);

  assert.equal(response.status, 201);

  const users = db.users;
  assert.equal(users.length, 1);
  assert.equal(users[0].name, "Alex Morgan");
  assert.equal(users[0].email, "alex@example.com");

  assert.equal(
    await verifyPassword("example-password", users[0].passwordHash),
    true,
  );
});

test("rejects invalid input", async (t) => {
  const { db, request } = newSignupTest(t, {
    name: "",
    email: "alex@example.com",
    password: "example-password",
  });

  const response = await signup(db, request);

  assert.equal(response.status, 400);
  assert.equal(db.users.length, 0);
});

function newSignupTest(t, input) {
  const db = openTestDatabase();
  t.after(() => closeTestDatabase(db));

  const request = new Request("https://example.test/signup", {
    method: "POST",
    body: JSON.stringify(input),
  });

  return { db, request };
}
```

For embedded queries, also read [SQL formatting](sql.md).

## TypeScript additions

The shared runtime examples also apply to TypeScript.
Use `.ts` source files and add type contracts through type-only imports where needed.
Apply these contracts to the same handlers, dependencies, and test fixture helpers.
Runtime validation still checks external data before the handler uses it.
The shared `instanceof InvalidSignupInput` check narrows caught errors in both languages.

```typescript
import type { TestContext } from "node:test";
import type { Database, SignupInput, User } from "@signup/model";

export type ParseSignupInput = (request: Request) => Promise<SignupInput>;

export type InsertUser = (db: Database, user: User) => Promise<void>;
export type HashPassword = (password: string) => Promise<string>;
export type Signup = (db: Database, request: Request) => Promise<Response>;

export type SignupTest = (
  test: TestContext,
  input: SignupInput,
) => { db: Database; request: Request };
```

Follow the project's module resolution and test framework for source and import extensions.
