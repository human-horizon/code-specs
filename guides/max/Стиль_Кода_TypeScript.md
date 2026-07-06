# Стиль Кода TypeScript (дополнение)

> Строгий code style для TypeScript-проектов.
---
**База**: см. [Стиль_Кода_TypeScript.md](http://../common/Стиль_Кода_TypeScript.md)
---

## Дополнительные разделы

## Содержание

1. [Классы](#классы)
1. [Структура файлов](#структура-файлов)
1. [Расширенные паттерны](#расширенные-паттерны)

---

## Классы

### Порядок членов

```typescript
// ✅ ПРАВИЛЬНО
class UserService {
    // 1. Private fields
    private readonly logger: Logger
    private readonly api: ApiClient

    // 2. Constructor
    constructor(config: Config) {
        this.logger = new Logger(config)
        this.api = new ApiClient(config)
    }

    // 3. Public methods
    public async createUser(data: CreateUserDto): Promise<Result<User>> {
        // ...
    }

    // 4. Private methods
    private validateInput(data: unknown): data is CreateUserDto {
        // ...
    }
}
```

### Не используй классы для данных

```typescript
// ❌ НЕПРАВИЛЬНО
class User {
    constructor(
        public readonly name: string,
        public readonly email: string
    ) {}
}

// ✅ ПРАВИЛЬНО - use interfaces/type aliases
type User = {
    readonly name: string
    readonly email: Email
}
```

---

## Структура файлов

### Feature-Sliced Design (кратко)

```text
src/
├── app/              # App initialization
├── pages/            # Route pages
├── widgets/          # Independent blocks
├── features/        # User scenarios
├── entities/        # Business entities
└── shared/          # Reusable code
```

### Barrel files

```typescript
// ✅ ПРАВИЛЬНО - явный экспорт
export { UserService } from './UserService'
export type { User, CreateUserDto } from './types'

// ❌ НЕПРАВИЛЬНО - re-export everything
export * from './UserService'
```

---

## Расширенные паттерны

### Builder Pattern

```typescript
class QueryBuilder {
    private query: string = ''
    private params: unknown[] = []

    select(fields: string[]): this {
        this.query = `SELECT ${fields.join(', ')}`
        return this
    }

    from(table: string): this {
        this.query += ` FROM ${table}`
        return this
    }

    where(condition: string): this {
        this.query += ` WHERE ${condition}`
        return this
    }

    build(): { query: string; params: unknown[] } {
        return { query: this.query, params: this.params }
    }
}
```

### Repository Pattern

```typescript
interface Repository<T> {
    findById(id: string): Promise<Result<T>>
    findAll(): Promise<Result<T[]>>
    save(entity: T): Promise<Result<T>>
    delete(id: string): Promise<Result<void>>
}

class UserRepository implements Repository<User> {
    constructor(private readonly db: Database) {}

    async findById(id: string): Promise<Result<User>> {
        // implementation
    }
}
```
