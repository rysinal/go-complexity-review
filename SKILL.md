---
name: go-complexity-review
description: |
  Go code complexity review expert, focusing on Cyclomatic Complexity and Cognitive Complexity analysis and refactoring.
  Trigger scenarios:
  (1) Review Go code complexity issues, e.g., "check the complexity of this function"
  (2) Analyze cyclomatic or cognitive complexity, e.g., "what's the cyclomatic complexity of this function"
  (3) Refactor high-complexity code, e.g., "this function is too complex, help me refactor"
  (4) Evaluate code maintainability during code review
  (5) User mentions "complexity", "cyclomatic complexity", "cognitive complexity", "CC"
  (6) User requests to reduce nesting levels or simplify conditional logic
  (7) User asks how to make code more readable or maintainable
---

# Go Code Complexity Review

## Core Metric Thresholds

| Metric | Threshold | Description |
|--------|-----------|-------------|
| Cyclomatic Complexity | ≤ 10 | if/for/switch branches + 1 |
| Cognitive Complexity | Minimize | Reduce nesting and control flow breaks |
| Nesting Depth | ≤ 3 levels | Use early return, guard clause |
| Function Length | ≤ 50 lines | Split into sub-functions if exceeded |

## Review Process

1. **Identify high-complexity functions** - Use tools or manual inspection
2. **Analyze complexity sources** - Count decision points, check nesting levels
3. **Choose refactoring strategy** - Select appropriate pattern based on complexity type
4. **Verify refactoring results** - Recalculate complexity, ensure tests pass

## Cyclomatic Complexity

Measures the number of logical branches in code, representing the minimum test cases needed to cover all paths.

**Formula:** `V(G) = P + 1` (P = number of decision points, i.e., if/for/switch branches)

**Decision Point Types:**

| Type | Example | Counting Rule |
|------|---------|---------------|
| Conditional | `if / else if / else` | Each counts as 1 |
| Loop | `for` | Termination condition with logic counts |
| Multi-branch | `switch...case` | Each `case` +1, `default` doesn't add |
| Logical operators | `&& / \|\|` | Each operator adds +1 |

**Assessment Criteria:**

| Cyclomatic Complexity | Code Status | Testability | Maintenance Cost |
|-----------------------|-------------|-------------|------------------|
| 1~10 | Clear, structured | High | Low |
| 10~20 | Complex | Medium | Medium |
| 20~30 | Very complex | Low | High |
| >30 | Unreadable | Untestable | Very High |

**Mandatory Threshold: ≤ 10**

## Cognitive Complexity

Measures code understandability, quantifying the cognitive burden required to read code.

**Three Core Principles:**

### Principle 1: Ignore Shorthand
Syntactic sugar doesn't add complexity (e.g., list comprehensions).

### Principle 2: Flow Breaks
Each linear flow interruption +1:
- Loops: `for`, `while`
- Conditionals: `if`, ternary operator
- `catch` statements: each +1
- `switch`: +1 total (not per case)
- Mixed logical operators: each new sequence +1 (e.g., `a && b || c` has `&&` and `||` each +1)
- Recursive calls: +1
- Jump labels: `goto`, `break/continue` to label +1

### Principle 3: Nesting Penalty
Deeper nesting levels add +1 per level.

**Examples:**
```go
// No nesting: 1 point
if condition { }  // +1

// 1 level nesting: 3 points
for i := range items {  // +1
    if condition { }    // +1 (base) +1 (nesting) = +2
}

// 2 level nesting: 6 points
if a {              // +1
    for i := range items {  // +1 +1(nesting) = +2
        if b { }    // +1 +2(nesting) = +3
    }
}
```

**Goal: Minimize** - Reduce through less nesting and fewer control flow breaks

## Complexity Reduction Techniques

See [references/refactoring-patterns.md](references/refactoring-patterns.md) for detailed patterns and code examples.

### Nesting Depth Control (≤ 3 levels)

```go
// Wrong: Too deep nesting (4 levels)
func process(user User, order Order) error {
    if user.IsActive {
        if order.IsValid {
            for _, item := range order.Items {
                if item.InStock {  // Level 4!
                    // ...
                }
            }
        }
    }
    return nil
}

// Correct: Use early return and guard clause
func process(user User, order Order) error {
    if !user.IsActive {
        return ErrInactiveUser
    }
    if !order.IsValid {
        return ErrInvalidOrder
    }
    for _, item := range order.Items {
        if !item.InStock {
            continue
        }
        // ...
    }
    return nil
}
```

### Function Length Control (≤ 50 lines)

Functions exceeding 50 lines should be split into sub-functions:

```go
// Wrong: Function too long
func processOrder(order *Order) error {
    // Validation logic (15 lines)
    // Calculation logic (20 lines)
    // Save logic (20 lines)
    // Notification logic (15 lines)
    // Total: 70 lines
}

// Correct: Split into sub-functions
func processOrder(order *Order) error {
    if err := validateOrder(order); err != nil {
        return err
    }
    if err := calculateOrder(order); err != nil {
        return err
    }
    if err := saveOrder(order); err != nil {
        return err
    }
    return notifyOrder(order)
}
```

### Refactoring Quick Reference

| Technique | Use Case | Effect |
|-----------|----------|--------|
| Early Return/Guard Clause | Deep nesting | Nesting ≤ 3 levels |
| Extract Function | Long functions | Length ≤ 50 lines |
| Invert Expression | Complex boolean `(a && b) \|\| !a` | Simplify conditions |
| Decompose Conditional | Multiple `&&/\|\|` nesting | Separate responsibilities |
| Consolidate Conditional | Scattered related checks | Reduce decision points |
| Remove Control Flag | `found/done` flag variables | Early return |
| Table-Driven | switch/if-else chains | Lookup replaces branching |
| defer | Multiple return path cleanup | Centralized resource management |

## CLI Tools

```bash
# Install
go install github.com/fzipp/gocyclo/cmd/gocyclo@latest
go install github.com/uudashr/gocognit/cmd/gocognit@latest

# Usage (with thresholds)
gocyclo -over 10 -avg .    # Show functions with CC > 10
gocognit -over 15 .        # Show functions with cognitive complexity > 15
```

## Naming Conventions (Reduce Cognitive Complexity)

See [references/naming-conventions.md](references/naming-conventions.md) for detailed conventions.

**Core Principles:**
- Variables: camelCase `userName`, booleans use `is/has/can` prefix
- Functions: verb prefix `Get/Set/Create/Delete/Parse/Check`
- Avoid: vague names `data/info/value`, uncommon abbreviations `usr/pwd`
