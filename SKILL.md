---
name: fsd-frontend
description: Generate frontend code for React or Next.js projects. Creates API layers, components, forms, and data fetching hooks. Just describe what you need - no architecture knowledge required.
---

# FSD Frontend Architecture Skill

## Simple User Commands

Users can request code generation with simple natural language:

- "Create a User with name, email, and avatar"
- "Create a Product entity with title, price, and images"
- "Create a card component for Product"
- "Create a form for Orders"
- "Setup API client"
- "Setup React Query provider"

**The skill handles all architecture decisions automatically.**

---

## Overview

This skill scaffolds Feature-Sliced Design (FSD) architecture for React and Next.js 15 projects using React Query (TanStack Query v5) for server state management.

## FSD Layer Structure

```
src/
├── app/                    # Application layer (providers, routing, global styles)
│   ├── providers/          # React context providers, QueryClient setup
│   ├── styles/             # Global styles
│   └── index.tsx           # App entry point
│
├── pages/                  # Pages layer (Next.js: app/ directory)
│   └── [page-name]/        # Page components, layouts
│
├── widgets/                # Widget layer (composite UI blocks)
│   └── [widget-name]/
│       ├── ui/             # Widget components
│       ├── model/          # Widget-specific state/logic
│       └── index.ts        # Public API
│
├── features/               # Feature layer (user interactions)
│   └── [feature-name]/
│       ├── api/            # Feature-specific API calls
│       ├── ui/             # Feature UI components
│       ├── model/          # Feature state/logic
│       ├── lib/            # Feature utilities
│       └── index.ts        # Public API
│
├── entities/               # Entity layer (business entities)
│   └── [entity-name]/
│       ├── api/            # API layer (services, queries, mutations, types)
│       │   ├── index.ts
│       │   ├── services.ts
│       │   ├── queries.ts
│       │   ├── mutations.ts
│       │   └── types.ts
│       ├── ui/             # Entity UI components
│       ├── model/          # Entity state/logic
│       ├── lib/            # Entity utilities/hooks
│       └── index.ts        # Public API
│
└── shared/                 # Shared layer (reusable utilities)
    ├── api/                # API client, interceptors
    ├── ui/                 # UI kit components
    ├── lib/                # Utilities, helpers
    ├── config/             # Configuration
    ├── types/              # Shared types
    └── const/              # Constants
```

## Instructions

When scaffolding FSD architecture:

1. **Identify the layer** - Determine if creating an entity, feature, widget, or shared module
2. **Create directory structure** - Follow the standard structure for that layer
3. **Generate API layer** - For entities/features that need data fetching
4. **Create UI components** - Follow component patterns below
5. **Export public API** - Use barrel exports (index.ts)
6. **Connect layers** - Follow FSD import rules (only import from lower layers)

## API Layer Pattern

### Directory Structure
```
entities/[EntityName]/api/
├── index.ts       # Barrel exports
├── services.ts    # API service functions
├── queries.ts     # useQuery hooks + query keys
├── mutations.ts   # useMutation hooks
└── types.ts       # TypeScript interfaces
```

### services.ts Template
```typescript
import { api } from "@/shared/api";
import type { IEntityResponse, IEntity, ICreateEntityDTO, IUpdateEntityDTO } from "./types";

const BASE_URL = "/api/entities";

export const fetchEntities = async (params?: Record<string, unknown>) => {
  const response = await api.get<IEntityResponse<IEntity[]>>(BASE_URL, { params });
  return response.data.result;
};

export const fetchEntityById = async (id: number | string) => {
  const response = await api.get<IEntityResponse<IEntity>>(`${BASE_URL}/${id}`);
  return response.data.result;
};

export const createEntity = async (data: ICreateEntityDTO) => {
  const response = await api.post<IEntityResponse<IEntity>>(BASE_URL, data);
  return response.data.result;
};

export const updateEntity = async ({ id, data }: { id: number | string; data: IUpdateEntityDTO }) => {
  const response = await api.patch<IEntityResponse<IEntity>>(`${BASE_URL}/${id}`, data);
  return response.data.result;
};

export const deleteEntity = async (id: number | string) => {
  const response = await api.delete(`${BASE_URL}/${id}`);
  return response.data;
};
```

