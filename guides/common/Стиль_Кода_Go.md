# Стиль Кода Go

## Форматирование

**Всегда запускай `gofmt` или `gofumpt` перед коммитом
**

```bash
gofmt -l -w .
# или более строгий
gofumpt -l -w .
```

- **Один таб** для отступов (стандарт Go)
- **Без висячих запятых** — в Go их нет
- **120 символов** максимум в строке (рекомендация)
- Импорты сортируются: std → external → internal

## Именование

| Что | Стиль | Пример |
|-----|-------|--------|
| Функции, переменные | camelCase | getUser, userName |
| Экспортируемые | PascalCase | GetUser, UserService |
| Константы | camelCase | maxRetryCount |
| Булевы | camelCase | isActive, hasPermission |
| Пакеты | lowercase, одно слово | user, http, config |
| Файлы | snake_case | `user_service.go` |

### Правила именования

```go
// ✅ ПРАВИЛЬНО
var isActive bool
const maxRetryCount = 3

// ❌ НЕПРАВИЛЬНО — не Go-стиль
var IsActive bool
const MAX_RETRY_COUNT = 3
```

### Акронимы

```go
// ✅ ПРАВИЛЬНО — акронимы в одном регистре
func GetHTTPClient() *http.Client
func ParseURL(url string) *URL
var userID string

// ❌ НЕПРАВИЛЬНО
func GetHttpClient()
func ParseUrl()
```

## Типы и Объявления

### Zero-initialization

```go
// ✅ ПРАВИЛЬНО — zero value
var user User
var count int

// ✅ ПРАВИЛЬНО — короткая форма
user := User{Name: "Alice"}
count := 0
```

### Явные типы для экспортируемых

```go
// ✅ ПРАВИЛЬНО
var Version = "1.0.0"           // ok, string очевиден
var MaxConnections = 100        // ok, int очевиден

// Лучше явно
var Version string = "1.0.0"
```

### Используй точные типы

```go
// ✅ ПРАВИЛЬНО
type UserID int64
type Email string

// ✅ ПРАВИЛЬНО — для API
type Config struct {
    Port    int    `json:"port"`
    Timeout time.Duration `json:"timeout"`
}
```

## Обработка Ошибок

### Всегда проверяй ошибки

```go
// ✅ ПРАВИЛЬНО
result, err := doSomething()
if err != nil {
    return fmt.Errorf("do something: %w", err)
}

// ❌ НЕПРАВИЛЬНО
result, _ := doSomething()
```

### Оборачивай ошибки с контекстом

```go
// ✅ ПРАВИЛЬНО
if err != nil {
    return fmt.Errorf("process user %s: %w", userID, err)
}

// ❌ НЕПРАВИЛЬНО
if err != nil {
    return err
}
```

### Определяй свои типы ошибок

```go
// ✅ ПРАВИЛЬНО
type ErrorCode int

const (
    ErrNotFound ErrorCode = iota + 1
    ErrInvalidInput
    ErrInternal
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
```

### Sentinel errors

```go
// ✅ ПРАВИЛЬНО
var ErrNotFound = errors.New("not found")
var ErrInvalidInput = errors.New("invalid input")

func GetUser(id string) (*User, error) {
    if id == "" {
        return nil, ErrInvalidInput
    }
    // ...
}
```

## Структуры

### Порядок полей

```go
// ✅ ПРАВИЛЬНО
type User struct {
    // Идентификаторы
    ID   UserID `json:"id"`
    Name string `json:"name"`

    // Настройки
    IsActive bool `json:"is_active"`

    // Внутреннее
    createdAt time.Time
}
```

### Constructor function

```go
// ✅ ПРАВИЛЬНО
func NewUser(name string, email Email) (*User, error) {
    if name == "" {
        return nil, ErrInvalidInput
    }
    return &User{
        Name:      name,
        Email:     email,
        createdAt: time.Now(),
    }, nil
}
```

### Functional options pattern

