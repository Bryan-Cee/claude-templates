# Project CLAUDE.md — React / Next.js
> Extends global ~/.claude/CLAUDE.md — only project-specific conventions live here.
> Fill in the [PLACEHOLDER] sections when starting a new project.

---

## Project Overview
- **Name:** [PLACEHOLDER]
- **Purpose:** [PLACEHOLDER — one sentence describing what this app does]
- **API / Backend:** [PLACEHOLDER — REST, GraphQL, external service, etc.]

---

## Stack
- Next.js App Router — no Pages Router patterns
- TypeScript strict mode
- Tailwind CSS for styling
- React Query for server state
- Zustand for client state
- React Hook Form + Zod for forms and validation
- Vitest + React Testing Library for unit/integration tests
- Playwright for E2E tests
- ESLint + Prettier enforced via Husky pre-commit hooks

---

## Project Structure
```
src/
├── app/                  # Next.js App Router — routes and layouts only
│   ├── (auth)/           # Route groups for layout scoping
│   ├── layout.tsx        # Root layout
│   └── page.tsx          # Root page
├── components/           # Reusable, stateless, no direct API calls
├── layouts/              # Page-level layout components
├── features/             # [optional] feature-scoped logic if app is large
├── hooks/                # Custom React hooks
├── services/             # API service files — all fetch logic lives here
├── stores/               # Zustand store definitions
├── types/                # Shared TypeScript types and interfaces
├── lib/                  # Third-party client setup (React Query, axios, etc.)
├── utils/                # Pure utility functions
└── schemas/              # Zod validation schemas
```

---

## Routing — Next.js App Router
- All routes live under `src/app/` — never mix with `pages/`
- Route files are `page.tsx`, layouts are `layout.tsx`, loading states are `loading.tsx`
- Use route groups `(groupName)/` to scope layouts without affecting the URL
- Server Components by default — add `"use client"` only when necessary (event handlers, hooks, browser APIs)
- Never fetch data in a Client Component — fetch in Server Components and pass down as props
- Dynamic routes use `[param]` folders — always validate params with Zod before use

---

## Component Conventions
- One component per file — filename matches the component name in PascalCase
- Every component folder contains:
  ```
  Button/
  ├── Button.tsx        # Component
  ├── Button.test.tsx   # Co-located tests
  ├── Button.module.css # Only if Tailwind isn't sufficient (rare)
  └── index.ts          # Re-export for clean imports
  ```
- Props interfaces are defined in the same file, named `[ComponentName]Props`
- `src/components/` — purely presentational, no business logic, no direct API calls
- `src/layouts/` — page-level structure components, no business logic
- If a component needs data, it receives it via props from a Server Component or a React Query hook

---

## Styling — Tailwind Only
- Tailwind utility classes only — no inline styles, no CSS Modules unless Tailwind genuinely can't do it
- Design tokens (colors, spacing, fonts) live in `tailwind.config.ts` — never hardcode raw values
- Always provide `dark:` variants for any color utility
- Mobile-first — base styles are mobile, `md:` and `lg:` scale up
- Use `cn()` utility (clsx + tailwind-merge) for conditional class merging — never template literals
- No `@apply` in CSS files — if you're reaching for it, just use a component instead

---

## State Management

### Ownership Rules
- **Server state** (anything from an API) → React Query exclusively
- **Local UI state** (toggle, input focus, hover) → `useState` in the component
- **Shared UI state** (modals, sidebar, selected IDs, theme, wizard progress) → Zustand
- Never store derived state — compute it from source state on render
- The boundary test: if the page refreshes and this state should reset → Zustand. If it should rehydrate from the server → React Query

### The Golden Pattern — Zustand drives, React Query fetches
Zustand stores hold selection and UI state. React Query hooks consume that state to fetch. Never the other way around.

```ts
// ✅ Correct — Zustand drives the selection, React Query handles the fetch
const selectedUserId = useUIStore((s) => s.selectedUserId)
const { data: user } = useGetUser(selectedUserId)

// ❌ Wrong — copying server state into Zustand creates two sources of truth
const { data } = useGetUser(userId)
useEffect(() => { if (data) setUserStore(data) }, [data])
```

### Hard Rules
- Never copy React Query data into a Zustand store — creates two sources of truth that will drift
- Never use Zustand as a loading/error state relay — read `isLoading` and `isError` directly from the React Query hook
- Zustand stores live in `src/stores/` — one file per domain (e.g. `useUIStore.ts`, `useAuthStore.ts`)
- Stores export a typed hook — never import the raw store directly in components

---

## API & Data Fetching
- All fetch logic lives in `src/services/` — components and hooks never call `fetch` directly
- Service files are plain async functions that return typed data or throw typed errors
- React Query hooks live in `src/hooks/` and wrap service functions — e.g. `useGetUser.ts`
- Always define `queryKey` factories alongside the hook to avoid key collisions
- API base URL comes from `process.env.NEXT_PUBLIC_API_URL` — never hardcode
- All service functions validate response shape with Zod before returning data
- Mutations use `useMutation` — always handle `onError` and invalidate relevant queries `onSuccess`

---

## Forms — React Hook Form + Zod
- Every form has a corresponding Zod schema in `src/schemas/`
- Use `zodResolver` to connect the schema to React Hook Form
- Never manage form state manually with `useState` — always use RHF
- Display field-level errors from RHF's `formState.errors` — no custom error state
- Zod schemas are the single source of truth for types — infer TypeScript types from them:
  ```ts
  const loginSchema = z.object({ ... })
  type LoginInput = z.infer<typeof loginSchema>
  ```
- Schemas in `src/schemas/` can be reused across forms and API validation

---

## Path Aliases
- Always use path aliases — never relative imports that go up more than one level
- Configured in `tsconfig.json` and `vite.config.ts`:
  ```ts
  "@/components/*" → "src/components/*"
  "@/hooks/*"      → "src/hooks/*"
  "@/services/*"   → "src/services/*"
  "@/stores/*"     → "src/stores/*"
  "@/types/*"      → "src/types/*"
  "@/lib/*"        → "src/lib/*"
  "@/utils/*"      → "src/utils/*"
  "@/schemas/*"    → "src/schemas/*"
  ```

---

## Testing
- **Unit/integration** → Vitest + React Testing Library
- **E2E** → Playwright (tests live in `e2e/` at the repo root)
- Test files co-locate with source: `Button.tsx` → `Button.test.tsx`
- Test behaviour, not implementation — never assert on internal state or implementation details
- Mock at the service layer — never mock React Query or Zustand internals
- Every form submission, API error state, and loading state needs a test
- E2E tests cover critical user journeys only — not unit-level scenarios

---

## Code Quality & Linting
- ESLint + Prettier run automatically on pre-commit via Husky — never commit code that fails either
- If a lint rule needs to be disabled, add a comment explaining why — no silent suppressions
- Prettier config is the source of truth for formatting — don't manually reformat

---

## Environment Variables
- All env vars documented in `.env.example` — keep it up to date whenever a new var is added
- Client-side vars must be prefixed with `NEXT_PUBLIC_`
- Never access `process.env` directly in components — wrap in a typed config object in `src/lib/config.ts`

---

## Placeholder Checklist — Complete Before First Session
- [ ] Fill in Project Overview section above
- [ ] Update `src/` structure if this project omits any folders
- [ ] Add any third-party services (auth provider, payment, analytics) to the Stack section
- [ ] Add API-specific conventions if backend is known
- [ ] Confirm path aliases match actual `tsconfig.json` config
