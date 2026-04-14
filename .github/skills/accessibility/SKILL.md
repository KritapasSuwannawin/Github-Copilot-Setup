---
name: accessibility
description: Accessibility standards and implementation patterns for designer-ui-ux, frontend-developer, and tester-usability. Target WCAG 2.1 Level AA compliance.
---

# Skill: Accessibility (a11y)

## Purpose

Accessibility standards and implementation patterns for `designer-ui-ux`, `frontend-developer`, and `tester-usability`. Target: WCAG 2.1 Level AA compliance.

---

## Core Principles (POUR)

- **Perceivable** — information can be perceived by all users (including with assistive tech)
- **Operable** — UI is operable with keyboard and assistive tech
- **Understandable** — content and UI behaviour are understandable
- **Robust** — works with current and future assistive technologies

---

## Semantic HTML First

Use the right HTML element for the job — it provides accessibility for free:

```tsx
// [GOOD] Semantic
<button onClick={handleSubmit}>Save</button>
<nav aria-label="Main navigation">...</nav>
<main>...</main>
<article>...</article>

// [BAD] Non-semantic (requires manual ARIA to fix)
<div onClick={handleSubmit}>Save</div>
```

### Landmark Roles

Every page should have:

- `<header>` — site/page header
- `<nav>` — navigation (with `aria-label` if multiple navs)
- `<main>` — main content
- `<footer>` — footer

---

## Keyboard Navigation

- [ ] All interactive elements reachable via `Tab`
- [ ] Logical tab order (matches visual reading order)
- [ ] Visible focus indicator on all interactive elements (not `outline: none` without alternative)
- [ ] Modals trap focus inside when open, return focus to trigger when closed
- [ ] Dropdowns and menus operable with arrow keys
- [ ] `Escape` key closes modals, dropdowns, and dialogs

---

## Colour & Contrast

| Text Type                        | Minimum Contrast Ratio |
| -------------------------------- | ---------------------- |
| Normal text (< 18pt)             | 4.5:1                  |
| Large text (≥ 18pt or 14pt bold) | 3:1                    |
| UI components and graphics       | 3:1                    |

- [ ] Never convey information by colour alone — add an icon, label, or pattern
- [ ] Check contrast with a tool (e.g. Figma's contrast checker, axe DevTools)

---

## Images & Icons

```tsx
// [GOOD] Informative image
<img src="chart.png" alt="Bar chart showing Q3 revenue up 12% vs Q2" />

// [GOOD] Decorative image (hidden from screen readers)
<img src="decoration.png" alt="" role="presentation" />

// [GOOD] Icon button with accessible label
<button aria-label="Close dialog">
  <CloseIcon aria-hidden="true" />
</button>
```

---

## Forms

```tsx
// [GOOD] Accessible form field
<div>
  <label htmlFor="email">Email address</label>
  <input
    id="email"
    type="email"
    aria-required="true"
    aria-describedby="email-error"
  />
  {error && (
    <span id="email-error" role="alert">
      {error}
    </span>
  )}
</div>
```

Rules:

- Every input has a visible `<label>` (not just a placeholder)
- Use `aria-required="true"` for required fields
- Error messages use `role="alert"` for screen reader announcement
- Group related inputs with `<fieldset>` and `<legend>`

---

## Dynamic Content & ARIA

Use ARIA only when native HTML doesn't cover the case:

```tsx
// Live regions — announce dynamic changes to screen readers
<div aria-live="polite">
  {statusMessage}
</div>

// Dialog / Modal
<div role="dialog" aria-modal="true" aria-labelledby="dialog-title">
  <h2 id="dialog-title">Confirm deletion</h2>
  ...
</div>

// Loading state
<button aria-busy={isLoading} disabled={isLoading}>
  {isLoading ? 'Saving...' : 'Save'}
</button>
```

---

## Testing Accessibility

### Automated Tools (catches ~30-40% of issues)

| Tool                                      | Use Case                                                       |
| ----------------------------------------- | -------------------------------------------------------------- |
| `jest-axe`                                | Component-level tests (Jest + React Testing Library)           |
| `axe-playwright` (`@axe-core/playwright`) | E2E accessibility checks in Playwright                         |
| `axe DevTools` browser extension          | Manual page audit in Chrome/Firefox                            |
| `@axe-core/react`                         | Development-time warnings (dev only, not in production bundle) |

```typescript
// jest-axe example (component test)
import { axe, toHaveNoViolations } from 'jest-axe';
expect.extend(toHaveNoViolations);

it('should have no accessibility violations', async () => {
  const { container } = render(<LoginForm />);
  expect(await axe(container)).toHaveNoViolations();
});

// axe-playwright example (E2E test)
import AxeBuilder from '@axe-core/playwright';

test('login page should have no accessibility violations', async ({ page }) => {
  await page.goto('/login');
  const results = await new AxeBuilder({ page }).analyze();
  expect(results.violations).toEqual([]);
});
```

### Visual Accessibility Regression With Playwright

- Use Playwright screenshot assertions for visible focus states, inline error states, zoomed layouts, and key responsive breakpoints.
- Pair screenshot assertions with keyboard-only navigation so focus indicators and order are captured intentionally.
- Treat these checks as a complement to semantic and screen reader testing, not a replacement for them.

### Manual Checks

- [ ] Navigate the entire flow with keyboard only (no mouse)
- [ ] Test with screen reader (macOS VoiceOver or NVDA on Windows)
- [ ] Zoom to 200% — does layout still work?
- [ ] Test with Windows High Contrast mode

---

## Anti-Patterns to Avoid

- `outline: none` or `outline: 0` without a visible replacement
- Colour-only status indicators (red = error, green = success — add an icon too)
- `aria-label` on elements that already have visible text
- Positive `tabindex` values (anything other than `0` or `-1`)
- `<div>` and `<span>` with click handlers instead of `<button>`
- Auto-playing media without controls
