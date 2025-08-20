# **Parameter & Return Value Guidelines**

This section defines standards for function parameters and return values to ensure readability, maintainability, and consistency across the codebase.

---
<br>

## Always Pass `context.Context` as the First Parameter

> Order parameter based on their declaration ex: here Context should always be the first parameter to make cancellation, timeouts, and tracing consistent across all functions.

**Do:**

```go
func getUser(ctx context.Context, id string) (User, error) {
    return userRepo.FindByID(ctx, id)
}
```

**Don’t:**

```go
func getUser(id string, ctx context.Context) (User, error) {
    // Non-standard order leads to inconsistency.
    return userRepo.FindByID(ctx, id)
}
```

---
<br>

## Avoid Too Many Parameters

If more than 3–4 parameters are needed, group them into a struct to improve readability and flexibility.

**Do:**

```go
type UpdateUserParams struct {
    ID      string
    Name    string
    Email   string
    IsAdmin bool
}

func updateUser(ctx context.Context, params UpdateUserParams) error {
    return userRepo.Update(ctx, params.ID, params.Name, params.Email, params.IsAdmin)
}
```

**Don’t:**

```go
func updateUser(ctx context.Context, id, name, email string, isAdmin bool) error {
    // Too many positional parameters can cause confusion.
    return userRepo.Update(ctx, id, name, email, isAdmin)
}
```

---
<br>

## Use Appropriate Parameter Types

Avoid passing generic types when a more specific type is available.
Use `string` for text, `int64` for IDs if needed, and typed enums for predefined sets.

**Do:**

```go
type UserStatus string

const (
    StatusActive   UserStatus = "active"
    StatusInactive UserStatus = "inactive"
)

func setUserStatus(ctx context.Context, userID string, status UserStatus) error {
    return userRepo.UpdateStatus(ctx, userID, string(status))
}
```

**Don’t:**

```go
func setUserStatus(ctx context.Context, userID string, status string) error {
    // Accepting any string increases risk of invalid values.
    return userRepo.UpdateStatus(ctx, userID, status)
}
```

---
<br>

## Return Multiple Values Instead of Pointers for Zero Values

If returning a zero value is valid, return it directly instead of a nil pointer.

**Do:**

```go
func findUser(ctx context.Context, id string) (User, error) {
    user, err := userRepo.FindByID(ctx, id)
    if err != nil {
        return User{}, err
    }
    return user, nil
}
```

**Don’t:**

```go
func findUser(ctx context.Context, id string) (*User, error) {
    // Using pointer forces extra nil checks in calling code unnecessarily.
    return &User{}, nil
}
```

---
<br>

## Return Specific Error Values

Return typed or sentinel errors so the caller can handle them appropriately.

**Do:**

```go
var ErrUserNotFound = errors.New("user not found")

func getUser(ctx context.Context, id string) (User, error) {
    user, err := userRepo.FindByID(ctx, id)
    if err == mongo.ErrNoDocuments {
        return User{}, ErrUserNotFound
    }
    return user, err
}
```

**Don’t:**

```go
func getUser(ctx context.Context, id string) (User, error) {
    return userRepo.FindByID(ctx, id) // Passing DB-specific error directly leaks implementation details.
}
```

---
<br>

## Use Named Return Values for Clarity (When Appropriate)

> Named return values improve readability in short functions but should not be overused.

**Do:**

```go
func countActiveUsers(ctx context.Context) (count int, err error) {
    count, err = userRepo.CountByStatus(ctx, "active")
    return
}
```

**Don’t:**

```go
func countActiveUsers(ctx context.Context) (int, error) {
    return userRepo.CountByStatus(ctx, "active")
}
// Named return values are not necessary here since function is short and obvious.
```

---
<br>

## Always Return Zero Values with Errors

When an error occurs, return the zero value for all other return values to avoid undefined state.

**Do:**

```go
func deleteUser(ctx context.Context, id string) (bool, error) {
    if err := userRepo.DeleteByID(ctx, id); err != nil {
        return false, err
    }
    return true, nil
}
```

**Don’t:**

```go
func deleteUser(ctx context.Context, id string) (bool, error) {
    if err := userRepo.DeleteByID(ctx, id); err != nil {
        return true, err // Misleading: true suggests deletion happened.
    }
    return true, nil
}
```

---
<br>

## Avoid Using `interface{}/any` as Parameter or Return Type

> Use generics or concrete types for better type safety.

**Do:**

```go
func getUserEmails(ctx context.Context, userIDs []string) ([]string, error) {
    return emailRepo.GetByUserIDs(ctx, userIDs)
}
```

**Don’t:**

```go
func getUserEmails(ctx context.Context, ids interface{}) (interface{}, error) {
    // Loses compile-time type safety and increases runtime checks.
    return nil, nil
}
```

---
<br>

## Order Return Values Consistently

When returning multiple values:

1. Data comes first.
2. Error comes last.

**Do:**

```go
func getUser(ctx context.Context, id string) (User, error) { ... }
```

**Don’t:**

```go
func getUser(ctx context.Context, id string) (error, User) { ... }
```

