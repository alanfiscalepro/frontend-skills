# FSD Frontend Skill

A Claude Code skill that generates production-ready frontend code for React and Next.js projects. Just describe what you need in plain language - no architecture knowledge required.

---

## Installation / Установка

### English

1. Open Claude Code settings:
   ```bash
   claude config
   ```

2. Add the skill to your configuration. Edit `~/.claude/settings.json`:
   ```json
   {
     "skills": [
       {
         "name": "fsd-frontend",
         "url": "https://github.com/alanfiscalepro/frontend-skills"
       }
     ]
   }
   ```

3. Alternatively, clone directly to your skills folder:
   ```bash
   git clone https://github.com/alanfiscalepro/frontend-skills.git ~/.claude/skills/fsd-frontend
   ```

4. Restart Claude Code to load the skill.

### Русский

1. Откройте настройки Claude Code:
   ```bash
   claude config
   ```

2. Добавьте скилл в конфигурацию. Отредактируйте `~/.claude/settings.json`:
   ```json
   {
     "skills": [
       {
         "name": "fsd-frontend",
         "url": "https://github.com/alanfiscalepro/frontend-skills"
       }
     ]
   }
   ```

3. Или клонируйте напрямую в папку скиллов:
   ```bash
   git clone https://github.com/alanfiscalepro/frontend-skills.git ~/.claude/skills/fsd-frontend
   ```

4. Перезапустите Claude Code для загрузки скилла.

---

## Usage / Использование

### English

Just describe what you need in plain language. The skill will generate all necessary files following best practices.

#### Create Entities (Data Types)

```
Create a User with name, email, and avatar
```

```
Create a Product entity with title, price, description, category, and images
```

```
Create an Order with items, total, status, and customerInfo
```

#### Create Components

```
Create a card component for Product
```

```
Create a list component for Users with pagination
```

```
Create a form for creating and editing Products
```

#### Setup Project

```
Setup API client for my Next.js project
```

```
Setup React Query provider
```

#### Create Features

```
Create a login feature with form and validation
```

```
Create a shopping cart feature
```

### Русский

Просто опишите что вам нужно на обычном языке. Скилл сгенерирует все необходимые файлы следуя лучшим практикам.

#### Создание сущностей (типов данных)

```
Создай сущность User с полями name, email и avatar
```

```
Создай сущность Product с полями title, price, description, category и images
```

```
Создай сущность Order с полями items, total, status и customerInfo
```

#### Создание компонентов

```
Создай компонент карточки для Product
```

```
Создай компонент списка для Users с пагинацией
```

```
Создай форму для создания и редактирования Products
```

#### Настройка проекта

```
Настрой API клиент для моего Next.js проекта
```

```
Настрой провайдер React Query
```

#### Создание фич

```
Создай фичу логина с формой и валидацией
```

```
Создай фичу корзины покупок
```

---

## What Gets Generated / Что генерируется

### English

When you create an entity (e.g., "Create a Product with name and price"), you get:

```
src/entities/Product/
├── api/
│   ├── index.ts        # Exports
│   ├── types.ts        # IProduct, ICreateProductDTO, IUpdateProductDTO
│   ├── services.ts     # fetchProducts, createProduct, updateProduct, deleteProduct
│   ├── queries.ts      # useProducts, useProduct, useInfiniteProducts
│   └── mutations.ts    # useCreateProduct, useUpdateProduct, useDeleteProduct
├── ui/
│   ├── ProductCard.tsx
│   ├── ProductList.tsx
│   ├── ProductForm.tsx
│   └── index.ts
└── index.ts            # Public API
```

### Русский

Когда вы создаете сущность (например, "Создай Product с name и price"), вы получаете:

```
src/entities/Product/
├── api/
│   ├── index.ts        # Экспорты
│   ├── types.ts        # IProduct, ICreateProductDTO, IUpdateProductDTO
│   ├── services.ts     # fetchProducts, createProduct, updateProduct, deleteProduct
│   ├── queries.ts      # useProducts, useProduct, useInfiniteProducts
│   └── mutations.ts    # useCreateProduct, useUpdateProduct, useDeleteProduct
├── ui/
│   ├── ProductCard.tsx
│   ├── ProductList.tsx
│   ├── ProductForm.tsx
│   └── index.ts
└── index.ts            # Публичный API
```

