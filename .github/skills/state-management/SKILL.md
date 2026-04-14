---
name: state-management
description: Redux + Redux Toolkit state management patterns for frontend-developer. Covers slices, selectors, async thunks, and how to decide what belongs in global state.
---

# Skill: State Management (Redux + Redux Toolkit)

## Purpose

This skill defines how global client state is managed in this project using Redux Toolkit (RTK). Follow these patterns consistently across all features.

Frontend architecture in this repo follows `Feature-Sliced Design`. Keep global store wiring in `src/app/store/`, and co-locate reducers, thunks, and selectors in the owning slice's `model/` segment when practical.

---

## What Goes in Redux State

Only put data in Redux if it meets one or more of these criteria:

- Shared across two or more unrelated components
- Needs to persist across route changes
- Drives UI decisions in multiple parts of the tree (e.g., auth state, theme, notifications)

**Do not** put form state, component-local UI state (open/closed, hover), or server-fetched data that is only used in one place into Redux. Use `useState` or a custom hook instead.

---

## Slice Pattern

One Redux slice per app-wide concern. Keep slice logic in the owning FSD slice such as `src/features/<feature>/model/` or `src/entities/<entity>/model/`. Only the shared store setup lives in `src/app/store/`.

```typescript
// src/features/auth/model/slice.ts
import { createSlice, PayloadAction } from "@reduxjs/toolkit";

interface AuthState {
  currentUser: User | null;
  status: "idle" | "loading" | "succeeded" | "failed";
  error: string | null;
}

const initialState: AuthState = {
  currentUser: null,
  status: "idle",
  error: null,
};

const authSlice = createSlice({
  name: "auth",
  initialState,
  reducers: {
    setCurrentUser(state, action: PayloadAction<User>) {
      state.currentUser = action.payload;
    },
    clearAuth(state) {
      state.currentUser = null;
      state.status = "idle";
      state.error = null;
    },
  },
  extraReducers: (builder) => {
    builder
      .addCase(loginThunk.pending, (state) => {
        state.status = "loading";
        state.error = null;
      })
      .addCase(loginThunk.fulfilled, (state, action) => {
        state.status = "succeeded";
        state.currentUser = action.payload;
      })
      .addCase(loginThunk.rejected, (state, action) => {
        state.status = "failed";
        state.error = action.error.message ?? "Login failed";
      });
  },
});

export const { setCurrentUser, clearAuth } = authSlice.actions;
export default authSlice.reducer;
```

---

## Async Thunks

Use `createAsyncThunk` for all side effects that interact with an API.

```typescript
// src/features/auth/model/thunks.ts
import { createAsyncThunk } from "@reduxjs/toolkit";
import { authService } from "../api/authService";

export const loginThunk = createAsyncThunk(
  "auth/login",
  async (credentials: LoginCredentials) => {
    return authService.login(credentials);
  },
);
```

---

## Selectors

Co-locate selectors with their slice. Name them `select<FeatureDescription>`.

```typescript
// src/features/auth/model/selectors.ts
import type { RootState } from "@/app/store/store";

export const selectCurrentUser = (state: RootState) => state.auth.currentUser;
export const selectAuthStatus = (state: RootState) => state.auth.status;
export const selectIsAuthenticated = (state: RootState) =>
  state.auth.currentUser !== null;
```

For derived/computed values, use `createSelector` from `reselect` (bundled with RTK):

```typescript
import { createSelector } from "@reduxjs/toolkit";

export const selectUserDisplayName = createSelector(
  selectCurrentUser,
  (user) => (user ? `${user.firstName} ${user.lastName}` : "Guest"),
);
```

---

## Store Setup

```typescript
// src/app/store/store.ts
import { configureStore } from "@reduxjs/toolkit";
import { authReducer } from "@/features/auth";

export const store = configureStore({
  reducer: {
    auth: authReducer,
    // add more slices here
  },
});

export type RootState = ReturnType<typeof store.getState>;
export type AppDispatch = typeof store.dispatch;
```

Typed hooks — create once and reuse everywhere:

```typescript
// src/app/store/hooks.ts
import { useDispatch, useSelector } from "react-redux";
import type { RootState, AppDispatch } from "./store";

export const useAppDispatch = () => useDispatch<AppDispatch>();
export const useAppSelector = <T>(selector: (state: RootState) => T) =>
  useSelector(selector);
```

---

## Folder Structure

```
src/
├── app/
│   └── store/
│       ├── store.ts          ← configureStore + RootState + AppDispatch
│       └── hooks.ts          ← useAppDispatch, useAppSelector
├── features/
│   └── auth/
│       ├── api/
│       ├── model/
│       │   ├── slice.ts
│       │   ├── thunks.ts
│       │   └── selectors.ts
│       └── index.ts
└── entities/
    └── cart/
        ├── model/
        │   ├── slice.ts
        │   └── selectors.ts
        └── index.ts
```

---

## Rules

- Never dispatch actions directly from components — use thunks or action creators
- Never access `store.getState()` outside of tests — use `useAppSelector` in components
- Keep reducers pure — no API calls, no side effects
- Keep slices focused — one domain per slice, not one slice for the entire app
- Register reducers in `src/app/store/store.ts`, but keep business-specific Redux logic in the owning FSD slice
- Export reducers, actions, and selectors through the slice's public API instead of deep imports
