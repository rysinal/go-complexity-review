# Complexity Reduction Refactoring Patterns

## 1. Invert Expression

Convert complex positive conditions to simpler negative conditions, reducing logical branches.

**Before (Cyclomatic Complexity=4):**
```go
func process(condition1, condition2 bool) string {
    // Condition: (condition1 && condition2) || !condition1
    // Branches: if/else + && + || = 4
    if (condition1 && condition2) || !condition1 {
        return "A"
    } else {
        return "B"
    }
}
```

**After (Cyclomatic Complexity=3):**
```go
func process(condition1, condition2 bool) string {
    // Logical equivalence: only condition1 && !condition2 returns B
    if condition1 && !condition2 {
        return "B"
    }
    return "A"
}
```

**Further Optimized (Cyclomatic Complexity=2):**
```go
func process(condition1, condition2 bool) string {
    if !condition1 || condition2 {
        return "A"
    }
    return "B"
}
```

---

## 2. Decompose Conditional

Split complex conditions into multiple single-responsibility functions, each handling one check.

**Before (Cyclomatic Complexity=5):**
```go
func handleAlert(alert Alert) {
    if alert.IsUrgent {
        if alert.IsAdmin {
            notifyAdmin(alert)
        }
        escalate(alert)
    } else if alert.IsNormal {
        if alert.IsOnline {
            sendNotification(alert)
        }
    }
}
```

**After (Main function Cyclomatic Complexity=3):**
```go
func handleAlert(alert Alert) {
    if alert.IsUrgent {
        handleUrgentAlert(alert)
        return
    }
    if alert.IsNormal {
        handleNormalAlert(alert)
    }
}

func handleUrgentAlert(alert Alert) {
    if alert.IsAdmin {
        notifyAdmin(alert)
    }
    escalate(alert)
}

func handleNormalAlert(alert Alert) {
    if alert.IsOnline {
        sendNotification(alert)
    }
}
```

---

## 3. Consolidate Conditional

Merge scattered, logically related condition checks into more concise expressions.

**Before (Cyclomatic Complexity=4):**
```go
func isInvalid(user User) bool {
    if user.Age < 0 {
        return true
    }
    if user.Name == "" {
        return true
    }
    if user.Email == "" {
        return true
    }
    return false
}
```

**After (Cyclomatic Complexity=2):**
```go
func isInvalid(user User) bool {
    // Consecutive || belongs to same conditional expression, no extra decision points
    return user.Age < 0 || user.Name == "" || user.Email == ""
}
```

---

## 4. Remove Control Flag

Replace control flag variables (like `found`, `done`) with early returns.

**Before (Cyclomatic Complexity=3):**
```go
func hasAdmin(users []string) bool {
    found := false
    for i := 0; i < len(users) && !found; i++ {
        if users[i] == "admin" {
            found = true
        }
    }
    return found
}
```

**After (Cyclomatic Complexity=2):**
```go
func hasAdmin(users []string) bool {
    for _, user := range users {
        if user == "admin" {
            return true
        }
    }
    return false
}
```

> Note: Simple `for range` loops don't count as decision conditions; only termination conditions with logic count.

---

## 5. Guard Clause (Avoid Nesting)

Use guard clauses to flatten nested conditions, handling exceptional cases first with early returns.

**Before (Cyclomatic Complexity=5, Nesting=3 levels):**
```go
func canAccess(user User, resource Resource) bool {
    if user.IsAuthenticated {
        if resource.IsPublic {
            return true
        } else if user.Role == "admin" {
            return true
        } else if user.ID == resource.OwnerID {
            return true
        }
    }
    return false
}
```

**After - Guard Clause (Cyclomatic Complexity=5, Nesting=0 levels):**
```go
func canAccess(user User, resource Resource) bool {
    if !user.IsAuthenticated {
        return false
    }
    if resource.IsPublic {
        return true
    }
    if user.Role == "admin" {
        return true
    }
    if user.ID == resource.OwnerID {
        return true
    }
    return false
}
```

**Further Optimized - Consolidate Conditional (Cyclomatic Complexity=3):**
```go
func canAccess(user User, resource Resource) bool {
    if !user.IsAuthenticated {
        return false
    }
    return resource.IsPublic ||
           user.Role == "admin" ||
           user.ID == resource.OwnerID
}
```

