---
name: agent-output-review
description: "Use when: You are about to write or modify .md files in the .github/docs/ folder. You must pause and request user approval before taking any further steps. This ensures all agent-produced documentation is reviewed and approved by user before the workflow continues."
applyTo: "**/.github/docs/**/*.md"
---

# Agent Output Review Gate

## Checkpoint Rule

Whenever you write or modify **any** `.md` file in the `.github/docs/` folder, you **MUST**:

1. **List all files created/modified** in this session
2. **Report the paths** relative to the workspace root (e.g., `.github/docs/feature-name/tasks.md`)
3. **Pause and explicitly request user approval** with the following message:

```
---
## Output Review Required

I have created/modified the following documentation files:

- [ ] .github/docs/{feature}/{filename}.md
- [ ] .github/docs/{feature}/{task}/{filename}.md

**Next Step:** Please review these files in your editor. You may:
- Read them as-is
- Edit them directly in the editor
- Request changes via chat

Once you've reviewed and are satisfied, reply with:
> Approved. Continue to the next step.

I will not proceed until you explicitly approve.
---
```

## Approval Pattern

You must WAIT for the user to explicitly reply with approval language such as:

- "Approved. Continue."
- "Looks good. Proceed."
- "Approved."
- Any clear confirmation that review is complete

Do **not** assume approval or proceed automatically after a long silence.

## If User Edits Files

If the user edits the `.md` files directly in the editor:

- Acknowledge the edits
- Confirm you've read the updated content
- Ask for approval again if needed

## Exception: Non-Doc Outputs

This rule applies **only** to `.md` files in `.github/docs/`. Other outputs (code, config files, etc.) follow normal workflow—no review gate needed unless explicitly instructed.

---

**Purpose:** This gate ensures high-quality documentation by giving you a checkpoint to refine agent outputs before they inform downstream work.
