# Стиль Кода TypeScript

## Форматирование

- **4 пробела** для отступов
- **Без точек с запятой
**
- **Одинарные кавычки** для строк
- **Висячие запятые** в многострочных структурах
- **100 символов** максимум в строке

```typescript
const obj = {
    name: 'John',
    age: 30,
}

function example() {
    if (condition) {
        doSomething()
    }
}
```

## Именование

| Что | Стиль | Пример |
|-----|-------|--------|
| Переменные, функции | camelCase | userName, getUserName |
| Классы, интерфейсы | PascalCase | UserProfile, BoxContract |
| Константы | UPPER_SNAKE_CASE | MAX_RETRY_COUNT |
| Файлы | Соответствуют сущности | Box.ts, user-helpers.ts |

## Система Типов

### Никогда any

```typescript
// ❌ ЗАПРЕЩЕНО
function process(data: any): string {
    return data.toUpperCase()
}

// ✅ ПРАВИЛЬНО
function process(data: unknown): string {
    if (typeof data === 'string') {
        return data.toUpperCase()
    }
    throw new Error('Invalid data')
}
```

### Никогда as (кроме as const)

```typescript
// ❌ ЗАПРЕЩЕНО
const box = value as Box

// ✅ ПРАВИЛЬНО
if (isBox(value)) {
    console.log(value.x)
}

// ✅ ПРАВИЛЬНО - as const
const config = { name: 'app' } as const
```

### Явные возвращаемые типы

```typescript
// ❌ ЗАПРЕЩЕНО
function getUser(id: string) {
    return { name: 'John' }
}

// ✅ ПРАВИЛЬНО
function getUser(id: string): User {
    return { name: 'John' }
}
```

### Type Guards

```typescript
function isString(value: unknown): value is string {
    return typeof value === 'string'
}

function assertIsString(value: unknown): asserts value is string {
    if (typeof value !== 'string') {
        throw new Error('Must be string')
    }
}
```

## Функции

### Объявления на верхнем уровне

```typescript
// ✅ ПРАВИЛЬНО
function calculateTotal(items: Item[]): number {
    return items.reduce((sum, item) => sum + item.price, 0)
}

// ❌ НЕПРАВИЛЬНО
const calculateTotal = (items: Item[]): number => {
    return items.reduce((sum, item) => sum + item.price, 0)
}
```

### Деструктуризация для >3 параметров

```typescript
// ✅ ПРАВИЛЬНО - 3 или меньше
function createPoint(x: number, y: number, z: number): Point {
    return { x, y, z }
}

// ✅ ПРАВИЛЬНО - больше 3
interface BoxParams {
    x: number
    y: number
    width: number
    height: number
}

function createBox({ x, y, width, height }: BoxParams): Box {
    return new Box(x, y, width, height)
}
```

## Интерфейсы и Типы

### interface для объектов, type для unions

```typescript
// ✅ interface для объектов
interface User {
    name: string
    age: number
}

// ✅ type для unions
type Status = 'active' | 'inactive' | 'pending'

// ✅ type для intersections
type UserWithId = User & { id: string }
```

### Без префикса I

```typescript
// ✅ ПРАВИЛЬНО
interface User {}
interface BoxContract {}

// ❌ НЕПРАВИЛЬНО
interface IUser {}
```

## Импорты

### import type для типов

```typescript
import type { User } from './types'
import type { BoxContract } from './Box.contract'
import { Box } from './Box'
```

### Группировка импортов

```typescript
import { useState } from 'react'
import type { FC } from 'react'

import { Box } from './Box'
import type { BoxContract } from './Box.contract'

import { helper } from '../utils'
```

## Паттерны

### Result тип для ошибок

```typescript
type Result<T, E = Error> =
    | { ok: true; value: T }
    | { ok: false; error: E }

async function fetchUser(id: string): Promise<Result<User>> {
    try {
        const response = await fetch(`/users/${id}`)
        if (!response.ok) {
            return { ok: false, error: new Error(`HTTP ${response.status}`) }
        }
        return { ok: true, value: await response.json() }
    } catch (error) {
        return { ok: false, error: error as Error }
    }
}
```

### Union типы вместо enum

```typescript
// ✅ ПРАВИЛЬНО
type Status = 'active' | 'inactive' | 'pending'

// ❌ НЕПРАВИЛЬНО
enum Status {
    Active = 'active',
    Inactive = 'inactive',
}
```

## Конфигурация

### Prettier

```json
{
    "tabWidth": 4,
    "semi": false,
    "singleQuote": true,
    "trailingComma": "all",
    "printWidth": 100
}
```

### TSConfig

```json
{
    "compilerOptions": {
        "strict": true,
        "noImplicitAny": true,
        "noUncheckedIndexedAccess": true
    }
}
```

## Ключевые Правила

1. 4 пробела, без точек с запятой
1. Никогда `any` — только `unknown
`
1. Никогда `as` — только type guards
1. Явные возвращаемые типы
1. `interface` для объектов, `type` для unions
1. `import type` для типов
1. Result тип для ошибок
1. Union типы вместо enum
