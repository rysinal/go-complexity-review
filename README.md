# go-complexity-review

A Claude Code skill for Go code complexity analysis and refactoring guidance.

## Installation

```bash
# Install via skills CLI
npx skills add rysinal/go-complexity-review

# Or clone to local skills directory
git clone https://github.com/rysinal/go-complexity-review.git ~/.claude/skills/go-complexity-review
```

## Features

- **Cyclomatic Complexity Analysis** - Measure logical branch count (threshold ≤ 10)
- **Cognitive Complexity Analysis** - Evaluate code understandability (minimize)
- **Nesting Depth Control** - Keep nesting ≤ 3 levels with guard clauses
- **Function Length Control** - Split functions exceeding 50 lines
- **8 Refactoring Patterns** - Practical techniques with before/after examples
- **Go Naming Conventions** - Best practices for readable code

## Thresholds

| Metric | Threshold | Description |
|--------|-----------|-------------|
| Cyclomatic Complexity | ≤ 10 | if/for/switch branches + 1 |
| Cognitive Complexity | Minimize | Reduce nesting and control flow breaks |
| Nesting Depth | ≤ 3 levels | Use early return, guard clause |
| Function Length | ≤ 50 lines | Split into sub-functions if exceeded |

## Trigger Phrases

The skill activates when you mention:
- "complexity", "cyclomatic complexity", "cognitive complexity", "CC"
- "check the complexity of this function"
- "this function is too complex, help me refactor"
- "reduce nesting levels" or "simplify conditional logic"
- "make code more readable/maintainable"

## Refactoring Patterns

1. **Invert Expression** - Simplify complex boolean conditions
2. **Decompose Conditional** - Split into single-responsibility functions
3. **Consolidate Conditional** - Merge scattered related checks
4. **Remove Control Flag** - Replace flags with early returns
5. **Guard Clause** - Flatten nested conditions
6. **Extract Function** - Split large functions
7. **Table-Driven** - Replace switch/if-else chains with lookups
8. **defer** - Centralize resource cleanup

## CLI Tools

```bash
# Install
go install github.com/fzipp/gocyclo/cmd/gocyclo@latest
go install github.com/uudashr/gocognit/cmd/gocognit@latest

# Usage
gocyclo -over 10 -avg .    # Show functions with CC > 10
gocognit -over 15 .        # Show functions with cognitive complexity > 15
```

## License

MIT
