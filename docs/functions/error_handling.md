# **Error Handling in Functions**

This section defines best practices for handling errors in Go functions.
Proper error handling improves debugging, prevents silent failures, and keeps error messages consistent across the codebase.

---

## Always Check and Handle Errors

Never ignore returned errors — handle them or propagate them.

**Do:**

```go
func SaveUser(ctx context.Context, user User) error {
    if err := userRepo.Insert(ctx, user); err != nil {
        return fmt.Errorf("SaveUser: failed to insert user: %w", err)
    }
    return nil
}
```

**Don’t:**

```go
func SaveUser(ctx context.Context, user User) error {
    userRepo.Insert(ctx, user) // Ignored error — may cause silent failure.
    return nil
}
```

---

## Add Context to Errors

Wrap errors with meaningful context before returning them to help debugging.

**Do:**

```go
func UpdateUser(ctx context.Context, id string, updates UserUpdate) error {
    if err := userRepo.Update(ctx, id, updates); err != nil {
        return fmt.Errorf("UpdateUser: failed to update user %s: %w", id, err)
    }
    return nil
}
```

**Don’t:**

```go
func UpdateUser(ctx context.Context, id string, updates UserUpdate) error {
    return err // No clue where or why it failed.
}
```

---

## Use Sentinel or Typed Errors for Known Conditions

Known conditions should be represented by constants or typed errors so they can be compared with `errors.Is`.

**Do:**

```go
var ErrUserNotFound = errors.New("user not found")

func GetUser(ctx context.Context, id string) (User, error) {
    user, err := userRepo.FindByID(ctx, id)
    if err == mongo.ErrNoDocuments {
        return User{}, ErrUserNotFound
    }
    return user, err
}

if errors.Is(err, ErrUserNotFound) {
    // Handle not found
}
```

**Don’t:**

```go
func GetUser(ctx context.Context, id string) (User, error) {
    return userRepo.FindByID(ctx, id) // Leaks DB-specific error to callers.
}
```

---

## Return Zero Values Alongside Errors

When returning an error, all other return values should be their zero values.

**Do:**

```go
func DeleteUser(ctx context.Context, id string) (bool, error) {
    if err := userRepo.DeleteByID(ctx, id); err != nil {
        return false, err
    }
    return true, nil
}
```

**Don’t:**

```go
func DeleteUser(ctx context.Context, id string) (bool, error) {
    if err := userRepo.DeleteByID(ctx, id); err != nil {
        return true, err // Misleading — deletion didn’t succeed.
    }
    return true, nil
}
```

---

## Fail Fast Inside Functions

Stop processing as soon as an unrecoverable error occurs.

**Do:**

```go
func SaveUserAndNotify(ctx context.Context, user User) error {
    if err := userRepo.Insert(ctx, user); err != nil {
        return fmt.Errorf("SaveUserAndNotify: insert failed: %w", err)
    }
    return emailService.Send(ctx, user.Email, "Welcome!")
}
```

**Don’t:**

```go
func SaveUserAndNotify(ctx context.Context, user User) error {
    err := userRepo.Insert(ctx, user)
    emailService.Send(ctx, user.Email, "Welcome!") // Still runs even if insert failed.
    return err
}
```

---

## Avoid Logging and Returning the Same Error

Log an error **or** return it — not both — to avoid duplicate logs.

**Do:**

```go
func SaveUser(ctx context.Context, user User) error {
    return fmt.Errorf("SaveUser: failed to save: %w", err)
}
```

*Log at the caller level.*

**Don’t:**

```go
func SaveUser(ctx context.Context, user User) error {
    log.Printf("SaveUser failed: %v", err)
    return err
}
// Causes duplicate logs when caller also logs the error.
```

---

## Don’t Use Panics for Normal Errors

Reserve panics for truly exceptional situations (e.g., programmer errors), not normal operational errors.

**Do:**

```go
func GetUser(ctx context.Context, id string) (User, error) {
    if id == "" {
        return User{}, errors.New("empty user ID")
    }
    return userRepo.FindByID(ctx, id)
}
```

**Don’t:**

```go
func GetUser(ctx context.Context, id string) (User, error) {
    if id == "" {
        panic("empty user ID") // Not an exceptional crash-worthy condition.
    }
    return userRepo.FindByID(ctx, id)
}
```

---

## Match Error Handling to Caller’s Responsibility

Return errors to higher layers for decision-making rather than deciding inside low-level functions.

**Do:**

```go
func DeleteUser(ctx context.Context, id string) error {
    return userRepo.DeleteByID(ctx, id)
}

// Caller decides how to handle:
if err := DeleteUser(ctx, id); errors.Is(err, ErrUserNotFound) {
    return httpError(http.StatusNotFound, "user not found")
}
```

**Don’t:**

```go
func DeleteUser(ctx context.Context, id string) error {
    if err := userRepo.DeleteByID(ctx, id); err != nil {
        log.Fatal(err) // Kills the entire app instead of letting caller handle.
    }
    return nil
}
```