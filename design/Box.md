# Box

## Роль

Базовый примитив UI Kit. Все остальные компоненты строятся на Box. Аналог `<div>`, но с расширенным API для стилизации и полиморфизма.

## API

### Props

| Prop | Тип | Описание |
| :---- | :---- | :---- |
| sx | string | Tailwind-классы |
| as | keyof JSX.IntrinsicElements | Рендерит как указанный тег (по умолчанию div) |
| asChild | boolean | Не рендерит свой тег, прокидывает все пропы (включая sx как className) на child-элемент |
| children | ReactNode | Дочерние элементы |
| ref | Ref | React ref |
| ...HTML attributes | — | Все стандартные HTML-атрибуты (id, style, data-*, role, aria-*, event handlers) |

### PandaCSS-style props

Все CSS-свойства доступны как пропы напрямую:

```tsx
<Box padding="4" backgroundColor="red.500" display="flex" gap="2">
  content
</Box>
```

Значения — токены дизайн-системы (`red.500`, `spacing.4`, `lg` и т.д.).
Responsive значения через объект:

```tsx
<Box padding={{ base: "4", md: "8", lg: "12" }}>
  responsive
</Box>
```

### Псевдо-селекторы

Через PandaCSS-style пропы:

```tsx
<Box
  backgroundColor="blue.500"
  _hover={{ backgroundColor: "blue.600" }}
  _active={{ backgroundColor: "blue.700" }}
/>
```

### Комбинация sx и PandaCSS-пропов

`sx` (Tailwind) и PandaCSS-пропы можно использовать вместе. При конфликте — sx имеет приоритет (Tailwind utility-class specificity).

```tsx
<Box display="flex" sx="items-center gap-2 hover:bg-red-500">
  mixed
</Box>
```

## Поведение

### as

```tsx
<Box as="section">  // <section>...</section>
<Box as="a" href="/"> // <a href="/">...</a>
```

### asChild

Работает как в Chakra UI v3 (Slot-паттерн). Компонент не рендерит свой тег — клонирует единственный child и мержит на него все свои пропы (className, style, event handlers, ref). Child получает полное поведение Box, но рендерится своим тегом.

```tsx
<Box asChild sx="p-4 bg-red-500">
  <button>Click</button>
</Box>
// Результат: <button class="p-4 bg-red-500">Click</button>
```

Если child уже имеет className — классы мержатся.

### as + asChild

`asChild` переопределяет `as`. Если оба указаны, используется только `asChild`.

## Ограничения

- `asChild` принимает ровно один child-элемент (иначе ошибка)
- `as` не принимает asChild-компоненты (только HTML-теги)
- PandaCSS-пропы резолвятся через дизайн-токены, не raw CSS-значения