**Guard Clause Core Principles:**
- Early return, handle exceptions: Check unsatisfied conditions at function start, return immediately
- Flat checks, replace nesting: Merge or split inner conditions to same-level `if` statements
- Continue in loops, skip invalid items: Use `continue` to skip items that don't meet conditions

---

## 6. Extract Function

Split large functions into multiple single-responsibility small functions, achieving high cohesion and low coupling.

**Before (Cyclomatic Complexity=7):**
```go
func processOrder(action string, order *Order) error {
    if action == "create" {
        if order.ID == "" {
            order.ID = generateID()
        }
        return saveOrder(order)
    } else if action == "pay" {
        if order.Status != "created" {
            return errors.New("invalid status")
        }
        return processPayment(order)
    } else if action == "cancel" {
        if order.Status == "paid" {
            return errors.New("cannot cancel paid order")
        }
        return cancelOrder(order)
    }
    return errors.New("unknown action")
}
```

**After (Main function Cyclomatic Complexity=2):**
```go
func processOrder(action string, order *Order) error {
    handler, ok := orderHandlers[action]
    if !ok {
        return errors.New("unknown action")
    }
    return handler(order)
}

func handleCreate(order *Order) error {
    if order.ID == "" {
        order.ID = generateID()
    }
    return saveOrder(order)
}

func handlePay(order *Order) error {
    if order.Status != "created" {
        return errors.New("invalid status")
    }
    return processPayment(order)
}

func handleCancel(order *Order) error {
    if order.Status == "paid" {
        return errors.New("cannot cancel paid order")
    }
    return cancelOrder(order)
}

var orderHandlers = map[string]func(*Order) error{
    "create": handleCreate,
    "pay":    handlePay,
    "cancel": handleCancel,
}
```

---

## 7. Table-Driven Programming

Replace multi-branch logic with table lookups, moving discrete condition checks into table structures.

**Before (Cyclomatic Complexity=4):**
```go
func calculate(op string, a, b int) (int, error) {
    switch op {
    case "+":
        return a + b, nil
    case "-":
        return a - b, nil
    case "*":
        return a * b, nil
    default:
        return 0, errors.New("unknown operator")
    }
}
```

**After (Cyclomatic Complexity=2):**
```go
var operations = map[string]func(int, int) int{
    "+": func(a, b int) int { return a + b },
    "-": func(a, b int) int { return a - b },
    "*": func(a, b int) int { return a * b },
}

func calculate(op string, a, b int) (int, error) {
    fn, ok := operations[op]
    if !ok {
        return 0, errors.New("unknown operator")
    }
    return fn(a, b), nil
}
```

**Table-Driven Use Cases:**
- Multiple `case` branches performing similar operations
- Branch logic can be abstracted into functions
- Need to dynamically extend branches

---

## 8. Use defer

Centralize resource cleanup, avoiding duplicate code across multiple return paths.

**Before:**
```go
func readFile(path string) ([]byte, error) {
    f, err := os.Open(path)
    if err != nil {
        return nil, err
    }

    data, err := io.ReadAll(f)
    if err != nil {
        f.Close()  // Easy to forget
        return nil, err
    }

    f.Close()  // Duplicate code
    return data, nil
}
```

**After:**
```go
func readFile(path string) ([]byte, error) {
    f, err := os.Open(path)
    if err != nil {
        return nil, err
    }
    defer f.Close()  // Unified management, executes regardless of return path

    return io.ReadAll(f)
}
```

**defer Use Cases:**
- File/connection close: `defer f.Close()`
- Lock release: `defer mu.Unlock()`
- Error recovery: `defer func() { recover() }()`
- State rollback: `defer func() { os.Chdir(originalDir) }()`

---

## Refactoring Effect Comparison

| Technique | Typical Scenario | Complexity Reduction |
|-----------|------------------|----------------------|
| Invert Expression | `(a && b) \|\| !a` | 4 → 2 |
| Decompose Conditional | Nested if-else | 5 → 3 |
| Consolidate Conditional | Multiple if return | 4 → 2 |
| Remove Control Flag | found/done variables | 3 → 2 |
| Guard Clause | Deep nesting | 5 → 3 |
| Extract Function | Large multi-responsibility function | 7 → 2 |
| Table-Driven | switch-case chains | 4 → 2 |
