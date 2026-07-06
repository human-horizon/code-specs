# TDD: TypeScript

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

```typescript
Пример
// ❌ RED - тест падает
test('validates correct email', () => {
    const result = validateEmail('user@example.com')
    expect(result.ok).toBe(true)
})

// ✅ GREEN - минимум кода
function validateEmail(email: string): Result<string, Error> {
    return { ok: true, value: email }
}

// ❌ RED - следующий тест
test('rejects empty email', () => {
    const result = validateEmail('')
    expect(result.ok).toBe(false)
})

// ✅ GREEN - добавляем проверку
function validateEmail(email: string): Result<string, Error> {
    if (email === '') {
        return { ok: false, error: new Error('Empty email') }
    }
    return { ok: true, value: email }
}
```

```bash
Настройка Vitest
pnpm add -D vitest @vitest/coverage-v8
```

`
`

```typescript
vitest.config.ts:
import { defineConfig } from 'vitest/config'

export default defineConfig({
    test: {
        globals: true,
        environment: 'node',
        coverage: {
            provider: 'v8',
            reporter: ['text', 'html'],
            lines: 60,
            functions: 60,
        },
    },
})
```

```typescript
Структура тестов
import { describe, it, expect, beforeEach } from 'vitest'

describe('UserService', () => {
    let service: UserService

    beforeEach(() => {
        service = new UserService()
    })

    describe('createUser', () => {
        it('creates user with valid data', () => {
            const result = service.createUser({ name: 'Alice', age: 30 })
            expect(result.ok).toBe(true)
            if (result.ok) {
                expect(result.value.name).toBe('Alice')
            }
        })

        it('returns error for invalid age', () => {
            const result = service.createUser({ name: 'Bob', age: -5 })
            expect(result.ok).toBe(false)
        })
    })
})
```

## Молниеносные Тесты**
**

**КРИТИЧН**О: Тесты должны выполняться мгновенно!
| Тип | Максимальное время |
|-----|-------------------|
| Unit | < 10ms |
| Integration | < 100ms |

### |** Весь suit**e |** < 1 се**к |

```typescript
Моки вместо реальных зависимостей
import { vi } from 'vitest'

// ❌ НЕПРАВИЛЬНО - реальный fetch
test('fetches user', async () => {
    const user = await fetch('https://api.example.com/users/123')
})

// ✅ ПРАВИЛЬНО - мок
test('fetches user', async () => {
    const mockFetch = vi.fn().mockResolvedValue({ name: 'Alice' })
    global.fetch = mockFetch

    const user = await fetchUser('123')

    expect(mockFetch).toHaveBeenCalledWith('/users/123')
    expect(user.name).toBe('Alice')
})
```

```typescript
Fake Timers
test('delays execution', () => {
    vi.useFakeTimers()

    const callback = vi.fn()
    delayedCall(callback, 1000)

    vi.advanceTimersByTime(1000)
    expect(callback).toHaveBeenCalled()

    vi.useRealTimers()
})
```

```bash
In-Memory File System (memfs)
pnpm add -D memfs
```

```typescript

import { vol, fs } from 'memfs'

vi.mock('fs', () => ({ default: fs }))
vi.mock('fs/promises', () => fs.promises)

beforeEach(() => {
    vol.fromJSON({
        '/tmp/test.txt': 'content',
    })
})

afterEach(() => {
    vol.reset()
})

test('reads file', async () => {
    const content = await fs.promises.readFile('/tmp/test.txt', 'utf-8')
    expect(content).toBe('content')
})
```

### Лучшие практики

```typescript
AAA Pattern
test('adds items to cart', () => {
    // Arrange
    const cart = new ShoppingCart()
    const item = { id: '1', name: 'Book', price: 10 }

    // Act
    cart.addItem(item)

    // Assert
    expect(cart.items).toHaveLength(1)
    expect(cart.total).toBe(10)
})
```

```typescript
Тестируй поведение, не реализацию
// ✅ ПРАВИЛЬНО - поведение
test('sorts array in ascending order', () => {
    const result = sort([3, 1, 2])
    expect(result).toEqual([1, 2, 3])
})

// ❌ НЕПРАВИЛЬНО - реализация
test('uses quicksort algorithm', () => {
    const spy = vi.spyOn(SortAlgorithms, 'quicksort')
    sort([3, 1, 2])
    expect(spy).toHaveBeenCalled()
})
```

```typescript
Описательные имена
// ✅ ПРАВИЛЬНО
test('returns validation error when email is empty', () => {})

// ❌ НЕПРАВИЛЬНО
test('validation', () => {})
```

```bash
Команды
npm test              # Запустить тесты
npm test -- --watch   # Watch mode (обязательно!)
npm test -- --coverage # Отчёт о покрытии
```

## Чеклист

- Перед началом:
- [ ] Написал failing тест?
- [ ] Тест компилируется?
- [ ] Тест действительно падает (RED)?
- Перед коммитом:
- [ ] Все тесты GREEN?
- [ ] Coverage > 60%?

## [ ] Весь suite < 1 сек?

```json
Packages
{
    "devDependencies": {
        "vitest": "latest",
        "@vitest/coverage-v8": "latest",
        "memfs": "latest"
    },
    "scripts": {
        "test": "vitest",
        "test:watch": "vitest --watch",
        "test:coverage": "vitest --coverage"
    }
}
```