### queries.ts Template
```typescript
import { useQuery, useInfiniteQuery, type UseQueryOptions } from "@tanstack/react-query";
import { fetchEntities, fetchEntityById } from "./services";
import type { IEntity, IPaginatedResponse } from "./types";

// Query Keys
export const entityKeys = {
  all: ["entities"] as const,
  lists: () => [...entityKeys.all, "list"] as const,
  list: (params?: Record<string, unknown>) => [...entityKeys.lists(), params] as const,
  details: () => [...entityKeys.all, "detail"] as const,
  detail: (id: number | string) => [...entityKeys.details(), id] as const,
};

// List Query
export const useEntities = (
  params?: Record<string, unknown>,
  options?: Omit<UseQueryOptions<IPaginatedResponse<IEntity[]>>, "queryKey" | "queryFn">
) => {
  return useQuery({
    queryKey: entityKeys.list(params),
    queryFn: () => fetchEntities(params),
    ...options,
  });
};

// Detail Query
export const useEntity = (
  id: number | string,
  options?: Omit<UseQueryOptions<IEntity>, "queryKey" | "queryFn">
) => {
  return useQuery({
    queryKey: entityKeys.detail(id),
    queryFn: () => fetchEntityById(id),
    enabled: Boolean(id),
    ...options,
  });
};

// Infinite Query (for pagination)
export const useInfiniteEntities = (params?: Record<string, unknown>) => {
  return useInfiniteQuery({
    queryKey: entityKeys.list(params),
    queryFn: ({ pageParam = 1 }) => fetchEntities({ ...params, page: pageParam }),
    getNextPageParam: (lastPage) => lastPage.next ?? undefined,
    initialPageParam: 1,
  });
};
```

### mutations.ts Template
```typescript
import { useMutation, useQueryClient, type UseMutationOptions } from "@tanstack/react-query";
import { createEntity, updateEntity, deleteEntity } from "./services";
import { entityKeys } from "./queries";
import type { IEntity, ICreateEntityDTO, IUpdateEntityDTO } from "./types";

type MutationOptions<TData, TVariables> = Omit<
  UseMutationOptions<TData, Error, TVariables>,
  "mutationFn"
>;

export const useCreateEntity = (options?: MutationOptions<IEntity, ICreateEntityDTO>) => {
  const queryClient = useQueryClient();

  return useMutation({
    mutationFn: createEntity,
    onSuccess: (data, variables, context) => {
      queryClient.invalidateQueries({ queryKey: entityKeys.lists() });
      options?.onSuccess?.(data, variables, context);
    },
    ...options,
  });
};

export const useUpdateEntity = (
  options?: MutationOptions<IEntity, { id: number | string; data: IUpdateEntityDTO }>
) => {
  const queryClient = useQueryClient();

  return useMutation({
    mutationFn: updateEntity,
    onSuccess: (data, variables, context) => {
      queryClient.invalidateQueries({ queryKey: entityKeys.lists() });
      queryClient.invalidateQueries({ queryKey: entityKeys.detail(variables.id) });
      options?.onSuccess?.(data, variables, context);
    },
    ...options,
  });
};

export const useDeleteEntity = (options?: MutationOptions<void, number | string>) => {
  const queryClient = useQueryClient();

  return useMutation({
    mutationFn: deleteEntity,
    onSuccess: (data, variables, context) => {
      queryClient.invalidateQueries({ queryKey: entityKeys.lists() });
      options?.onSuccess?.(data, variables, context);
    },
    ...options,
  });
};
```

### types.ts Template
```typescript
// API Response Types
export interface IApiResponse<T> {
  result: T;
  message: string | null;
  resultCode: "SUCCESS" | "FAIL";
}

export interface IPaginatedResponse<T> {
  results: T;
  count: number;
  next: number | null;
  previous: number | null;
}

// Entity Interface
export interface IEntity {
  id: number;
  // Add entity-specific fields
  createdAt: string;
  updatedAt: string;
}

// DTOs
export interface ICreateEntityDTO {
  // Fields for creating entity
}

export interface IUpdateEntityDTO {
  // Fields for updating entity (usually Partial<ICreateEntityDTO>)
}

// Re-export for convenience
export type IEntityResponse<T> = IApiResponse<T>;
export type IEntityListResponse = IApiResponse<IPaginatedResponse<IEntity[]>>;
```

### index.ts (Barrel Export)
```typescript
// API
export * from "./services";
export * from "./queries";
export * from "./mutations";
export * from "./types";
```

