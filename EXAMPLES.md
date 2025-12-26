# FSD Frontend Skill - Usage Examples

## Quick Start

### Example 1: Create a "Product" Entity

**Request:**
> Create a Product entity with name, price, category, and image fields

**Generated Structure:**
```
src/entities/Product/
├── api/
│   ├── index.ts
│   ├── services.ts
│   ├── queries.ts
│   ├── mutations.ts
│   └── types.ts
├── ui/
│   ├── ProductCard/
│   ├── ProductList/
│   ├── ProductForm/
│   └── index.ts
└── index.ts
```

**types.ts:**
```typescript
export interface IProduct {
  id: number;
  name: string;
  price: number;
  category: string;
  image: string;
  isActive: boolean;
  createdAt: string;
  updatedAt: string;
}

export interface ICreateProductDTO {
  name: string;
  price: number;
  category: string;
  image?: File;
}

export interface IUpdateProductDTO {
  name?: string;
  price?: number;
  category?: string;
  image?: File;
}

export interface IProductFilters {
  search?: string;
  category?: string;
  minPrice?: number;
  maxPrice?: number;
  page?: number;
}
```

---

### Example 2: Create a "User" Feature with Authentication

**Request:**
> Create a feature for user profile management with edit capabilities

**Generated Structure:**
```
src/features/user-profile/
├── api/
│   ├── services.ts      # updateProfile, changePassword, uploadAvatar
│   ├── mutations.ts     # useUpdateProfile, useChangePassword
│   └── types.ts
├── ui/
│   ├── ProfileForm/
│   ├── PasswordForm/
│   └── AvatarUpload/
├── model/
│   └── hooks.ts         # useProfileManager
└── index.ts
```

**model/hooks.ts:**
```typescript
import { useCallback } from "react";
import { useUpdateProfile, useChangePassword } from "../api";
import { toast } from "sonner";

export const useProfileManager = () => {
  const updateMutation = useUpdateProfile({
    onSuccess: () => toast.success("Profile updated"),
    onError: (error) => toast.error(error.message),
  });

  const passwordMutation = useChangePassword({
    onSuccess: () => toast.success("Password changed"),
    onError: (error) => toast.error(error.message),
  });

  return {
    updateProfile: updateMutation.mutate,
    changePassword: passwordMutation.mutate,
    isUpdating: updateMutation.isPending,
    isChangingPassword: passwordMutation.isPending,
  };
};
```

---

### Example 3: Widget with Multiple Entities

**Request:**
> Create an OrderSummary widget that shows order details with products

**Generated Structure:**
```
src/widgets/OrderSummary/
├── ui/
│   ├── OrderSummary.tsx
│   ├── OrderItems.tsx
│   ├── OrderTotal.tsx
│   └── index.ts
├── model/
│   └── hooks.ts
└── index.ts
```

**ui/OrderSummary.tsx:**
```typescript
"use client";

import type { FC } from "react";
import { useOrder } from "@/entities/Order";
import { ProductCard } from "@/entities/Product";
import { OrderItems } from "./OrderItems";
import { OrderTotal } from "./OrderTotal";
import styles from "./OrderSummary.module.css";

interface OrderSummaryProps {
  orderId: number;
}

export const OrderSummary: FC<OrderSummaryProps> = ({ orderId }) => {
  const { data: order, isLoading, error } = useOrder(orderId);

  if (isLoading) return <OrderSummarySkeleton />;
  if (error) return <ErrorState message={error.message} />;
  if (!order) return null;

  return (
    <div className={styles.container}>
      <h2>Order #{order.publicId}</h2>
      <OrderItems items={order.items} />
      <OrderTotal
        subtotal={order.subtotal}
        tax={order.tax}
        total={order.total}
      />
    </div>
  );
};
```

---

### Example 4: Page with Filters and Pagination

**Request:**
> Create a Products page with search, category filter, and infinite scroll

**Generated Structure:**
```
src/pages/products/          # or app/products/ for Next.js 15
├── page.tsx
├── components/
│   ├── ProductFilters.tsx
│   └── ProductGrid.tsx
└── hooks/
    └── useProductFilters.ts
```

**page.tsx (Next.js 15 App Router):**
```typescript
"use client";

import { useState } from "react";
import { useInfiniteProducts } from "@/entities/Product";
import { ProductCard } from "@/entities/Product";
import { ProductFilters } from "./components/ProductFilters";
import { useInView } from "react-intersection-observer";
import { useEffect } from "react";

export default function ProductsPage() {
  const [filters, setFilters] = useState<IProductFilters>({});

  const {
    data,
    fetchNextPage,
    hasNextPage,
    isFetchingNextPage,
    isLoading,
  } = useInfiniteProducts(filters);

  // Infinite scroll trigger
  const { ref, inView } = useInView();

  useEffect(() => {
    if (inView && hasNextPage && !isFetchingNextPage) {
      fetchNextPage();
    }
  }, [inView, hasNextPage, isFetchingNextPage, fetchNextPage]);

  return (
    <div className="container">
      <h1>Products</h1>

      <ProductFilters
        filters={filters}
        onChange={setFilters}
      />

      <div className="grid">
        {data?.items.map((product) => (
          <ProductCard key={product.id} product={product} />
        ))}
      </div>

      {/* Infinite scroll sentinel */}
      <div ref={ref}>
        {isFetchingNextPage && <Spinner />}
      </div>
    </div>
  );
}
```

