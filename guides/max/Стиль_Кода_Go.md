# Стиль Кода Go (дополнение)

> Строгий code style для Go-проектов.
---
**База**: см. [Стиль_Кода_Go.md](http://../common/Стиль_Кода_Go.md)
---

## Дополнительные разделы

## Содержание

1. [Расширенные паттерны](#расширенные-паттерны)
1. [Обработка ошибок](#обработка-ошибок)
1. [Тестирование](#тестирование)

---

## Расширенные паттерны

### Builder Pattern

```go
type StringBuilder struct {
    buffer   strings.Builder
    capacity int
}

func NewStringBuilder(capacity int) *StringBuilder {
    var sb strings.Builder
    sb.Grow(capacity)
    return &StringBuilder{
        buffer:   sb,
        capacity: capacity,
    }
}

func (b *StringBuilder) Write(str string) {
    b.buffer.WriteString(str)
}

func (b *StringBuilder) String() string {
    return b.buffer.String()
}
```

### Interface Pattern

```go
// Reader определяет контракт чтения
type Reader interface {
    Read(p []byte) (n int, err error)
}

// FileReader реализует Reader для файлов
type FileReader struct {
    file *os.File
}

func (r *FileReader) Read(p []byte) (n int, err error) {
    return r.file.Read(p)
}
```

### Functional Options Pattern

```go
type Server struct {
    port    int
    timeout time.Duration
    logger  *slog.Logger
}

type Option func(*Server)

func WithPort(port int) Option {
    return func(s *Server) {
        s.port = port
    }
}

func WithTimeout(timeout time.Duration) Option {
    return func(s *Server) {
        s.timeout = timeout
    }
}

func NewServer(opts ...Option) *Server {
    s := &Server{
        port:    8080,
        timeout: 30 * time.Second,
        logger:  slog.Default(),
    }
    for _, opt := range opts {
        opt(s)
    }
    return s
}
```

---

## Обработка ошибок

### Custom Error Types

```go
type ErrorCode int

const (
    ErrNotFound    ErrorCode = iota + 1
    ErrInvalidInput
    ErrInternal
    ErrUnauthorized
)

type AppError struct {
    Code    ErrorCode
    Message string
    Err     error
}

func (e *AppError) Error() string {
    return e.Message
}

func (e *AppError) Unwrap() error {
    return e.Err
}

// Error messages
func (e ErrorCode) String() string {
    switch e {
    case ErrNotFound:
        return "not found"
    case ErrInvalidInput:
        return "invalid input"
    case ErrInternal:
        return "internal error"
    case ErrUnauthorized:
        return "unauthorized"
    default:
        return "unknown error"
    }
}
```

### Error Recovery Pattern

```go
func ProcessWithFallback(input string) (string, error) {
    result, err := primaryParse(input)
    if err != nil {
        // Fallback to alternative
        return fallbackParse(input)
    }
    return result, nil
}
```

---

## Тестирование

### Table-Driven Tests

```go
func TestValidateEmail(t *testing.T) {
    tests := []struct {
        name          string
        input         string
        wantValid     bool
        wantErrorCode ErrorCode
    }{
        {name: "valid email", input: "test@example.com", wantValid: true},
        {name: "invalid", input: "invalid", wantValid: false, wantErrorCode: ErrInvalidInput},
        {name: "empty", input: "", wantValid: false, wantErrorCode: ErrInvalidInput},
        {name: "no domain", input: "@example.com", wantValid: false, wantErrorCode: ErrInvalidInput},
    }
    for _, tt := range tests {
        t.Run(tt.name, func(t *testing.T) {
            result, err := ValidateEmail(tt.input)
            if tt.wantValid && err != nil {
                t.Errorf("ValidateEmail(%q) = %v, want nil", tt.input, err)
            }
            if !tt.wantValid && err == nil {
                t.Errorf("ValidateEmail(%q) = nil, want error", tt.input)
            }
        })
    }
}
```

### Mock with Interfaces

```go
// Repository interface
type UserRepository interface {
    FindByID(id string) (*User, error)
    Save(user *User) error
}

// Mock implementation
type MockUserRepository struct {
    users    map[string]*User
    failNext bool
}

func NewMockUserRepository() *MockUserRepository {
    return &MockUserRepository{
        users: make(map[string]*User),
    }
}

func (m *MockUserRepository) FindByID(id string) (*User, error) {
    if m.failNext {
        m.failNext = false
        return nil, errors.New("mock failure")
    }
    user, ok := m.users[id]
    if !ok {
        return nil, ErrNotFound
    }
    return user, nil
}

func (m *MockUserRepository) Save(user *User) error {
    if m.failNext {
        m.failNext = false
        return errors.New("mock failure")
    }
    m.users[user.ID] = user
    return nil
}

func TestUserService(t *testing.T) {
    mockRepo := NewMockUserRepository()
    service := NewUserService(mockRepo)

    user, err := service.Create("Alice")
    if err != nil {
        t.Fatalf("Create() = %v", err)
    }

    found, err := service.FindByID(user.ID)
    if err != nil {
        t.Fatalf("FindByID() = %v", err)
    }
    if found.Name != "Alice" {
        t.Errorf("found.Name = %q, want %q", found.Name, "Alice")
    }
}
```

### Fuzz Testing

```go
func FuzzValidateEmail(f *testing.F) {
    f.Add("user@example.com")
    f.Add("invalid")
    f.Add("")
    f.Add("@")

    f.Fuzz(func(t *testing.T, input string) {
        // Should not panic
        _, err := ValidateEmail(input)
        if err != nil {
            // Error is expected for invalid inputs
            // Just verify it doesn't panic
        }
    })
}

// Run with: go test -fuzz=FuzzValidateEmail
```
