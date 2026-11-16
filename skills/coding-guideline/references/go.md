# Go Formatting Examples

Read [common rules](common.md) for length limits, logical groups, helper reuse, and test exemptions.
Use `gofmt` and the project's Go conventions.

## Production code

The signup handler parses input, validates it, hashes a password, inserts a user, and returns an HTTP response.
The examples share the same behavior.

Imports under `example.com/signup/internal/...` are dummy application paths.
The model package supplies `SignupInput` with `Name`, `Email`, and `Password`.
It supplies `User` with `Name`, `Email`, and `PasswordHash`.
The signup-specific validation package checks required fields and the bcrypt password length limit.
The repository supplies `InsertUser(context.Context, *sql.DB, model.User) error`.

### Incorrect

The multiline handler lacks empty lines between logical groups.
The nontrivial hashing helper occupies one line immediately after the handler.

```go
package signup

import (
	"database/sql"
	"encoding/json"
	"net/http"

	"example.com/signup/internal/model"
	"example.com/signup/internal/repo"
	"example.com/signup/internal/signup/validation"
	"golang.org/x/crypto/bcrypt"
)

func signup(db *sql.DB, response http.ResponseWriter, request *http.Request) {
	var input model.SignupInput
	if err := json.NewDecoder(request.Body).Decode(&input); err != nil {
		http.Error(response, "invalid request body", http.StatusBadRequest)
		return
	}
	if err := validation.Signup(input); err != nil {
		http.Error(response, err.Error(), http.StatusBadRequest)
		return
	}
	hash, err := hashPassword(input.Password)
	if err != nil {
		http.Error(response, "signup failed", http.StatusInternalServerError)
		return
	}
	user := model.User{
		Name:         input.Name,
		Email:        input.Email,
		PasswordHash: hash,
	}
	if err := repo.InsertUser(request.Context(), db, user); err != nil {
		http.Error(response, "signup failed", http.StatusInternalServerError)
		return
	}
	response.WriteHeader(http.StatusCreated)
}
func hashPassword(password string) (string, error) { hash, err := bcrypt.GenerateFromPassword([]byte(password), bcrypt.DefaultCost); if err != nil { return "", err }; return string(hash), nil }
```

### Correct

Keep the short handler signature on one line and separate logical groups inside its body.
The handler imports reusable hashing from the shared password module.

The handler lives in `signup/handler.go`.

```go
package signup

import (
	"database/sql"
	"encoding/json"
	"net/http"

	"example.com/signup/internal/model"
	"example.com/signup/internal/password"
	"example.com/signup/internal/repo"
	"example.com/signup/internal/signup/validation"
)

func signup(db *sql.DB, response http.ResponseWriter, request *http.Request) {
	var input model.SignupInput
	if err := json.NewDecoder(request.Body).Decode(&input); err != nil {
		http.Error(
			response,
			"invalid request body",
			http.StatusBadRequest,
		)
		return
	}

	if err := validation.Signup(input); err != nil {
		http.Error(response, err.Error(), http.StatusBadRequest)
		return
	}

	hash, err := password.Hash(input.Password)
	if err != nil {
		http.Error(
			response,
			"signup failed",
			http.StatusInternalServerError,
		)
		return
	}

	user := model.User{
		Name:         input.Name,
		Email:        input.Email,
		PasswordHash: hash,
	}
	if err := repo.InsertUser(request.Context(), db, user); err != nil {
		http.Error(
			response,
			"signup failed",
			http.StatusInternalServerError,
		)
		return
	}

	response.WriteHeader(http.StatusCreated)
}
```

The shared helper lives in `internal/password/hash.go`.

```go
package password

import "golang.org/x/crypto/bcrypt"

func Hash(password string) (string, error) {
	hash, err := bcrypt.GenerateFromPassword(
		[]byte(password),
		bcrypt.DefaultCost,
	)
	if err != nil {
		return "", err
	}

	return string(hash), nil
}
```

## Test code

Both tests call the same production handler.
The dummy test utility opens an isolated database, reads inserted users, and closes the database.
The success case checks the stored user and verifies the password hash.
The invalid-input case checks the response and absence of inserted users.

### Incorrect

Both tests repeat database setup, cleanup registration, and request construction.
Setup, action, and assertions lack logical-group spacing.