## Component Layer Pattern

### Entity UI Structure
```
entities/[EntityName]/ui/
├── EntityCard/
│   ├── EntityCard.tsx
│   ├── EntityCard.module.css   # or .scss, styled-components
│   └── index.ts
├── EntityList/
│   ├── EntityList.tsx
│   └── index.ts
├── EntityForm/
│   ├── EntityForm.tsx
│   └── index.ts
└── index.ts
```

### Component Template
```typescript
// entities/[EntityName]/ui/EntityCard/EntityCard.tsx
import type { FC } from "react";
import type { IEntity } from "../../api";
import styles from "./EntityCard.module.css";

interface EntityCardProps {
  entity: IEntity;
  onEdit?: (entity: IEntity) => void;
  onDelete?: (id: number) => void;
}

export const EntityCard: FC<EntityCardProps> = ({ entity, onEdit, onDelete }) => {
  return (
    <div className={styles.card}>
      <h3>{entity.name}</h3>
      <div className={styles.actions}>
        {onEdit && <button onClick={() => onEdit(entity)}>Edit</button>}
        {onDelete && <button onClick={() => onDelete(entity.id)}>Delete</button>}
      </div>
    </div>
  );
};
```

### List Component with Query
```typescript
// entities/[EntityName]/ui/EntityList/EntityList.tsx
import type { FC } from "react";
import { useEntities } from "../../api";
import { EntityCard } from "../EntityCard";
import styles from "./EntityList.module.css";

interface EntityListProps {
  filters?: Record<string, unknown>;
  onEdit?: (entity: IEntity) => void;
  onDelete?: (id: number) => void;
}

export const EntityList: FC<EntityListProps> = ({ filters, onEdit, onDelete }) => {
  const { data, isLoading, error } = useEntities(filters);

  if (isLoading) return <div>Loading...</div>;
  if (error) return <div>Error: {error.message}</div>;

  return (
    <div className={styles.list}>
      {data?.results.map((entity) => (
        <EntityCard
          key={entity.id}
          entity={entity}
          onEdit={onEdit}
          onDelete={onDelete}
        />
      ))}
    </div>
  );
};
```

## Feature Layer Pattern

Features combine entities with user interactions:

```
features/[feature-name]/
├── api/              # Feature-specific API (if needed)
├── ui/
│   ├── FeatureForm/
│   └── FeatureWidget/
├── model/
│   ├── hooks.ts      # Feature-specific hooks
│   └── store.ts      # Feature state (if needed)
├── lib/
│   └── utils.ts
└── index.ts
```

### Feature Hook Example
```typescript
// features/manage-entity/model/hooks.ts
import { useCallback } from "react";
import { useCreateEntity, useUpdateEntity, useDeleteEntity } from "@/entities/entity/api";
import { toast } from "sonner";

export const useManageEntity = () => {
  const createMutation = useCreateEntity({
    onSuccess: () => toast.success("Created successfully"),
    onError: (error) => toast.error(error.message),
  });

  const updateMutation = useUpdateEntity({
    onSuccess: () => toast.success("Updated successfully"),
    onError: (error) => toast.error(error.message),
  });

  const deleteMutation = useDeleteEntity({
    onSuccess: () => toast.success("Deleted successfully"),
    onError: (error) => toast.error(error.message),
  });

  const handleCreate = useCallback((data: ICreateEntityDTO) => {
    createMutation.mutate(data);
  }, [createMutation]);

  const handleUpdate = useCallback((id: number, data: IUpdateEntityDTO) => {
    updateMutation.mutate({ id, data });
  }, [updateMutation]);

  const handleDelete = useCallback((id: number) => {
    deleteMutation.mutate(id);
  }, [deleteMutation]);

  return {
    handleCreate,
    handleUpdate,
    handleDelete,
    isCreating: createMutation.isPending,
    isUpdating: updateMutation.isPending,
    isDeleting: deleteMutation.isPending,
  };
};
```

## Shared Layer Pattern

### API Client Setup
```typescript
// shared/api/client.ts
import axios from "axios";

export const api = axios.create({
  baseURL: process.env.NEXT_PUBLIC_API_URL || "/api",
  timeout: 30000,
  headers: {
    "Content-Type": "application/json",
  },
});

// Request interceptor
api.interceptors.request.use((config) => {
  const token = localStorage.getItem("token");
  if (token) {
    config.headers.Authorization = `Bearer ${token}`;
  }
  return config;
});

// Response interceptor
api.interceptors.response.use(
  (response) => response,
  async (error) => {
    if (error.response?.status === 401) {
      // Handle token refresh or redirect to login
    }
    return Promise.reject(error);
  }
);
```

