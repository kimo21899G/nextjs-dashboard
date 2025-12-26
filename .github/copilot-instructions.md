# Next.js Dashboard AI Coding Guidelines

## Architecture Overview

**Stack**: Next.js 15+ (App Router), React, TypeScript, PostgreSQL, TailwindCSS, NextAuth  
**Key Pattern**: Server-side data fetching with PostgreSQL, client-side UI components with Tailwind styling  

This is a learning dashboard application with three main features:
- **Dashboard**: Analytics overview with revenue charts and latest invoices
- **Invoices**: CRUD operations with pagination and status management
- **Customers**: Customer directory with search functionality

### Data Flow
1. Data lives in PostgreSQL (accessed via `postgres` library)
2. Fetch functions in [app/lib/data.ts](app/lib/data.ts) are server-only utilities
3. Components in [app/ui/](app/ui/) render data, using `clsx` for conditional styling
4. Type definitions in [app/lib/definitions.ts](app/lib/definitions.ts) define all data shapes (User, Customer, Invoice, Revenue, etc.)

## Key Folders & Their Purpose

- **[app/lib/](app/lib/)**: Data fetching (`data.ts`), TypeScript types (`definitions.ts`), utilities (`utils.ts`)
- **[app/ui/](app/ui/)**: Reusable components; organized by feature (dashboard/, invoices/, customers/)
- **[app/dashboard/](app/dashboard/)**: Dashboard layout with SideNav; nested pages for invoices and customers
- **[app/query/](app/query/) & [app/seed/](app/seed/)**: API routes for custom queries and database seeding

## Development Workflow

**Start dev server**: `pnpm dev --turbopack` (Turbopack enables fast builds)  
**Build for production**: `pnpm build`  
**Start prod server**: `pnpm start`

Database is seeded via [app/seed/route.ts](app/seed/route.ts) — access `/seed` to initialize test data.

## Project-Specific Patterns

### Type-Driven Development
Types are explicitly defined, not auto-generated. Example from [app/lib/definitions.ts](app/lib/definitions.ts):
```typescript
export type Invoice = {
  id: string;
  customer_id: string;
  amount: number;
  date: string;
  status: 'pending' | 'paid';  // string union type
};
```

### Data Transformation
Raw database data differs from display data. Example from [app/lib/data.ts](app/lib/data.ts):
```typescript
const data = await sql<LatestInvoiceRaw[]>`SELECT ...`;
const latestInvoices = data.map((invoice) => ({
  ...invoice,
  amount: formatCurrency(invoice.amount),  // format in app, not database
}));
```

### Styling Convention
Use `clsx` for conditional Tailwind classes. Example from [app/ui/button.tsx](app/ui/button.tsx):
```typescript
className={clsx(
  'flex h-10 items-center rounded-lg bg-blue-500 px-4 text-sm font-medium text-white transition-colors hover:bg-blue-400',
  className,  // allow override
)}
```

### Custom Color Theme
Tailwind config in [tailwind.config.ts](tailwind.config.ts) defines custom blues:
- `bg-blue-400: #2589FE`, `bg-blue-500: #0070F3`, `bg-blue-600: #2F6FEB`

Use these throughout components; don't use generic Tailwind blues.

### Font Management
Fonts from Google Fonts in [app/ui/fonts.ts](app/ui/fonts.ts):
- `inter`: body text (default)
- `lusitana`: headings (applied via `.className` property)

Example from [app/ui/acme-logo.tsx](app/ui/acme-logo.tsx):
```typescript
className={`${lusitana.className} flex flex-row items-center ...`}
```

### Utility Functions
Format functions in [app/lib/utils.ts](app/lib/utils.ts):
- `formatCurrency(amount: number)`: Converts cents to USD format
- `formatDateToLocal(dateStr: string, locale?: string)`: ISO date to readable format

Always use these; don't hardcode formatting.

## Database & Environment

PostgreSQL connection via `postgres` library using `process.env.POSTGRES_URL` (requires SSL).  
Error handling: catch database errors in data fetch functions and throw user-friendly errors.

Example pattern from [app/lib/data.ts](app/lib/data.ts):
```typescript
try {
  const data = await sql<Revenue[]>`SELECT * FROM revenue`;
  return data;
} catch (error) {
  console.error('Database Error:', error);
  throw new Error('Failed to fetch revenue data.');
}
```

## Path Aliases

TypeScript paths configured in [tsconfig.json](tsconfig.json):
```json
"@/*": ["./*"]
```

Always import with `@/app/...`, not relative paths. Example:
```typescript
import { lusitana } from "@/app/ui/fonts";
import { fetchRevenue } from "@/app/lib/data";
```

## When Adding Features

1. Define types in `app/lib/definitions.ts` first
2. Add data fetching functions to `app/lib/data.ts` with error handling
3. Create UI components in `app/ui/` (organized by feature)
4. Add page in `app/dashboard/[feature]/page.tsx`
5. Use custom Tailwind colors and fonts consistently
6. Apply `clsx` for conditional styling, never string concatenation
