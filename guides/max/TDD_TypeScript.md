# NOTE

Stryker setup must use the approved TypeScript test stack. Vitest is the default test runner; Jest runner is not allowed unless separately approved in decisions/.

# TDD: TypeScript (дополнение)

> Строгий TDD-подход с mutation testing.
---
**База**: см. [TDD_TypeScript.md](http://../common/TDD_TypeScript.md)
---

## Дополнительные разделы

## Содержание

1. [Пирамида тестов](#пирамида-тестов)
1. [Mutation testing](#mutation-testing)
1. [Покрытие 80%+](#покрытие-80)
1. [Интеграционные тесты](#интеграционные-тесты)

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

### Unit-тесты (< 10ms)

```typescript
// ✅ unit тест - тестирует одну функцию
describe('validateEmail', () => {
    it('rejects invalid email', () => {
        const result = validateEmail('invalid')
        expect(result.ok).toBe(false)
    })
})
```

### Интеграционные тесты (10-100ms)

```typescript
// ✅ integration - несколько компонентов
describe('UserService + Database', () => {
    it('creates and retrieves user', async () => {
        const created = await userService.create({ name: 'Alice' })
        const retrieved = await userRepository.findById(created.id)
        expect(retrieved.name).toBe('Alice')
    })
})
```

## Mutation testing

### Зачем?

Мутационное тестирование проверяет качество тестов — убивает мутантов (изменённый код) и смотрит, ловят ли тесты эти изменения.

### Настройка Stryker

```bash
pnpm add -D @stryker-mutator/core @stryker-mutator/typescript-checker @stryker-mutator/vitest-runner
```

`stryker.conf.json`:

```json
{
    "$schema": "./node_modules/@stryker-mutator/core/schema/stryker-schema.json",
    "mutator": "typescript",
    "packageManager": "npm",
    "reporters": ["html", "clear-text", "dashboard"],
    "checkers": ["typescript"],
    "tsconfig": "./tsconfig.json",
    "mutate": [
        "src/**/*.ts"
    ],
    "jest": {
        "projectType": "ts-jest"
    },
    "thresholds": {
        "high": 80,
        "low": 70
    }
}
```

`package.json`:

```json
{
    "scripts": {
        "test:mutation": "stryker run"
    }
}
```

### Интерпретация результатов

| Mutation Score | Качество тестов |
|----------------|-----------------|
| > 80% | Отлично |
| 70-80% | Хорошо |
| < 70% | Плохо — добавь тестов |
---

## Покрытие 80%+

### Покрытие > 80% обязательно

```json
{
    "coverage": {
        "provider": "v8",
        "reporter": ["text", "html"],
        "lines": 80,
        "functions": 80,
        "branches": 80,
        "statements": 80
    }
}
```

### Что покрывать обязательно

- Все public функции
- Все ветвления (if/else, switch)
- Все кейсы ошибок
- Граничные случаи

### CI/CD

```yaml
- name: Coverage
  run: |
    npm run test:coverage
    npx stryker run
- name: Check Coverage
  run: |
    npx jest-coverage-badges --inputDir ./coverage
```

---

## Интеграционные тесты

### Тестирование с настоящей БД (в Docker)

```typescript
describe('UserRepository (integration)', () => {
    let db: Database

    beforeAll(async () => {
        db = await startDockerDatabase()
    })

    afterAll(async () => {
        await db.stop()
    })

    it('persists user', async () => {
        const user = await repo.save({ name: 'Alice' })
        const found = await repo.findById(user.id)
        expect(found.name).toBe('Alice')
    })
})
```

### Тестирование API

```typescript
describe('POST /users (integration)', () => {
    const app = createApp()

    it('creates user', async () => {
        const response = await app.request('/users', {
            method: 'POST',
            body: JSON.stringify({ name: 'Alice' })
        })
        expect(response.status).toBe(201)
    })
})
```