### Query Client Provider (Next.js 15 App Router)
```typescript
// app/providers/QueryProvider.tsx
"use client";

import { QueryClient, QueryClientProvider } from "@tanstack/react-query";
import { ReactQueryDevtools } from "@tanstack/react-query-devtools";
import { useState, type ReactNode } from "react";

export function QueryProvider({ children }: { children: ReactNode }) {
  const [queryClient] = useState(
    () =>
      new QueryClient({
        defaultOptions: {
          queries: {
            staleTime: 60 * 1000,
            retry: 1,
            refetchOnWindowFocus: false,
          },
        },
      })
  );

  return (
    <QueryClientProvider client={queryClient}>
      {children}
      <ReactQueryDevtools initialIsOpen={false} />
    </QueryClientProvider>
  );
}
```

### Next.js 15 Layout Integration
```typescript
// app/layout.tsx
import { QueryProvider } from "./providers/QueryProvider";

export default function RootLayout({ children }: { children: React.ReactNode }) {
  return (
    <html lang="en">
      <body>
        <QueryProvider>{children}</QueryProvider>
      </body>
    </html>
  );
}
```

## FSD Import Rules

1. **Layers can only import from layers below them**:
   - `app` → `pages`, `widgets`, `features`, `entities`, `shared`
   - `pages` → `widgets`, `features`, `entities`, `shared`
   - `widgets` → `features`, `entities`, `shared`
   - `features` → `entities`, `shared`
   - `entities` → `shared`
   - `shared` → nothing (only external deps)

2. **Cross-imports within a layer are forbidden**
   - `entities/user` cannot import from `entities/product`
   - If needed, lift shared logic to `shared` layer

3. **Use public API (index.ts) for all imports**
   ```typescript
   // Good
   import { useUsers, UserCard } from "@/entities/user";

   // Bad
   import { useUsers } from "@/entities/user/api/queries";
   import { UserCard } from "@/entities/user/ui/UserCard/UserCard";
   ```

## Path Aliases

Configure in `tsconfig.json`:
```json
{
  "compilerOptions": {
    "baseUrl": ".",
    "paths": {
      "@/*": ["./src/*"],
      "@/entities/*": ["./src/entities/*"],
      "@/features/*": ["./src/features/*"],
      "@/widgets/*": ["./src/widgets/*"],
      "@/shared/*": ["./src/shared/*"]
    }
  }
}
```

## Naming Conventions

| Type | Pattern | Example |
|------|---------|---------|
| Entity folder | PascalCase | `User`, `Product`, `Order` |
| Service functions | camelCase with verb | `fetchUsers`, `createUser` |
| Query hooks | `use` + Entity + action | `useUsers`, `useUser`, `useInfiniteUsers` |
| Mutation hooks | `use` + action + Entity | `useCreateUser`, `useUpdateUser` |
| Query keys | entityKeys object | `userKeys.all`, `userKeys.detail(id)` |
| Components | PascalCase | `UserCard`, `UserList`, `UserForm` |
| Types | `I` prefix for interfaces | `IUser`, `ICreateUserDTO` |

## Supported Technologies

Works with any combination of:

- **Frameworks**: React, Next.js 13/14/15 (App Router or Pages Router)
- **Styling**: Tailwind CSS, CSS Modules, styled-components, any UI library (shadcn/ui, MUI, Ant Design, etc.)
- **State**: React Query (TanStack Query v5)
- **Forms**: React Hook Form (optional)
- **HTTP**: Axios

## Quick Reference

### Generate Entity Command
When asked to create a new entity, generate these files:
1. `entities/[Name]/api/types.ts` - Define interfaces
2. `entities/[Name]/api/services.ts` - API functions
3. `entities/[Name]/api/queries.ts` - Query hooks
4. `entities/[Name]/api/mutations.ts` - Mutation hooks
5. `entities/[Name]/api/index.ts` - Barrel export
6. `entities/[Name]/ui/` - UI components
7. `entities/[Name]/index.ts` - Public API

See [templates/](./templates/) for complete file templates.
