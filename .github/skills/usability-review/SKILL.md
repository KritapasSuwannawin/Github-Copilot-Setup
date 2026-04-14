---
name: usability-review
description: Usability evaluation framework for tester-usability, combining Nielsen's heuristics with practical UX checks for modern web applications.
---

# Skill: Usability Review

## Purpose

Usability evaluation framework for `tester-usability`, combining Nielsen's heuristics with practical UX checks for modern web applications.

---

## Nielsen's 10 Usability Heuristics

### 1. Visibility of System Status

- [ ] Users always know what's happening — loading states, progress indicators, confirmations
- [ ] Actions have immediate feedback (button press → visual change within 100ms)
- [ ] Long operations show progress, not just a spinner

### 2. Match Between System and Real World

- [ ] Language matches users' vocabulary — no technical jargon for non-technical users
- [ ] Icons are familiar and labelled
- [ ] Concepts map to real-world mental models

### 3. User Control and Freedom

- [ ] Users can undo or cancel actions
- [ ] Destructive actions are reversible or require confirmation
- [ ] Users can exit flows they started by mistake

### 4. Consistency and Standards

- [ ] Same actions look and behave the same throughout the app
- [ ] Consistent terminology — the same thing is called the same thing everywhere
- [ ] Follows platform conventions (browser back button works, links look like links)

### 5. Error Prevention

- [ ] Dangerous actions require confirmation
- [ ] Forms validate in real time where possible
- [ ] Clear constraints communicated before errors occur (e.g. password requirements shown upfront)

### 6. Recognition Rather Than Recall

- [ ] Options are visible — users don't need to remember commands
- [ ] Context is preserved across steps in a flow
- [ ] Recently used or relevant items are surfaced

### 7. Flexibility and Efficiency of Use

- [ ] Common actions are accessible quickly (not buried in menus)
- [ ] Power users can skip steps that novices need
- [ ] Keyboard shortcuts available for frequent actions

### 8. Aesthetic and Minimalist Design

- [ ] No irrelevant information cluttering the interface
- [ ] Information hierarchy is clear — most important content is prominent
- [ ] Visual noise is minimised

### 9. Help Users Recognise, Diagnose, and Recover from Errors

- [ ] Error messages are in plain language — not error codes
- [ ] Error messages explain what went wrong and how to fix it
- [ ] Errors appear close to where the problem is (inline form errors)

### 10. Help and Documentation

- [ ] Complex features have contextual help (tooltips, info icons)
- [ ] Empty states guide users on what to do next
- [ ] Error states offer a next action (retry, contact support, go back)

---

## Additional UX Checks

### All States Present

- [ ] **Loading** — user knows something is happening
- [ ] **Empty** — user knows what to do when there's no data
- [ ] **Error** — user knows something went wrong and what to do
- [ ] **Success** — user knows their action completed

### Forms

- [ ] Labels visible at all times (not just placeholders that disappear on focus)
- [ ] Required fields clearly marked
- [ ] Validation errors appear on field blur, not just on submit
- [ ] Submit button disabled or shows loading state while processing

### Navigation & Flow

- [ ] User always knows where they are in the application
- [ ] Breadcrumbs or back navigation available where appropriate
- [ ] Multi-step flows show progress
- [ ] Users don't lose form data unexpectedly

### Copy & Language

- [ ] Headings and labels are clear and descriptive
- [ ] CTAs (call-to-action buttons) describe the action: "Save changes" not "Submit"
- [ ] Confirmations are specific: "Delete account permanently?" not "Are you sure?"
- [ ] Error messages don't blame the user

## Evidence Collection

- [ ] Use Playwright-backed walkthroughs to capture screenshots for major states and any reported usability issue
- [ ] Record video when a flow is long, timing-sensitive, or hard to explain with still images
- [ ] Cite the supporting evidence in the report so reviewers can replay the issue

## Design Parity When Figma Is Provided

- [ ] If the user provided a Figma link, compare the implementation against the referenced frames, components, and tokens
- [ ] Call out spacing, typography, color, sizing, and state-behavior differences with evidence
- [ ] If no Figma link was provided, evaluate against `ui-spec.md` only and mark design-parity review as `N/A`

---

## Severity Levels

| Level      | Description                                              |
| ---------- | -------------------------------------------------------- |
| Blocking   | Prevents task completion or causes significant confusion |
| Major      | Noticeably degrades experience — should be fixed         |
| Minor      | Friction point — worth addressing                        |
| Suggestion | Enhancement idea, not a defect                           |

---

## Finding Format

```
## Usability Review: {Task Title}

### Spec Compliance
(Does implementation match ui-spec.md?)

### Evidence Collected

### Design Parity (if Figma source provided)

### Heuristics Violations
| # | Heuristic | Severity | Finding | Recommendation |
|---|---|---|---|---|

### Additional Findings
### Verdict: PASS / FAIL
```