---

## Supported Technologies / Поддерживаемые технологии

| Category / Категория | Technologies / Технологии |
|---------------------|---------------------------|
| Frameworks | React, Next.js 13/14/15 (App Router, Pages Router) |
| Styling | Tailwind CSS, CSS Modules, styled-components, any UI library |
| State Management | React Query (TanStack Query v5) |
| Forms | React Hook Form |
| HTTP Client | Axios |

---

## Examples / Примеры

### Example 1: E-commerce Product

**English:**
```
Create a Product entity with:
- name (string)
- price (number)
- description (string)
- category (string)
- images (string array)
- inStock (boolean)
```

**Русский:**
```
Создай сущность Product с полями:
- name (строка)
- price (число)
- description (строка)
- category (строка)
- images (массив строк)
- inStock (булево)
```

### Example 2: User Profile

**English:**
```
Create a User with name, email, avatar, role, and createdAt
```

**Русский:**
```
Создай User с полями name, email, avatar, role и createdAt
```

### Example 3: Full Feature

**English:**
```
Create a checkout feature with:
- Cart summary component
- Shipping form
- Payment integration
- Order confirmation
```

**Русский:**
```
Создай фичу оформления заказа с:
- Компонентом корзины
- Формой доставки
- Интеграцией оплаты
- Подтверждением заказа
```

---

## Project Setup / Настройка проекта

### English

Before using generated code, ensure your project has:

1. **Install dependencies:**
   ```bash
   npm install @tanstack/react-query axios
   npm install -D @tanstack/react-query-devtools
   ```

2. **Setup path aliases** in `tsconfig.json`:
   ```json
   {
     "compilerOptions": {
       "baseUrl": ".",
       "paths": {
         "@/*": ["./src/*"]
       }
     }
   }
   ```

3. **Ask Claude to setup providers:**
   ```
   Setup React Query provider for my Next.js 15 project
   Setup API client with axios
   ```

### Русский

Перед использованием сгенерированного кода убедитесь что в проекте есть:

1. **Установите зависимости:**
   ```bash
   npm install @tanstack/react-query axios
   npm install -D @tanstack/react-query-devtools
   ```

2. **Настройте алиасы путей** в `tsconfig.json`:
   ```json
   {
     "compilerOptions": {
       "baseUrl": ".",
       "paths": {
         "@/*": ["./src/*"]
       }
     }
   }
   ```

3. **Попросите Claude настроить провайдеры:**
   ```
   Настрой провайдер React Query для моего Next.js 15 проекта
   Настрой API клиент с axios
   ```

---

## Tips / Советы

### English

1. **Be specific about fields** - "Create User with name (string), age (number), isActive (boolean)"
2. **Request only what you need** - "Create only the API layer for Orders"
3. **Specify styling preference** - "Create ProductCard using Tailwind CSS"
4. **Mention your UI library** - "Create a form for User using shadcn/ui components"

### Русский

1. **Указывайте типы полей** - "Создай User с name (string), age (number), isActive (boolean)"
2. **Запрашивайте только нужное** - "Создай только API слой для Orders"
3. **Указывайте стиль** - "Создай ProductCard используя Tailwind CSS"
4. **Упоминайте UI библиотеку** - "Создай форму для User используя компоненты shadcn/ui"

---

## Architecture / Архитектура

This skill follows **Feature-Sliced Design (FSD)** architecture:

```
src/
├── app/          # App configuration, providers
├── pages/        # Page components (or app/ for Next.js App Router)
├── widgets/      # Complex UI blocks combining features
├── features/     # User interactions (login, checkout, etc.)
├── entities/     # Business entities (User, Product, Order)
└── shared/       # Shared utilities, UI kit, API client
```

**Import rules / Правила импорта:**
- Upper layers can import from lower layers only
- `features` → `entities` → `shared`
- Cross-imports within the same layer are forbidden

---

## License / Лицензия

MIT
