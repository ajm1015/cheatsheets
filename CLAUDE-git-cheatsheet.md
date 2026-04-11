# Git Cheatsheet — Build Spec

## Goal
Add `cheatsheets/git.html` to the GitHub Pages cheatsheet site and update the hub index to include it.

## What exists now
- `cheatsheets/index.html` — hub index page (SUN/NeXTSTEP warm workstation aesthetic) linking to existing sheets
- `cheatsheets/macos-shortcuts.html` — green terminal CRT
- `cheatsheets/windows-shortcuts.html` — SUN/NeXTSTEP
- `cheatsheets/windows-field-reference.html` — green terminal CRT
- Design reference file: use the attached `git-cheatsheet.html` as the **exact implementation** — do not redesign

## What done looks like
1. `cheatsheets/git.html` exists and matches the attached reference file exactly
2. `cheatsheets/index.html` has a new card/link for the git cheatsheet with appropriate title and description
3. All existing links on the index page still work
4. `git.html` opens standalone with all features functional: tab nav, `/` search, `Esc` clear, click-to-copy with toast, destructive flags visible

## Tasks
1. Copy the attached `git-cheatsheet.html` into `cheatsheets/git.html` verbatim
2. Add an entry to `cheatsheets/index.html` for the git cheatsheet — match the existing card pattern, use title "Git" with description "60 commands — click to copy"
3. Verify no broken links on index page

## What Claude Code gets wrong by default
1. **Redesigns the sheet** — the HTML is final. Copy it exactly. Do not refactor, restyle, or "improve" it.
2. **Mismatches the index card style** — read the existing index.html card markup pattern before adding. Match it precisely.
3. **Adds the card in the wrong position** — place it logically with the other sheets, not buried or prepended randomly.
4. **Forgets to commit** — stage both files in a single commit: `git add cheatsheets/git.html cheatsheets/index.html`
5. **Invents extra files** — this is a two-file change. Nothing else.
