# **Function Naming Guidelines**


## Introduction

Consistent and idiomatic function naming improves your code’s readability, maintainability, and ease of collaboration. This guide summarizes all naming, export, and usage rules for Go functions, with examples and a checklist for code review.

#### General Naming Principles

- **Use MixedCaps or mixedCaps for multiword names:** Prefer `ReadFile` or `readFile` over `read_file`. Avoid underscores.
- **Names are significant for visibility:** An identifier starting with an uppercase letter is exported (public), while lowercase is unexported (private).
- **Acronyms use all capitals:** Examples include `ServeHTTP`, `IDProcessor`. **todo: Take a look on it**
- **Be concise but clear:** Longer names do not necessarily enhance readability; brevity and clarity are key. If more explanation is needed, favor a good doc comment over a long name.
- **Variable names are short when the context is clear:** E.g., use `i` for loop indexes, `r` for readers. For wider scope, use more descriptive names. **todo: Take a look on it**


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
- **Nouns** or noun-like getters for retrieving data (e.g., `JobName()` not `GetJobName()`).
- **Avoid** `Get` prefix for getters.
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

## Final Function Naming \& Usage Checklist

Use this before **code review** or **Pull Request** submission:

1. **Exported vs. Unexported**
    - Does this function/method need to be used outside the package? → Capitalized name.
    - If this is only for internal use, is it lowercase?
2. **Function vs. Method Appropriateness**
    - Is this logic truly tied to a type’s data? → Method.
    - Is this a general-purpose utility? → Standalone function.
3. **Naming Patterns**
    - Is the function name a clear verb (for actions) or noun (for getters)?
    - No `Get` prefix on getters?
    - If panics on error, is it prefixed with `Must`?
    - Are type or format suffixes used only when there is function overloading?
    - No underscores in function names?
4. **Idiomatic Short Variable Names** 
    - Are variable names concise yet meaningful? Single letters only for short scope.
    - Is `err` used for error values, as is Go idiom? **todo Please revise this**
5. **Special Cases**
    - Are test/benchmark/example functions named correctly for test files?
    - For exported functions and types, is there a proper doc comment?
6. **Consistency \& Clarity**
    - Do names reflect their roles clearly in context?
    - Is there any repetitive or redundant naming that can be simplified?

#### Bonus Tips you should know about

- **Single-method interfaces end with -er:** Examples: `Reader`, `Writer`,
- **Use established names for common operations:** If your type implements a well-known interface (like converting to string), use the canonical method

example

```go

// Converting to String — String() string
// If your type can be converted to a string representation, implement the String() string method instead of something like ToString() or GetString().
// 
// This method is part of the fmt.Stringer interface, which is recognized by many standard library packages (e.g., fmt, log) to produce string outputs.

type User struct {
    Name string
    Age  int
}

// Canonical method to convert User to string
func (u User) String() string {
    return fmt.Sprintf("User{Name: %s, Age: %d}", u.Name, u.Age)
}

// Using fmt.Println(u) will call User.String() automatically


```

## Conclusion

Adhering to these function naming standards ensures your Go code is idiomatic, maintainable, and easy for others to understand and use. Regularly review and refactor for clarity and apply the checklist before each submission to maintain a high-quality codebase.

---
Backend is the backbone of **IRON** — clear names ensures it stays robust and easy to maintain.