```go
package signup

import (
	"net/http"
	"net/http/httptest"
	"strings"
	"testing"

	"example.com/signup/internal/testutil"
	"golang.org/x/crypto/bcrypt"
)

func TestSignup(t *testing.T) {
	db := testutil.NewDB(t)
	t.Cleanup(func() { testutil.CloseDB(t, db) })
	request := httptest.NewRequest(
		http.MethodPost,
		"/signup",
		strings.NewReader(`{
  "name": "Alex Morgan",
  "email": "alex@example.com",
  "password": "example-password"
}`),
	)
	response := httptest.NewRecorder()
	signup(db, response, request)
	if response.Code != http.StatusCreated {
		t.Fatalf("status = %d, want %d", response.Code, http.StatusCreated)
	}
	users := testutil.InsertedUsers(t, db)
	if len(users) != 1 {
		t.Fatalf("inserted users = %d, want 1", len(users))
	}
	if users[0].Name != "Alex Morgan" || users[0].Email != "alex@example.com" {
		t.Fatalf("unexpected user: %+v", users[0])
	}
	err := bcrypt.CompareHashAndPassword(
		[]byte(users[0].PasswordHash),
		[]byte("example-password"),
	)
	if err != nil {
		t.Fatalf("stored password hash does not match: %v", err)
	}
}
func TestSignupRejectsInvalidInput(t *testing.T) {
	db := testutil.NewDB(t)
	t.Cleanup(func() { testutil.CloseDB(t, db) })
	request := httptest.NewRequest(
		http.MethodPost,
		"/signup",
		strings.NewReader(`{
  "name": "",
  "email": "alex@example.com",
  "password": "example-password"
}`),
	)
	response := httptest.NewRecorder()
	signup(db, response, request)
	if response.Code != http.StatusBadRequest {
		t.Fatalf("status = %d, want %d", response.Code, http.StatusBadRequest)
	}
	if users := testutil.InsertedUsers(t, db); len(users) != 0 {
		t.Fatalf("inserted users = %d, want 0", len(users))
	}
}
```

### Correct

Extract repeated setup into `newSignupTest` and preserve cleanup through `t.Cleanup`.
Keep each case's input, signup call, and expected outcomes visible.
Test functions and helpers retain line-length and formatting rules while receiving file and function size exemptions.

```go
package signup

import (
	"database/sql"
	"net/http"
	"net/http/httptest"
	"strings"
	"testing"

	"example.com/signup/internal/testutil"
	"golang.org/x/crypto/bcrypt"
)

func TestSignup(t *testing.T) {
	db, response, request := newSignupTest(t, `{
  "name": "Alex Morgan",
  "email": "alex@example.com",
  "password": "example-password"
}`)

	signup(db, response, request)

	if response.Code != http.StatusCreated {
		t.Fatalf(
			"status = %d, want %d",
			response.Code,
			http.StatusCreated,
		)
	}

	users := testutil.InsertedUsers(t, db)
	if len(users) != 1 {
		t.Fatalf("inserted users = %d, want 1", len(users))
	}
	if users[0].Name != "Alex Morgan" {
		t.Fatalf("unexpected user: %+v", users[0])
	}
	if users[0].Email != "alex@example.com" {
		t.Fatalf("unexpected user: %+v", users[0])
	}

	err := bcrypt.CompareHashAndPassword(
		[]byte(users[0].PasswordHash),
		[]byte("example-password"),
	)
	if err != nil {
		t.Fatalf("stored password hash does not match: %v", err)
	}
}

func TestSignupRejectsInvalidInput(t *testing.T) {
	db, response, request := newSignupTest(t, `{
  "name": "",
  "email": "alex@example.com",
  "password": "example-password"
}`)

	signup(db, response, request)

	if response.Code != http.StatusBadRequest {
		t.Fatalf(
			"status = %d, want %d",
			response.Code,
			http.StatusBadRequest,
		)
	}
	if users := testutil.InsertedUsers(t, db); len(users) != 0 {
		t.Fatalf("inserted users = %d, want 0", len(users))
	}
}

func newSignupTest(t *testing.T, body string) (
	*sql.DB,
	*httptest.ResponseRecorder,
	*http.Request,
) {
	t.Helper()
	db := testutil.NewDB(t)
	t.Cleanup(func() { testutil.CloseDB(t, db) })

	request := httptest.NewRequest(
		http.MethodPost,
		"/signup",
		strings.NewReader(body),
	)
	response := httptest.NewRecorder()

	return db, response, request
}
```

For embedded queries, also read [SQL formatting](sql.md).
