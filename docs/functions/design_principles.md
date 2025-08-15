
# **Function Design Principles**

This section outlines the recommended best practices for designing functions in Go to ensure readability, maintainability, and scalability across the codebase.

---

## Focused and Single-Purpose (SRP)

A function should do **one thing and do it well**. Avoid mixing multiple responsibilities.

**Do:**

```go
// saveUser inserts a new user into the collection.
func saveUser(ctx context.Context, user User) error {
    return userRepo.Insert(ctx, user)
}

// sendWelcomeEmail sends a welcome email to a new user.
func sendWelcomeEmail(ctx context.Context, email string) error {
    return emailService.Send(ctx, email, "Welcome!")
}
```

**Don’t:**

```go
// saveUserAndSendEmail both saves the user and sends an email.
// This couples database logic with email logic, making it harder to maintain.
func saveUserAndSendEmail(ctx context.Context, user User) error {
    if err := userRepo.Insert(ctx, user); err != nil {
        return err
    }
    return emailService.Send(ctx, user.Email, "Welcome!")
}
```

---

##  Limit Function Size

A function should be short and fit within a single screen (\~20–30 lines). Break down large functions into smaller, reusable ones.

**Do:**

```go
func updateUser(ctx context.Context, userID string, updates UserUpdate) error {
    user, err := userRepo.FindByID(ctx, userID)
    if err != nil {
        return err
    }
    applyUserUpdates(&user, updates)
    return userRepo.Update(ctx, user)
}

func applyUserUpdates(user *User, updates UserUpdate) {
    if updates.Name != "" {
        user.Name = updates.Name
    }
    if updates.Email != "" {
        user.Email = updates.Email
    }
}
```

**Don’t:**

```go
func updateUser(ctx context.Context, userID string, updates UserUpdate) error {
    user, err := userRepo.FindByID(ctx, userID)
    if err != nil {
        return err
    }
    if updates.Name != "" {
        user.Name = updates.Name
    }
    if updates.Email != "" {
        user.Email = updates.Email
    }
    if err := userRepo.Update(ctx, user); err != nil {
        return err
    }
    return nil
}
```

*(This isn’t terrible, but as complexity grows, logic should be split out.)*

---

##  Use Clear and Consistent Parameter Ordering

For clarity:

1. **Context** comes first (`context.Context`).
2. **Inputs** come next.
3. **Outputs or callbacks** come last.

**Do:**

```go
func deleteUser(ctx context.Context, userID string) error {
    return userRepo.DeleteByID(ctx, userID)
}
```

**Don’t:**

```go
func deleteUser(userID string, ctx context.Context) error {
    // Non-standard parameter order makes it harder to follow.
    return userRepo.DeleteByID(ctx, userID)
}
```

---

##  Avoid Side Effects Without Clear Indication

A function should not modify external state unless clearly intended.

**Do:**

```go
func deactivateUser(ctx context.Context, userID string) error {
    return userRepo.UpdateStatus(ctx, userID, "inactive")
}
```

**Don’t:**

```go
func deactivateUser(ctx context.Context, userID string) error {
    globalConfig.LastModifiedUserID = userID // Hidden side effect
    return userRepo.UpdateStatus(ctx, userID, "inactive")
}
```

---

##  Prefer Explicit Over Implicit Behavior

Avoid `magic` inside functions. Callers should understand exactly what happens.

**Do:**

```go
func saveUser(ctx context.Context, user User, sendEmail bool) error {
    if err := userRepo.Insert(ctx, user); err != nil {
        return err
    }
    if sendEmail {
        return emailService.Send(ctx, user.Email, "Welcome!")
    }
    return nil
}
```

**Don’t:**

```go
func saveUser(ctx context.Context, user User) error {
    if err := userRepo.Insert(ctx, user); err != nil {
        return err
    }
    // Implicit behavior: Always sends email without caller’s control.
    return emailService.Send(ctx, user.Email, "Welcome!")
}
```

---

##  Minimize Parameters Where Possible

If a function has too many parameters, consider using a struct to group related ones.

**Do:**

```go
type SaveUserParams struct {
    User       User
    SendEmail  bool
    NotifyAdmin bool
}

func saveUser(ctx context.Context, params SaveUserParams) error {
    if err := userRepo.Insert(ctx, params.User); err != nil {
        return err
    }
    if params.SendEmail {
        _ = emailService.Send(ctx, params.User.Email, "Welcome!")
    }
    return nil
}
```

**Don’t:**

```go
func saveUser(ctx context.Context, user User, sendEmail bool, notifyAdmin bool) error {
    // Too many parameters, harder to maintain as function evolves.
    return nil
}
```

---

##  Always Handle Errors Inside Functions

Never ignore errors. Handle them or propagate them up with context.

**Do:**

```go
func saveUser(ctx context.Context, user User) error {
    if err := userRepo.Insert(ctx, user); err != nil {
        return fmt.Errorf("saveUser: failed to insert user: %w", err)
    }
    return nil
}
```

**Don’t:**

```go
func saveUser(ctx context.Context, user User) error {
    userRepo.Insert(ctx, user) // Ignoring error can cause data loss or inconsistencies.
    return nil
}
```

---

##  Make Return Values Predictable

Return `nil` errors and zero values consistently on success.

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
func findUser(ctx context.Context, id string) (User, error) {
    user, err := userRepo.FindByID(ctx, id)
    if err != nil {
        return user, err // Returning partial user on error can cause confusion.
    }
    return user, nil
}
```