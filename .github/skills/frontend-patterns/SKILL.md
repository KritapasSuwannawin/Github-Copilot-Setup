---
name: frontend-patterns
description: React and TypeScript patterns and conventions for frontend-developer. The agreed patterns to follow consistently across the codebase.
---

# Skill: Frontend Patterns

## Purpose

React + TypeScript patterns and conventions for the `frontend-developer`. These are the agreed patterns to follow consistently across the codebase.

Frontend architecture in this repo follows `Feature-Sliced Design`. Read `.github/skills/feature-sliced-design/SKILL.md` before deciding where code belongs or how slices may import each other.

---

## Component Design

### Component Types

| Type                | Purpose                               | Has State?      | Makes API Calls?             |
| ------------------- | ------------------------------------- | --------------- | ---------------------------- |
| Page slice          | Route-level composition               | Yes (via hooks) | Via lower layers             |
| Widget              | Reusable page section                 | Yes (via hooks) | Via features or entities     |
| Feature UI          | User-facing action or workflow        | Yes (via hooks) | Via slice `model/` or `api/` |
| Entity UI           | Entity-focused reusable block         | Yes (if needed) | Via slice `model/` or `api/` |
| Shared UI component | Generic presentational building block | No              | Never                        |

### Component Rules

- UI components must be **pure** — same props → same output
- Never fetch data inside a component body — use custom hooks
- Keep JSX concise — extract sub-components if JSX exceeds ~50 lines
- Props must be fully typed — no implicit `any`, no prop spreading without explicit types
- Place components in the owning FSD slice. Promote code to `shared/ui` only when it is truly generic and business-agnostic.

### Naming

- Components: `PascalCase` file and export: `UserCard.tsx`
- Hooks: `use` prefix: `useUserProfile.ts`
- Utilities: `camelCase`: `formatCurrency.ts`
- Constants: `SCREAMING_SNAKE_CASE` in `constants.ts`

---

## Props Patterns

```typescript
// [GOOD] Explicit props interface
interface UserCardProps {
  userId: string;
  onSelect?: (userId: string) => void;
  className?: string;
}

export function UserCard({ userId, onSelect, className }: UserCardProps) { ... }
```

- Use `children: React.ReactNode` for slot-style composition
- Use callback naming: `onAction` for event handlers passed as props
- Mark optional props with `?` and provide sensible defaults

---

## Custom Hooks

Custom hooks are the primary place for:

- Data fetching and caching
- Form state and validation
- Business logic that drives UI
- Side effects with cleanup

```typescript
// [GOOD] Hook encapsulates fetching logic
export function useUserProfile(userId: string) {
  const [profile, setProfile] = useState<UserProfile | null>(null);
  const [isLoading, setIsLoading] = useState(true);
  const [error, setError] = useState<Error | null>(null);

  useEffect(() => {
    userService
      .getProfile(userId)
      .then(setProfile)
      .catch(setError)
      .finally(() => setIsLoading(false));
  }, [userId]);

  return { profile, isLoading, error };
}
```

---

## State Management

### Local State

- Use `useState` for component-local state
- Use `useReducer` for complex local state with multiple sub-values

### Global State — Redux + Redux Toolkit

**This project uses Redux + Redux Toolkit for global state.** Do not introduce alternative state libraries.

- Organise state into **slices** (`createSlice`) — one slice per domain area (e.g. `authSlice`, `cartSlice`)
- Use **selectors** to read state from components (`useSelector`) — never access `store.getState()` directly in components
- Use **thunks** (`createAsyncThunk`) for async operations that need loading/error tracking in global state
- Use **Redux Toolkit Query (RTK Query)** for server state when caching, refetching, or optimistic updates are needed; otherwise use `useEffect + fetch` in a custom hook

```typescript
// slice example
export const authSlice = createSlice({
  name: "auth",
  initialState: { user: null as User | null, isLoading: false },
  reducers: {
    setUser: (state, action: PayloadAction<User>) => {
      state.user = action.payload;
    },
    clearUser: (state) => {
      state.user = null;
    },
  },
});
```

### Data Fetching

**This project uses native `useEffect + fetch`** inside custom hooks. Do not introduce React Query, SWR, or Axios unless `project-manager` approves. Keep fetch logic inside the owning slice's `api/` module or `shared/api`, not directly in UI components.

### Forms

**This project uses native controlled components** for forms. Do not introduce `react-hook-form` or `Formik`.

### State Rules

- Keep state as local as possible — lift to Redux only when multiple unrelated components need it
- Derived data should be computed in selectors, not stored: use `createSelector` for memoised derivations
- Never duplicate state — single source of truth

---

## Error & Loading States

Every data-dependent component must handle all three states:

```tsx
function UserProfile({ userId }: { userId: string }) {
  const { profile, isLoading, error } = useUserProfile(userId);

  if (isLoading) return <Skeleton />;
  if (error) return <ErrorMessage error={error} />;
  if (!profile) return <EmptyState message="Profile not found" />;

  return <ProfileCard profile={profile} />;
}
```

---

## Performance Patterns

- Use `React.memo` on pure UI components that render frequently with the same props
- Use `useCallback` on functions passed as props to memoised children
- Use `useMemo` for expensive computations — not for every derived value
- Lazy-load route-level components with `React.lazy` + `Suspense`
- Avoid anonymous functions and objects in JSX (they break referential equality)

---

## Folder Structure

```
src/
├── app/                ← App bootstrap, routing, providers, store wiring
│   ├── providers/
│   ├── router/
│   └── store/
├── pages/              ← Route-level slices
│   └── {page}/
│       ├── ui/
│       └── index.ts
├── widgets/            ← Composed page sections
│   └── {widget}/
│       ├── ui/
│       └── index.ts
├── features/           ← User actions and flows
│   └── {feature}/
│       ├── ui/
│       ├── model/
│       ├── api/
│       └── index.ts
├── entities/           ← Business entities and related code
│   └── {entity}/
│       ├── ui/
│       ├── model/
│       ├── api/
│       └── index.ts
└── shared/             ← Generic UI, API utilities, config, and libs
    ├── api/
    ├── config/
    ├── lib/
    └── ui/
```

---

## Anti-Patterns to Avoid

- Fetching data directly in component bodies
- Business logic in JSX
- Deep-importing another slice's internals or creating sibling-slice dependencies
- Dumping business-specific code into `shared/`
- Prop drilling deeper than 2 levels — use Context or composition
- Using array index as React `key` in dynamic lists
- Mutating state directly
- Multiple `useEffect` doing related things — consolidate into a custom hook
