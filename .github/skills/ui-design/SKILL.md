---
name: ui-design
description: UI design principles for designer-ui-ux. Governs layout, typography, spacing, and visual hierarchy in all designs produced for this project.
---

# Skill: UI Design

## Purpose

UI design principles for `designer-ui-ux`. These govern layout, typography, spacing, and visual hierarchy in all designs produced for this project.

---

## Design Principles

1. **Clarity** — the purpose of every element should be immediately obvious
2. **Consistency** — same patterns for same interactions, everywhere
3. **Hierarchy** — guide the user's eye to the most important content first
4. **Feedback** — every action has a visible response
5. **Economy** — show only what's needed for the current task

## Working From Figma

- Use Figma MCP only when the user provides a Figma link or explicitly asks to work from Figma.
- When Figma is provided, treat the referenced frames and components as a concrete input to the design spec, then note any intentional deviations from the current design system.
- When Figma is not provided, do not assume a design file exists. Build the UI spec from user requirements plus the existing design system.

---

## Layout

### Grid

- Use a consistent column grid (8 or 12 columns depending on breakpoints)
- Align elements to the grid — avoid arbitrary positioning
- Use consistent gutters between columns

### Spacing Scale

Use a base-8 spacing scale: `4, 8, 12, 16, 24, 32, 48, 64, 96, 128`px

- Tight spacing (4–8px): within a component (label to input)
- Medium spacing (12–24px): between related elements
- Wide spacing (32–64px): between sections
- Section spacing (64px+): between major page sections

### Responsive Breakpoints

| Breakpoint | Width          |
| ---------- | -------------- |
| Mobile     | < 768px        |
| Tablet     | 768px – 1024px |
| Desktop    | > 1024px       |

---

## Typography

### Type Scale

| Style          | Usage                           |
| -------------- | ------------------------------- |
| Display / Hero | Page titles, marketing headings |
| H1             | Main page heading               |
| H2             | Section headings                |
| H3             | Sub-section headings            |
| Body Large     | Introductory paragraphs         |
| Body           | Default text                    |
| Body Small     | Supporting text, captions       |
| Label          | Form labels, tags, metadata     |
| Code           | Code snippets                   |

### Type Rules

- Body text: 16px minimum, 1.5 line height
- Heading line height: 1.1–1.3
- Line length: 60–80 characters for comfortable reading
- Don't use more than 2 typefaces in one UI
- Avoid all-caps for long text (acceptable for short labels/badges)

---

## Colour

### Semantic Colour Roles

Every project should define:

- **Primary** — main brand colour, primary actions
- **Secondary** — secondary actions, accents
- **Neutral** — backgrounds, borders, text
- **Success** — positive feedback (green family)
- **Warning** — cautionary states (amber family)
- **Error** — failure, destructive states (red family)
- **Info** — informational states (blue family)

### Colour Rules

- Use colour purposefully — not just decoratively
- Ensure sufficient contrast (see accessibility skill)
- Never use colour as the only differentiator

---

## Interactive Elements

### Buttons

| Variant      | Usage                                 |
| ------------ | ------------------------------------- |
| Primary      | One per view — the main action        |
| Secondary    | Alternative or supporting actions     |
| Destructive  | Irreversible actions (delete, remove) |
| Ghost / Text | Low-emphasis actions                  |

Button states: default → hover → focus → active → loading → disabled

### Form Fields

States: default → focus → filled → error → disabled

- Label always visible above the field
- Error message below the field
- Placeholder text is supplemental — never the only label

### Links vs Buttons

- Links → navigate somewhere
- Buttons → perform an action
- Never style a `<button>` to look like a link for navigation, or vice versa

---

## Iconography

- Use a consistent icon set throughout
- Icons must have a text label or `aria-label` when used alone
- Icon size should harmonise with adjacent text (typically 1–1.25× text size)
- Do not use icons as decorative elements without purpose

---

## Feedback & States

Every interactive feature needs designs for:
| State | Trigger |
|---|---|
| Loading | Async operation in progress |
| Empty | No data to display |
| Error | Operation failed |
| Success | Operation completed |
| Disabled | Action unavailable |

Never leave any of these states as an afterthought.

---

## Anti-Patterns to Avoid

- Inconsistent spacing (eyeballing instead of using the scale)
- Too many primary buttons on one view
- Placeholder text used as the only form label
- Text on low-contrast backgrounds
- Dense walls of text without visual hierarchy
- Hiding important actions in unexpected places
