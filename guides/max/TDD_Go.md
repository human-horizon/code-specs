# TDD: Go (дополнение)

> Строгий TDD-подход для Go.
---
**База**: см. [TDD_Go.md](http://../common/TDD_Go.md)
---

## Дополнительные разделы

## Содержание

1. [Пирамида тестов](#пирамида-тестов)
1. [Покрытие кода](#покрытие-кода)
1. [Интеграционные тесты](#интеграционные-тесты)

---

## Пирамида тестов

### 60% Unit / 30% Integration / 10% E2E

```text
       /\
      /  \      E2E (10%)
     /----\
    /      \   Integration (30%)
   /--------\
  /          \ Unit (60%)
 /************\
```

### Unit-тесты

```go
func TestParseEmail_ReturnsOK_ForValidEmail(t *testing.T) {
    result, err := ParseEmail("test@example.com")
    if err != nil {
        t.Errorf("ParseEmail() = %v, want nil", err)
    }
    if result != "test@example.com" {
        t.Errorf("got %q, want %q", result, "test@example.com")
    }
}
```

### Интеграционные тесты

```go
func TestUserRepository_SavesAndRetrieves(t *testing.T) {
    db := setupTestDB(t)
    defer db.Close()

    repo := NewUserRepository(db)
    user := &User{Name: "Alice"}
    err := repo.Save(user)
    if err != nil {
        t.Fatalf("Save() = %v", err)
    }

    found, err := repo.FindByID(user.ID)
    if err != nil {
        t.Fatalf("FindByID() = %v", err)
    }
    if found.Name != "Alice" {
        t.Errorf("found.Name = %q, want %q", found.Name, "Alice")
    }
}
```

---

## Покрытие кода

### Сборка с coverage

```bash
go test -coverprofile=coverage.out ./...
go tool cover -html=coverage.out -o coverage.html
```

### Покрытие > 80%

```bash
go test -coverprofile=coverage.out -covermode=atomic ./...
go tool cover -func=coverage.out | grep total
```

### Должны быть покрыты

- Все экспортированные функции
- Все кейсы ошибок
- Граничные случаи
- Пути ошибок

---

## Интеграционные тесты

### Тестирование с базой данных

```go
func TestUserRepository_PersistsAcrossRestart(t *testing.T) {
    db, err := sql.Open("postgres", "postgres://test:test@localhost/test?sslmode=disable")
    if err != nil {
        t.Fatalf("sql.Open() = %v", err)
    }
    defer db.Close()

    repo := NewUserRepository(db)
    user := &User{Name: "Alice"}
    err = repo.Save(user)
    if err != nil {
        t.Fatalf("Save() = %v", err)
    }

    // Симулируем переподключение
    db.Close()
    db, err = sql.Open("postgres", "postgres://test:test@localhost/test?sslmode=disable")
    if err != nil {
        t.Fatalf("sql.Open() = %v", err)
    }
    repo = NewUserRepository(db)

    found, err := repo.FindByID(user.ID)
    if err != nil {
        t.Fatalf("FindByID() = %v", err)
    }
    if found.Name != "Alice" {
        t.Errorf("found.Name = %q, want %q", found.Name, "Alice")
    }
}
```

### Тестирование HTTP

```go
func TestServer_HandlesPostUsers(t *testing.T) {
    handler := NewUserHandler(NewUserService(NewMockUserRepository()))
    server := httptest.NewServer(handler)
    defer server.Close()

    body := `{"name": "Alice"}`
    resp, err := http.Post(server.URL+"/users", "application/json", strings.NewReader(body))
    if err != nil {
        t.Fatalf("POST /users = %v", err)
    }
    defer resp.Body.Close()

    if resp.StatusCode != http.StatusCreated {
        t.Errorf("status = %d, want %d", resp.StatusCode, http.StatusCreated)
    }
}
```

### Тесты файловой системы

```go
func TestConfig_LoadsFromFile(t *testing.T) {
    content := `{"port": 8080, "debug": true}`
    tmpFile := filepath.Join(t.TempDir(), "config.json")
    if err := os.WriteFile(tmpFile, []byte(content), 0644); err != nil {
        t.Fatalf("WriteFile() = %v", err)
    }

    cfg, err := LoadConfigFromFile(tmpFile)
    if err != nil {
        t.Fatalf("LoadConfigFromFile() = %v", err)
    }
    if cfg.Port != 8080 {
        t.Errorf("Port = %d, want %d", cfg.Port, 8080)
    }
    if !cfg.Debug {
        t.Error("Debug = false, want true")
    }
}
```

### Сетевые тесты

```go
func TestTCPClient_ConnectsToServer(t *testing.T) {
    listener, err := net.Listen("tcp", "localhost:0")
    if err != nil {
        t.Fatalf("Listen() = %v", err)
    }
    defer listener.Close()

    go func() {
        conn, err := listener.Accept()
        if err != nil {
            return
        }
        defer conn.Close()
        io.Copy(conn, conn) // echo
    }()

    conn, err := net.Dial("tcp", listener.Addr().String())
    if err != nil {
        t.Fatalf("Dial() = %v", err)
    }
    defer conn.Close()

    msg := "hello"
    _, err = conn.Write([]byte(msg))
    if err != nil {
        t.Fatalf("Write() = %v", err)
    }

    buf := make([]byte, 64)
    n, err := conn.Read(buf)
    if err != nil {
        t.Fatalf("Read() = %v", err)
    }
    if string(buf[:n]) != msg {
        t.Errorf("got %q, want %q", string(buf[:n]), msg)
    }
}
```

---

## Ключевые правила (дополнение)

1. **Unit-тесты first** — 60% тестов
1. **Integration second** — 30%
1. **E2E last** — 10%
1. **Coverage > 80%** для продакшена
1. **Используй интерфейсы** для test doubles
1. **Настоящие интеграционные тесты** — testcontainers или temp-ресурсы
1. **Cleanup после каждого теста** — t.Cleanup / defer
1. **Изолированные тесты** — нет зависимостей между тестами
