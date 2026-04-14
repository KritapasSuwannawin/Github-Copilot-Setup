---
name: state-management
description: Redux + Redux Toolkit state management patterns for frontend-developer. Covers slices, selectors, async thunks, and how to decide what belongs in global state.
---

# Skill: State Management (Redux + Redux Toolkit)

## Purpose

This skill defines how global client state is managed in this project using Redux Toolkit (RTK). Follow these patterns consistently across all features.

---

## What Goes in Redux State

Only put data in Redux if it meets one or more of these criteria:

- Shared across two or more unrelated components
- Needs to persist across route changes
- Drives UI decisions in multiple parts of the tree (e.g., auth state, theme, notifications)

**Do not** put form state, component-local UI state (open/closed, hover), or server-fetched data that is only used in one place into Redux. Use `useState` or a custom hook instead.

---

## Slice Pattern

One slice per feature domain. Keep slices in `src/store/<feature>/`.

```typescript
// src/store/auth/authSlice.ts
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
// src/store/auth/authThunks.ts
import { createAsyncThunk } from "@reduxjs/toolkit";
import { authService } from "@/services/authService";

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
// src/store/auth/authSelectors.ts
import type { RootState } from "@/store";

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
// src/store/index.ts
import { configureStore } from "@reduxjs/toolkit";
import authReducer from "./auth/authSlice";

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
// src/store/hooks.ts
import { useDispatch, useSelector } from "react-redux";
import type { RootState, AppDispatch } from "./index";

export const useAppDispatch = () => useDispatch<AppDispatch>();
export const useAppSelector = <T>(selector: (state: RootState) => T) =>
  useSelector(selector);
```

---

## Folder Structure

```
src/store/
  index.ts          ← configureStore + RootState + AppDispatch
  hooks.ts          ← useAppDispatch, useAppSelector
  auth/
    authSlice.ts
    authThunks.ts
    authSelectors.ts
  orders/
    ordersSlice.ts
    ordersThunks.ts
    ordersSelectors.ts
```

---

## Rules

- Never dispatch actions directly from components — use thunks or action creators
- Never access `store.getState()` outside of tests — use `useAppSelector` in components
- Keep reducers pure — no API calls, no side effects
- Keep slices focused — one domain per slice, not one slice for the entire app
- Export reducer as default, export actions and selectors as named exports