```go
// ✅ ПРАВИЛЬНО — для опциональных параметров
type Option func(*Config)

func WithPort(port int) Option {
    return func(c *Config) {
        c.Port = port
    }
}

func NewServer(opts ...Option) *Server {
    cfg := &Config{Port: 8080}
    for _, opt := range opts {
        opt(cfg)
    }
    return &Server{config: cfg}
}
```

## Интерфейсы

### Маленькие интерфейсы

```go
// ✅ ПРАВИЛЬНО — один метод
type Reader interface {
    Read(p []byte) (n int, err error)
}

type Writer interface {
    Write(p []byte) (n int, err error)
}

// ✅ ПРАВИЛЬНО — композиция
type ReadWriter interface {
    Reader
    Writer
}
```

### Принимай интерфейсы, возвращай структуры

```go
// ✅ ПРАВИЛЬНО
func SaveUser(db Storage, user *User) error {
    return db.Save(user)
}

// ❌ НЕПРАВИЛЬНО
func SaveUser(db *PostgresDB, user *User) error {
    return db.Save(user)
}
```

### Interface на стороне пользователя

```go
// ✅ ПРАВИЛЬНО — интерфейс там, где используется
type UserRepository interface {
    FindByID(id string) (*User, error)
    Save(user *User) error
}

type UserService struct {
    repo UserRepository
}
```

## Пакеты

### Структура

```text
internal/
├── user/
│   ├── service.go      // бизнес-логика
│   ├── repository.go   // работа с данными
│   └── model.go        // модели
├── http/
│   ├── handler.go      // HTTP handlers
│   └── middleware.go    // middleware
└── config/
    └── config.go       // конфигурация

cmd/
└── app/
    └── main.go         // точка входа
```

### Имена пакетов

```go
// ✅ ПРАВИЛЬНО
package user
package http
package config

// ❌ НЕПРАВИЛЬНО
package userService
package user_utils
```

## Тесты

### Table-driven tests

```go
// ✅ ПРАВИЛЬНО
func TestValidateEmail(t *testing.T) {
    tests := []struct {
        name  string
        input string
        want  bool
    }{
        {name: "valid email", input: "user@example.com", want: true},
        {name: "empty", input: "", want: false},
        {name: "no domain", input: "user@", want: false},
    }
    for _, tt := range tests {
        t.Run(tt.name, func(t *testing.T) {
            got := ValidateEmail(tt.input)
            if got != tt.want {
                t.Errorf("ValidateEmail(%q) = %v, want %v", tt.input, got, tt.want)
            }
        })
    }
}
```

### Subtests

```go
func TestUserService(t *testing.T) {
    t.Run("create", func(t *testing.T) {
        // test create
    })
    t.Run("find", func(t *testing.T) {
        // test find
    })
}
```

### Test helpers

```go
// ✅ ПРАВИЛЬНО
func mustCreateUser(t *testing.T, service *UserService, name string) *User {
    t.Helper()
    user, err := service.Create(name)
    if err != nil {
        t.Fatalf("Create(%q) = %v", name, err)
    }
    return user
}
```

## Комментарии

- **Английский** для всех комментариев
- **Doc comments** для экспортируемых сущностей

```go
// Package user provides business logic for user management.
package user

// User represents a registered user in the system.
type User struct {
    ID   UserID
    Name string
}

// CreateUser validates input and creates a new user.
//
// Returns ErrInvalidInput if name is empty.
func CreateUser(name string) (*User, error) {
    // implementation
}
```

## Ключевые Правила

1. **`gofmt`/`gofumpt`** перед каждым коммитом
1. **Всегда проверяй ошибки** — никогда не игнорируй
1. **Wrap errors** с контекстом через `fmt.Errorf("...: %w", err)
`
1. **Маленькие интерфейсы** — 1-3 метода
1. **Принимай интерфейсы, возвращай структуры
**
1. **Table-driven тесты** — стандартный паттерн
1. **Zero-initialization** — используй zero values
1. **Экспортируй только то, что нужно
**
1. **Английские комментарии
**
1. **camelCase** для неэкспортируемого, **PascalCase** для экспортируемого
