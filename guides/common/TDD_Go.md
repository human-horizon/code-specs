# TDD: Go

## Философия

**Red-Green-Refactor** — единственный способ писать код.

1. **Никогда не пиши код без теста** — сначала тест, потом код
1. **Пиши минимум кода** — только чтобы тест прошёл
1. **Рефактори только на green** — все тесты должны проходить
1. **Молниеносные тесты** — < 1 секунда для всего suite

## Red-Green-Refactor

1. ❌ RED → Написать падающий тест
1. ✅ GREEN → Написать минимум кода для прохождения
1. 🔧 REFACTOR → Улучшить код (опционально)

### ↻ REPEAT → Следующий фича/тест

```go
Пример
// ❌ RED - тест падает
func TestValidateEmail_ValidEmail(t *testing.T) {
    result, err := ValidateEmail("user@example.com")
    if err != nil {
        t.Errorf("ValidateEmail() = %v, want nil", err)
    }
    if result != "user@example.com" {
        t.Errorf("got %q, want %q", result, "user@example.com")
    }
}

// ✅ GREEN - минимум кода
func ValidateEmail(email string) (string, error) {
    return email, nil
}

// ❌ RED - следующий тест
func TestValidateEmail_EmptyEmail(t *testing.T) {
    _, err := ValidateEmail("")
    if err == nil {
        t.Error("ValidateEmail('') = nil, want error")
    }
}

// ✅ GREEN - добавляем проверку
func ValidateEmail(email string) (string, error) {
    if email == "" {
        return "", errors.New("empty email")
    }
    return email, nil
}
```

```go
Структура тестов
import (
    "testing"
)

func TestUserService_CreateUser(t *testing.T) {
    service := NewUserService()

    t.Run("valid data", func(t *testing.T) {
        user, err := service.CreateUser("Alice", 30)
        if err != nil {
            t.Fatalf("CreateUser() = %v", err)
        }
        if user.Name != "Alice" {
            t.Errorf("user.Name = %q, want %q", user.Name, "Alice")
        }
    })

    t.Run("invalid age", func(t *testing.T) {
        _, err := service.CreateUser("Bob", -5)
        if err == nil {
            t.Error("CreateUser() = nil, want error")
        }
    })
}
```

## Молниеносные тесты**
**

**КРИТИЧН**О: Тесты должны выполняться мгновенно!
| Тип | Максимальное время |
|-----|-------------------|
| Unit | < 10ms |
| Integration | < 100ms |

### |** Весь suit**e |** < 1 се**к |

```go
Моки через интерфейсы
// Repository interface
type UserRepository interface {
    FindByID(id string) (*User, error)
    Save(user *User) error
}

// Mock implementation
type MockUserRepository struct {
    users map[string]*User
}

func NewMockUserRepository() *MockUserRepository {
    return &MockUserRepository{users: make(map[string]*User)}
}

func (m *MockUserRepository) FindByID(id string) (*User, error) {
    user, ok := m.users[id]
    if !ok {
        return nil, ErrNotFound
    }
    return user, nil
}

func (m *MockUserRepository) Save(user *User) error {
    m.users[user.ID] = user
    return nil
}

func TestUserService_FindByID(t *testing.T) {
    mockRepo := NewMockUserRepository()
    service := NewUserService(mockRepo)

    // Молниеносно — нет настоящей БД!
    user, err := service.FindByID("123")
    if err != ErrNotFound {
        t.Errorf("FindByID() = %v, want ErrNotFound", err)
    }
}
```

```go
Fake Timers
func TestDelayedExecution(t *testing.T) {
    // Используем настоящий таймер-канал для тестирования
    done := make(chan bool)
    go func() {
        time.Sleep(100 * time.Millisecond)
        done <- true
    }()

    select {
    case <-done:
        // ok
    case <-time.After(200 * time.Millisecond):
        t.Error("timeout")
    }
}
```

### Лучшие практики

```go
AAA Pattern
func TestShoppingCart_AddItem(t *testing.T) {
    // Arrange
    cart := NewShoppingCart()
    item := Item{ID: "1", Name: "Book", Price: 10}

    // Act
    err := cart.AddItem(item)

    // Assert
    if err != nil {
        t.Fatalf("AddItem() = %v", err)
    }
    if len(cart.Items) != 1 {
        t.Errorf("len(cart.Items) = %d, want 1", len(cart.Items))
    }
    if cart.Total != 10 {
        t.Errorf("cart.Total = %d, want 10", cart.Total)
    }
}
```

```go
Тестируй поведение, не реализацию
// ✅ ПРАВИЛЬНО - поведение
func TestSort_AscendingOrder(t *testing.T) {
    result := Sort([]int{3, 1, 2})
    expected := []int{1, 2, 3}
    if !reflect.DeepEqual(result, expected) {
        t.Errorf("got %v, want %v", result, expected)
    }
}
```

```go
Описательные имена
// ✅ ПРАВИЛЬНО
func TestValidateEmail_ReturnsError_WhenEmailIsEmpty(t *testing.T) {}

// ❌ НЕПРАВИЛЬНО
func TestValidateEmail(t *testing.T) {}
```

```go
Table-Driven Tests
func TestValidateEmail(t *testing.T) {
    tests := []struct {
        name    string
        input   string
        wantErr bool
    }{
        {name: "valid email", input: "user@example.com", wantErr: false},
        {name: "empty", input: "", wantErr: true},
        {name: "no domain", input: "user@", wantErr: true},
        {name: "no at sign", input: "userexample.com", wantErr: true},
    }

    for _, tt := range tests {
        t.Run(tt.name, func(t *testing.T) {
            _, err := ValidateEmail(tt.input)
            if (err != nil) != tt.wantErr {
                t.Errorf("ValidateEmail(%q) = err=%v, wantErr=%v",
                    tt.input, err, tt.wantErr)
            }
        })
    }
}
```

```go
Test Helpers
func mustCreateUser(t *testing.T, service *UserService, name string) *User {
    t.Helper()
    user, err := service.Create(name)
    if err != nil {
        t.Fatalf("Create(%q) = %v", name, err)
    }
    return user
}
```

```bash
Команды
go test ./...              # Запустить все тесты
go test -v ./...           # Подробный вывод
go test -run TestName      # Конкретный тест
go test -cover ./...       # Покрытие
```

## Чеклист

- Перед началом:
- [ ] Написал failing тест?
- [ ] Тест компилируется?
- [ ] Тест действительно падает (RED)?
- Перед коммитом:
- [ ] Все тесты GREEN?
- [ ] Все public функции покрыты?

## [ ] Весь suite < 1 сек?

## Ключевые правила**
**

- **Red → Green → Refacto**r**
**
- **Нет кода без тест**а**
**
- **Молниеносные тест**ы — < 1 сек**
**
- **Table-driven тест**ы — стандартный паттерн**
**
- **Моки через интерфейс**ы — без реальных I/O**
**
- **Описательные имена тесто**в — TestXxxYyyZzz**
**
- **Subtest**s для группировки**
**
- **t.Helper(**) в хелперах