---

### Example 5: Form with File Upload

**Request:**
> Create a form for creating products with image upload

**ProductForm with FormData:**
```typescript
"use client";

import { useForm } from "react-hook-form";
import { useCreateProductWithFormData } from "@/entities/Product";
import type { ICreateProductDTO } from "@/entities/Product";

interface FormValues {
  name: string;
  price: number;
  category: string;
  image: FileList;
}

export const CreateProductForm = () => {
  const { register, handleSubmit, formState: { errors } } = useForm<FormValues>();
  const createMutation = useCreateProductWithFormData();

  const onSubmit = (data: FormValues) => {
    const formData = new FormData();
    formData.append("name", data.name);
    formData.append("price", String(data.price));
    formData.append("category", data.category);
    if (data.image?.[0]) {
      formData.append("image", data.image[0]);
    }
    createMutation.mutate(formData);
  };

  return (
    <form onSubmit={handleSubmit(onSubmit)}>
      <input {...register("name", { required: true })} placeholder="Name" />
      <input {...register("price", { required: true, min: 0 })} type="number" placeholder="Price" />
      <input {...register("category", { required: true })} placeholder="Category" />
      <input {...register("image")} type="file" accept="image/*" />
      <button type="submit" disabled={createMutation.isPending}>
        {createMutation.isPending ? "Creating..." : "Create Product"}
      </button>
    </form>
  );
};
```

---

## Common Patterns

### Pattern 1: Conditional Query

```typescript
// Only fetch when ID exists
const { data } = useProduct(productId, {
  enabled: Boolean(productId),
});

// Only fetch when user is authenticated
const { data } = useUserOrders({
  enabled: isAuthenticated,
});
```

### Pattern 2: Dependent Queries

```typescript
// Fetch user first, then their orders
const { data: user } = useCurrentUser();
const { data: orders } = useUserOrders(user?.id, {
  enabled: Boolean(user?.id),
});
```

### Pattern 3: Prefetching on Hover

```typescript
import { useQueryClient } from "@tanstack/react-query";
import { prefetchProduct } from "@/entities/Product";

const ProductLink = ({ productId }) => {
  const queryClient = useQueryClient();

  return (
    <Link
      href={`/products/${productId}`}
      onMouseEnter={() => prefetchProduct(queryClient, productId)}
    >
      View Product
    </Link>
  );
};
```

### Pattern 4: Cache Invalidation Across Entities

```typescript
// In features/checkout/api/mutations.ts
export const useCompleteCheckout = () => {
  const queryClient = useQueryClient();

  return useMutation({
    mutationFn: completeCheckout,
    onSuccess: () => {
      // Invalidate multiple entities
      queryClient.invalidateQueries({ queryKey: orderKeys.all });
      queryClient.invalidateQueries({ queryKey: cartKeys.all });
      queryClient.invalidateQueries({ queryKey: productKeys.lists() }); // Update stock
    },
  });
};
```

### Pattern 5: Error Handling with Toast

```typescript
import { toast } from "sonner"; // or react-toastify, etc.

export const useCreateProduct = () => {
  return useMutation({
    mutationFn: createProduct,
    onSuccess: () => {
      toast.success("Product created successfully");
    },
    onError: (error) => {
      if (error.response?.status === 422) {
        toast.error("Validation error: " + extractErrors(error));
      } else {
        toast.error("Failed to create product");
      }
    },
  });
};
```

---

## Project Setup Checklist

### 1. Install Dependencies

```bash
# Core dependencies
npm install @tanstack/react-query axios react-hook-form

# Dev dependencies
npm install -D @tanstack/react-query-devtools

# Optional utilities
npm install sonner               # Toast notifications
npm install react-intersection-observer  # Infinite scroll
npm install zod                  # Schema validation
```

### 2. Configure Path Aliases

**tsconfig.json:**
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

### 3. Setup Providers

**app/layout.tsx (Next.js 15):**
```typescript
import { QueryProvider } from "@/shared/lib/query-provider";

export default function RootLayout({ children }) {
  return (
    <html>
      <body>
        <QueryProvider>
          {children}
        </QueryProvider>
      </body>
    </html>
  );
}
```

### 4. Create Shared API Client

Copy `templates/shared/api-client.ts.template` to `src/shared/api/client.ts`

### 5. Generate Your First Entity

Ask Claude:
> Create a [EntityName] entity with [field1], [field2], [field3] fields
