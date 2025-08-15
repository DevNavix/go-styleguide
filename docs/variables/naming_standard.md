# Variable Naming Guidelines

Good variable naming improves **readability**, **maintainability**, and **code comprehension**.  
Variable names should be **short but meaningful**. Short variable names are acceptable when the context makes the purpose obvious, but descriptive names must be used when the context is not immediately clear.

---

## General Rules
1. **Use Short Names for Temporary or Well-Known Loop Variables**
   - Acceptable for small scopes like `for` loops, indexes, and counters (`i`, `j`, `idx`, etc.).
   - The variable’s role should be obvious from the surrounding code.

2. **Use Descriptive Names for Context-Sensitive Values**
   - When the variable stores a meaningful domain-specific value (e.g., user ID, customer name), clearly indicate its meaning in the name.
   - Avoid abbreviations unless they are well-understood in your domain.

3. **Avoid Single-Letter Names Except for Idiomatic Cases**
   - Examples where single-letter names are okay: Loop counters (`i`, `j`), rune iteration (`r`), error (`err`).

4. **Match Name to Content and Purpose**
   - Variable names should explain *what* the variable holds.
   - Example: if a map key is `userID` and its value is `userName`, use those as your variable names.

5. **Boolean Variables Should Sound Like Conditions**
   - Examples: `isActive`, `hasPermission`, `shouldRetry`.

---

## Examples

### Short Name in Loops (OK)
```go
// OK: Using short variable name "i" for loop counter is acceptable.
for i := 0; i < len(users); i++ {
    fmt.Println(users[i])
}
```
✅ **Why okay?** Context is obvious, `i` is a standard loop counter.

***

### Descriptive Names from a Map
```go
// Map of userID to userName
userNames := map[int]string{
    101: "Alice",
    102: "Bob",
}

// ❌ Wrong: Unclear what k and v represent
for k, v := range userNames {
    fmt.Printf("%d: %s\n", k, v)
}

// ✅ Correct: Use meaningful variable names
for userID, userName := range userNames {
    fmt.Printf("%d: %s\n", userID, userName)
}
```
✅ **Why better?** Without comments, it’s still clear what `userID` and `userName` represent.

***

### Short Name When Context Is Local
```go
for idx, order := range orders {
    fmt.Printf("Processing order %d: %+v\n", idx, order)
}
```
✅ **Why okay?** `idx` is acceptable here because it’s clearly an index, and `order` describes the data being iterated.

***

###  Avoid Generic Temp Naming
```go
// ❌ Wrong: Unclear what data holds
data := getFromDB()

// ✅ Correct: Name reflects database result
customerRecord := getFromDB()
```
✅ **Why better?** A future developer knows `customerRecord` is about a customer’s data, not some arbitrary "data".

***

## **DOs and DON’Ts Table**
| ✅ Do | ❌ Don’t |
|------|----------|
| Use `i`, `j`, `idx` for loop counters or where context is clear  | Use `i`, `j` for unrelated values |
| Use `userID`, `userName` for map keys & values | Use `k`, `v` when meaning is unclear |
| Use domain-specific names (`orderID`, `customerList`) | Use generic names (`data`, `info`) |
| Use `err` for errors consistently | Invent new names for errors like `e1`, `myErr` unless distinction is necessary |

***

### **Summary**
- **Short names:** Fine for very small scope & universally understood cases.
- **Descriptive names:** Required for anything related to business logic or where meaning isn’t obvious.
- Think **self-documenting** — someone reading the variable name should know what it holds without digging into code.

---