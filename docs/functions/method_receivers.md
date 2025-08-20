# **Method Receivers Guidelines**

This section defines the standards for using method receivers in Go.
Choosing between **pointer** and **value** receivers correctly ensures consistent behavior, performance, and maintainability.

---
<br>

## Use Pointer Receivers When Modifying the Receiver

If a method changes the receiver’s internal state, use a pointer receiver so changes are reflected in the caller.

**Do:**

```go
func (u *User) Activate() {
    u.Status = "active"
}
```

**Don’t:**

```go
func (u User) Activate() {
    u.Status = "active" // Modifies only a copy, original value unchanged.
}
```

---
<br>

## Use Pointer Receivers for Large Structs

Passing large structs by value creates unnecessary copies and hurts performance.

**Do:**

```go
type User struct {
    ID     string
    Name   string
    Email  string
    Status string
    Roles  []string
}

func (u *User) SetEmail(email string) {
    u.Email = email
}
```

**Don’t:**

```go
func (u User) SetEmail(email string) {
    u.Email = email // Copies entire struct every call.
}
```

---
<br>

## Use Value Receivers for Small, Immutable Structs

If the struct is small (few fields) and immutable, a value receiver is fine.

**Do:**

```go
type Point struct {
    X, Y int
}

func (p Point) DistanceToOrigin() float64 {
    return math.Sqrt(float64(p.X*p.X + p.Y*p.Y))
}
```

**Don’t:**

```go
func (p *Point) DistanceToOrigin() float64 {
    // Pointer receiver unnecessary here — no state modification, very small struct.
    return math.Sqrt(float64(p.X*p.X + p.Y*p.Y))
}
```

---
<br>

## Be Consistent Within the Type

All methods for a type should have the same receiver type (pointer or value), except for clear exceptions.

**Do:**

```go
type User struct {
    ID, Name string
}

func (u *User) Save(ctx context.Context) error { return nil }
func (u *User) Delete(ctx context.Context) error { return nil }
```

**Don’t:**

```go
type User struct {
    ID, Name string
}

func (u User) Save(ctx context.Context) error { return nil }  // Value receiver
func (u *User) Delete(ctx context.Context) error { return nil } // Pointer receiver
// Inconsistent receiver choice confuses maintainers.
```

---
<br>

## Prefer Functions Over Methods for Stateless Operations

If an operation does not depend on the type’s state, make it a function instead of a method.

**Do:**

```go
func validateEmail(email string) bool {
    return strings.Contains(email, "@")
}
```

**Don’t:**

```go
func (u User) validateEmail() bool {
    return strings.Contains(u.Email, "@") // Doesn't need to be tied to User struct.
}
```

---
<br>

## Avoid Pointer Receivers for Maps, Slices, and Channels

These are already reference types — using a pointer receiver gives no benefit unless modifying fields outside of these types.

**Do:**

```go
type UserList []User

func (ul UserList) Count() int {
    return len(ul)
}
```

**Don’t:**

```go
func (ul *UserList) Count() int {
    return len(*ul) // Pointer receiver unnecessary for slice.
}
```

---
<br>

## Avoid Exporting Methods That Don’t Belong to the Type

A method should only exist on a type if it logically operates on that type’s data.

**Do:**

```go
func (u *User) ChangeName(name string) {
    u.Name = name
}
```

**Don’t:**

```go
func (u *User) ParseJSON(data []byte) error {
    // JSON parsing is not inherently a responsibility of the User type.
    return json.Unmarshal(data, u)
}
```