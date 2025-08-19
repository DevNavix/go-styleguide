# **Function Naming Guidelines**


## Introduction

Consistent and idiomatic function naming improves your code’s readability, maintainability, and ease of collaboration. This guide summarizes all naming, export, and usage rules for Go functions, with examples and a checklist for code review.

#### General Naming Principles

- **Use MixedCaps or mixedCaps for multiword names:** Prefer `ReadFile` or `readFile` over `read_file`. Avoid underscores.
- **Names are significant for visibility:** An identifier starting with an uppercase letter is exported (public), while lowercase is unexported (private).
- **Acronyms use all capitals:** Examples include `ServeHTTP`, `IDProcessor`.
- **Be concise but clear:** Longer names do not necessarily enhance readability; brevity and clarity are key. If more explanation is needed, favor a good doc comment over a long name.
- **Variable names are short when the context is clear:** E.g., use `i` for loop indexes, `r` for readers. For wider scope, use more descriptive names. 

## Core Naming Conventions

### Exported vs. Unexported Functions

- **Exported functions** (public to other packages) **start with an uppercase letter**.
- **Unexported functions** (private within the package) **start with a lowercase letter**.

**Example:**

```go

// In package user

// Exported function
func RegisterUser(name, email string) (*User, error) { /*..logic behind the api..*/ }

// Unexported helper
func validateEmail(email string) bool { /*..working of api..*/ }

```


### When to Use a Method vs. a Function

- **Method:** Logic is tightly linked to a type; needs access to its fields or must implement an interface.
- **Function:** Standalone utility or logic unaffected by or not belonging to a specific type.

**Example:**

```go

type User struct {
    Name  string
    Email string
    Age   int
}

// Method (belongs to User)
func (u *User) IsAdult() bool { return u.Age >= 18 }

// Standalone function (utility)
func CalculateBMI(weightKg, heightM float64) float64 {
    return weightKg / (heightM * heightM)
}

```


### Naming Patterns and Idioms

- Multiword names: **MixedCaps** (camel case), no underscores.
- **Verbs** for functions that perform actions (e.g., `SaveFile`, `UpdateRecord`) ex func use to run db queries.
- If function panics on error, **prefix with `Must`** (e.g., `MustParseConfig`).
- **Type or format suffix** for overloaded or specialized functions (`ParseInt`, `ParseInt64`).
- Short, idiomatic variable names (`i`, `r`, `err`) for small scope; meaningful names for broader scope.
- **Test functions:** Always use `TestXxx`, `BenchmarkXxx`, `ExampleXxx` in test files.

## Comprehensive Examples

### Exported/Unexported, Method/Function, Helper Patterns

```go
package account

type Account struct {
    ID    int
    email string
}

// Exported function
func NewAccount(id int, email string) *Account {
    return &Account{ID: id, email: email}
}

// Exported method
func (a *Account) Email() string {
    return a.email
}

// Unexported helper
func normalizeEmail(email string) string {
    return strings.ToLower(strings.TrimSpace(email))
}

// Unexported method (internal to package)
func (a *Account) isEmailValid() bool {
    return strings.Contains(a.email, "@")
}

// Must-prefix pattern (panics on error)
func MustOpenConfig(path string) *Config {
    cfg, err := openConfig(path)
    if err != nil {
        panic(err)
    }
    return cfg
}
```
---

## Duplicate or Ambiguously Similar Exported Functions

Exported functions in Go are differentiated by their names. Go is case-sensitive, so ValidateUser and validateUser are two distinct identifiers. However, for exported functions, only names beginning with uppercase letters (ValidateUser) are accessible outside the package. Having names that differ only in casing or are extremely similar can lead to confusion and errors:

- **Searchability problem:** Developers may accidentally search for ValidateUser but miss validateUser (or vice versa), leading to overlooked code when debugging or refactoring.
- **Maintenance risk:** Accidentally introducing both ValidateUser and validateUser in the same package can cause ambiguity in intent, especially if both are meant to validate a user but with slightly different logic or visibility.
- **Collisions and clarity:** If you later change a private function to exported (capitalize the first letter), you might collide with an already existing exported name, causing compilation errors.

**❌ Don’t:**
```go

func validateOrder(o Order) bool { ... }
func ValidateOrder(o Order) bool { ... } // now will conflict if internal gets exported later

```
---

## Avoid Generic Method Names

- **Clarity and Purpose:** The function name should immediately convey what the method does. Generic names like Process are vague and force developers to dig into the code to understand its behavior. A new team member or someone maintaining the code months later will struggle with ambiguous names.
- **Search and Maintenance:** If every package has a method called Process, searching for usages or debugging becomes cumbersome. You can't tell from the name alone whether it's handling reports, payloads, events, etc.
- **Domain Language and Consistency:** Good Go naming uses clear, domain-relevant terms. This helps business stakeholders and technical teams understand what code does at a glance, and fosters consistency across the codebase.
- **Risk of Collisions and Misuse:** Generic names increase the risk of naming collisions and accidental misuse, especially in larger codebases or when refactoring.

```go

// Good: Explicit, clear method names
func (r *ReportService) GenerateReport(data ReportData) (*Report, error) { ... }
func (p *PayloadHandler) ValidatePayload(payload Payload) error { ... }

// Bad: Generic method names
func (r *ReportService) Process(data interface{}) error { ... }
func (p *PayloadHandler) Process(input interface{}) error { ... }

```

---


## Naming Scenario Table

| Usage | Exported/Public | Unexported/Internal | Example |
| :-- | :-- | :-- | :-- |
| Standalone function | Yes | Yes | RegisterUser, validateEmail |
| Method tied to a type (struct) | Yes | Yes | Email, isEmailValid |
| Getters (no `Get` prefix) | Yes | Yes | Email(), name() |
| Constructors (public) | Yes | (rarely, if at all) | NewAccount |
| Must-prefix for panic-on-failure | Yes | Yes | MustOpenConfig |
| Type/format suffix for specialization | Yes | Yes | ParseInt, ParseInt64 |
| Test/benchmark/example functions | Yes | - | TestRegisterUser, BenchmarkEmail |

