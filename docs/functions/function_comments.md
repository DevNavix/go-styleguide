# **Function Comments Guidelines**

This section defines how to write clear, consistent, and useful comments for functions in Go.
Following these rules ensures that code is self-explanatory to both current and future developers.

---
<br>

## Follow Go’s `godoc` Style

Start the comment with the function name, followed by a concise description of its purpose.
The first sentence should be a summary; further details can follow.

**Do:**

```go
// SaveUser inserts a new user into the database.
// It returns an error if the user already exists.
func SaveUser(ctx context.Context, user User) error { ... }
```

**Don’t:**

```go
// This function saves the user into DB and if already exists it gives error.
func SaveUser(ctx context.Context, user User) error { ... }

// save user to db
func SaveUser(ctx context.Context, user User) error { ... }
```

---
<br>


## Document Side Effects

If a function triggers side effects like sending emails or modifying global state, document them.

**Do:**

```go
// DeleteUser removes the user from the database.
// It also sends an account deletion email if the user has a registered email address.
func DeleteUser(ctx context.Context, id string) error { ... }
```

**Don’t:**

```go
// DeleteUser removes the user from the database.
func DeleteUser(ctx context.Context, id string) error { ... }
// (Caller won’t know about the email being sent.)
```

---
<br>

## Keep Comments Updated

Outdated comments are worse than no comments — they mislead developers.

**Do:**

```go
// GetUser retrieves a user by ID. Returns ErrUserNotFound if not found.
func GetUser(ctx context.Context, id string) (User, error) { ... }
```

**Don’t:**

```go
// GetUser retrieves a user by ID and all their orders.
// (Function no longer returns orders, but comment was not updated.)
func GetUser(ctx context.Context, id string) (User, error) { ... }
```

---
<br>

## Avoid Redundant Comments

Don’t restate what is already obvious from the code.

**Do:**

```go
// CountActiveUsers returns the number of users with status "active".
func CountActiveUsers(ctx context.Context) (int, error) { ... }
```

**Don’t:**

```go
// CountActiveUsers counts active users.
func CountActiveUsers(ctx context.Context) (int, error) { ... }
```

---
<br>

## Use Complete Sentences

Write comments in full sentences with proper punctuation — they should read like documentation, not code notes.

**Do:**

```go
// Authenticate verifies the user’s credentials and returns a token if valid.
func Authenticate(ctx context.Context, email, password string) (string, error) { ... }
```

**Don’t:**

```go
// authenticate user and return token
func Authenticate(ctx context.Context, email, password string) (string, error) { ... }
```

---
<br>

## Mention Error Conditions

Callers should know what kind of errors to expect from a function.

**Do:**

```go
// FindUser returns the user with the given ID.
// Returns ErrUserNotFound if no user exists with the given ID.
func FindUser(ctx context.Context, id string) (User, error) { ... }
```

**Don’t:**

```go
// FindUser returns the user with the given ID.
func FindUser(ctx context.Context, id string) (User, error) { ... }
```
