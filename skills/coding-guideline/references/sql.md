# SQL Formatting

Read [common rules](common.md) and the host-language reference for embedded queries.
Use the same line-length and production size limits as other languages.
Test queries retain formatting and reuse requirements while receiving test size exemptions.

Put clauses, selected fields, inserted columns, and bound values on separate indented lines.
Use the host language's multiline string form for embedded queries and preserve significant whitespace.
Follow the project's SQL dialect and formatter configuration.

## Insert a signup user

The application hashes the password before executing this query.
Parameters provide the user's name, email, and password hash.
The examples retain the existing PostgreSQL-style positional placeholders.

### Incorrect

```sql
INSERT INTO users (name, email, password_hash) VALUES ($1, $2, $3);
```

### Correct

```sql
INSERT INTO users (
  name,
  email,
  password_hash
)
VALUES (
  $1,
  $2,
  $3
);
```

## Get a user by email

Retrieve public user fields using a bound email parameter.
Both layouts return the same fields.

### Incorrect

```sql
SELECT u.id, u.name, u.email FROM users AS u WHERE u.email = $1;
```

### Correct

```sql
SELECT
  u.id,
  u.name,
  u.email
FROM users AS u
WHERE u.email = $1;
```

## SQL inside TypeScript

The repository uses this query for the same signup operation.
Keep values separate from the query through the database client's parameter binding.

### Incorrect

```typescript
const query = "INSERT INTO users (name, email, password_hash) VALUES ($1, $2, $3);";
```

### Correct

```typescript
const query = `
  INSERT INTO users (
    name,
    email,
    password_hash
  )
  VALUES (
    $1,
    $2,
    $3
  );
`;
```
