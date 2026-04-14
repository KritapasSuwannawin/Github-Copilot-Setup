---
name: design-system
description: Guidelines for building and consuming a consistent design system across frontend components. Used by designer-ui-ux and frontend-developer.
---

# Skill: Design System

## Purpose

Guidelines for building and consuming a consistent design system across frontend components. Used by `designer-ui-ux` and `frontend-developer`.

---

## What is a Design System?

A design system is the single source of truth for:

- **Design tokens** — the values (colours, spacing, typography) used consistently
- **Component library** — reusable UI building blocks
- **Patterns** — solutions to recurring design problems
- **Documentation** — how and when to use each piece

---

## Design Tokens

Define all visual values as tokens — never hardcode values in components.

```typescript
// tokens.ts (or CSS custom properties)
export const tokens = {
  colors: {
    primary: {
      50: "#eff6ff",
      500: "#3b82f6",
      900: "#1e3a8a",
    },
    neutral: {
      0: "#ffffff",
      100: "#f5f5f5",
      900: "#111111",
    },
    error: "#ef4444",
    success: "#22c55e",
    warning: "#f59e0b",
  },
  spacing: {
    1: "4px",
    2: "8px",
    3: "12px",
    4: "16px",
    6: "24px",
    8: "32px",
    12: "48px",
    16: "64px",
  },
  typography: {
    fontFamily: {
      sans: '"Inter", system-ui, sans-serif',
      mono: '"JetBrains Mono", monospace',
    },
    fontSize: {
      xs: "12px",
      sm: "14px",
      base: "16px",
      lg: "18px",
      xl: "20px",
      "2xl": "24px",
      "3xl": "30px",
      "4xl": "36px",
    },
  },
  radii: {
    sm: "4px",
    md: "8px",
    lg: "12px",
    full: "9999px",
  },
  shadows: {
    sm: "0 1px 2px rgba(0,0,0,0.05)",
    md: "0 4px 6px rgba(0,0,0,0.07)",
    lg: "0 10px 15px rgba(0,0,0,0.10)",
  },
};
```

## Figma Alignment When Provided

- Use Figma MCP only when the user provides a Figma link or explicitly asks to work from Figma.
- When Figma is provided, map the referenced frames, components, and tokens to the existing design system before proposing new variants.
- Record the Figma references used in `ui-spec.md` or implementation notes so downstream reviewers know which source was followed.
- When no Figma link exists, Storybook and the existing design system remain the source of truth.

---

## Component Anatomy

Every design system component should be documented with:

1. **Purpose** — what problem it solves and when to use it
2. **Variants** — all visual/functional variations
3. **Props API** — all configurable properties
4. **States** — default, hover, focus, active, disabled, loading, error
5. **Do / Don't** — usage examples

---

## Core Component Checklist

Before building a new feature, check if these exist. Build them if they don't:

### Foundation

- [ ] Button (primary, secondary, destructive, ghost, icon-only)
- [ ] Input (text, email, password, number, textarea)
- [ ] Select / Dropdown
- [ ] Checkbox & Radio
- [ ] Toggle / Switch
- [ ] Label
- [ ] Badge / Tag

### Feedback

- [ ] Alert / Banner (success, warning, error, info)
- [ ] Toast / Notification
- [ ] Skeleton loader
- [ ] Spinner / Loading indicator
- [ ] Empty state
- [ ] Error state

### Navigation

- [ ] Link
- [ ] Breadcrumb
- [ ] Tabs
- [ ] Pagination

### Overlay

- [ ] Modal / Dialog
- [ ] Drawer / Side panel
- [ ] Tooltip
- [ ] Popover

### Layout

- [ ] Card
- [ ] Divider
- [ ] Stack / Flex layout primitive
- [ ] Grid layout primitive

---

## Component Rules

### Reuse Before Building

- Always check the component library before creating a new component
- Extend an existing component rather than building a parallel one
- If you need a new component, document it in the design system first

### Component Composition

- Build components from smaller components — don't monolith
- Use `children` for flexible content slots
- Use compound components for complex patterns (e.g. `<Modal>`, `<Modal.Header>`, `<Modal.Body>`)

### Theming

- Components must use design tokens — never hardcoded values
- Support light and dark themes via CSS custom properties
- Test all components in both themes

---

## Storybook

**Storybook is the component documentation standard for this project.** Every design system component must have a corresponding story.

### Story Requirements

Each component story must include:

1. **Default** — the component in its baseline state
2. **All variants** — one story per visual/functional variant
3. **All states** — disabled, loading, error, empty (where applicable)
4. **Interactive controls** — use `argTypes` so reviewers can explore props in the Storybook UI
5. **Accessibility check** — include the `@storybook/addon-a11y` panel to surface violations

```typescript
// Button.stories.tsx
import type { Meta, StoryObj } from "@storybook/react";
import { Button } from "./Button";

const meta: Meta<typeof Button> = {
  title: "Foundation/Button",
  component: Button,
  tags: ["autodocs"],
};
export default meta;
type Story = StoryObj<typeof Button>;

export const Primary: Story = {
  args: { variant: "primary", children: "Save changes" },
};

export const Disabled: Story = {
  args: { variant: "primary", children: "Save changes", disabled: true },
};
```

### Rules

- Never ship a new component without a story
- Stories live alongside the component: `Button.tsx` → `Button.stories.tsx`
- Storybook must remain buildable and free of errors in CI

## Visual Regression With Playwright

- Use Playwright screenshot assertions for stable component or page states where layout drift would be expensive.
- Storybook stories are good screenshot targets because they are deterministic and isolate a single component state.
- Cover the default state plus important variants such as loading, error, empty, and responsive breakpoints when those states matter.
- If a Figma link exists, use Figma screenshots as a review reference, not as the automated regression baseline.

---

## Versioning Design System Changes

When modifying existing components:

- **Non-breaking** (add optional prop, new variant): no version bump needed
- **Breaking** (rename prop, remove variant, change behaviour): requires team discussion and migration plan
- Document all changes in a `CHANGELOG.md`

---

## Anti-Patterns to Avoid

- Hardcoding hex colours in component files
- One-off components built for a single feature without generalising
- Component variants that could be achieved by composing existing components
- Skipping documentation for new components
- Building the same pattern differently in different parts of the app
