# Go Naming Conventions

Good naming is key to reducing cognitive complexity, making code self-documenting.

## Variable Naming

### CamelCase
```go
// Correct: camelCase
userName := "john"
totalPrice := 100.0
isValid := true

// Wrong: underscores or all caps
user_name := "john"    // Not recommended
TOTAL_PRICE := 100.0   // Only for constants
```

### Exported vs Unexported
```go
// Exported variables (capitalized) - accessible from other packages
type User struct {
    Name  string  // Exported field
    Email string
}

// Unexported variables (lowercase) - current package only
type user struct {
    name  string  // Unexported field
    email string
}
```

### Brevity Principle
```go
// Short names acceptable in local scope
for i := 0; i < len(items); i++ { }
for _, s := range strings { }

// Avoid redundant context
type User struct {
    Name string  // Correct
    // UserName string  // Redundant, already in User struct
}
```

### Semantic Naming
```go
// Correct: self-documenting
maxRetryCount := 3
connectionTimeout := 30 * time.Second
userEmailAddress := "user@example.com"

// Wrong: vague naming
data := getData()      // What data?
info := getInfo()      // What info?
value := getValue()    // What value?
temp := process()      // What temp?
```

### Boolean Variables
```go
// Use is/has/can/should prefix
isValid := true
hasPermission := false
canEdit := true
shouldRetry := false

// Error variables start with Err
var ErrNotFound = errors.New("not found")
var ErrInvalidInput = errors.New("invalid input")
```

### Slice/Map Naming
```go
// Reflect the element type stored
userList := []User{}
userMap := map[string]User{}
orderIDs := []string{}

// Avoid generic naming
arr := []User{}   // Not recommended
m := map[string]User{}  // Not recommended
```

### Avoid Uncommon Abbreviations
```go
// Correct
user := getUser()
password := getPassword()
configuration := loadConfig()

// Not recommended
usr := getUser()      // Use user
pwd := getPassword()  // Use password
cfg := loadConfig()   // cfg is acceptable, commonly used
```

---

## Function Naming

### CamelCase
```go
// Exported functions (capitalized)
func ReadFile(path string) ([]byte, error) { }
func ParseJSON(data []byte) (interface{}, error) { }

// Unexported functions (lowercase)
func readFile(path string) ([]byte, error) { }
func parseJSON(data []byte) (interface{}, error) { }
```

### Verb Prefix
```go
// Common verbs
func GetUser(id string) (*User, error) { }      // Get
func SetConfig(cfg *Config) error { }           // Set
func CreateOrder(req *OrderReq) (*Order, error) { }  // Create
func DeleteUser(id string) error { }            // Delete
func ParseConfig(data []byte) (*Config, error) { }   // Parse
func CheckPermission(user *User) bool { }       // Check
func ValidateInput(input string) error { }      // Validate
func CalculateTotal(items []Item) float64 { }   // Calculate
```

### Constructors
```go
// Use NewXXX naming
func NewClient(addr string) *Client { }
func NewConfig(opts ...Option) *Config { }
func NewUserService(repo UserRepo) *UserService { }
```

### Test Functions
```go
// Must start with Test
func TestCalculateTotal(t *testing.T) { }
func TestUserService_Create(t *testing.T) { }
func TestParseConfig_InvalidInput(t *testing.T) { }
```

### Avoid Redundancy
```go
// Correct
func (u *User) Name() string { }

// Redundant
func (u *User) GetUserName() string { }  // User already indicates it's a user
```

### Parameter and Return Value Naming
```go
// Named return values enhance readability
func divide(a, b int) (quotient, remainder int, err error) {
    if b == 0 {
        err = errors.New("division by zero")
        return
    }
    quotient = a / b
    remainder = a % b
    return
}
```

---

## Constant Naming

```go
// ALL_CAPS + underscores (public constants)
const MAX_RETRY_COUNT = 3
const DEFAULT_TIMEOUT = 30

// Or use camelCase (Go official style)
const MaxRetryCount = 3
const DefaultTimeout = 30

// Enum constants
const (
    StatusPending   = "pending"
    StatusActive    = "active"
    StatusCompleted = "completed"
)
```

---

## Package Naming

```go
// Short, lowercase, single word
package user
package order
package config

// Avoid
package userService    // Don't use camelCase
package user_service   // Don't use underscores
package util           // Too generic
package common         // Too generic
package misc           // Too generic
```

---

## Interface Naming

```go
// Single-method interface: method name + er
type Reader interface {
    Read(p []byte) (n int, err error)
}

type Writer interface {
    Write(p []byte) (n int, err error)
}

type Closer interface {
    Close() error
}

// Multi-method interface: descriptive name
type UserRepository interface {
    FindByID(id string) (*User, error)
    Create(user *User) error
    Update(user *User) error
    Delete(id string) error
}
```

---

## Naming Checklist

- [ ] Is the variable name self-documenting?
- [ ] Avoided vague naming (data, info, value)?
- [ ] Boolean variables use is/has/can prefix?
- [ ] Function names start with a verb?
- [ ] Avoided unnecessary abbreviations?
- [ ] Avoided redundant context?
- [ ] Exported/unexported used correctly?
