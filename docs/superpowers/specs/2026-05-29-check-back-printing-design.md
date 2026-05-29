# Check Back Printing Design

## Goal

Add check back generation to the existing Vue check printer without changing the core check creation workflow. The main preview should show the check front and check back on one printable sheet with a gap between them for cutting. Users should also be able to print only the front or only the back.

## Assumptions

- The back template source is `/Users/teddy/Downloads/back.svg`.
- The SVG will be moved into the app assets as `printmechecks/src/assets/check-back.svg`.
- The current check front layout remains the source of truth for signature text.
- The back-side endorsement text should use the same handwritten signature styling as the front signature.
- Existing saved history entries may not have the new endorsement fields, so missing values must default cleanly.

## User Workflow

The default preview renders a single sheet with:

1. The existing check front.
2. A visible blank gap for cutting.
3. The check back template.

The form adds two mutually exclusive checkbox controls:

- `Mobile`: prints `For Mobile Deposit Only` plus the current signature on the check back.
- `Endorse`: enables a custom endorsement input and prints that custom text plus the current signature on the check back.

If neither checkbox is selected, the check back prints only the current signature.

When `Mobile` is checked, `Endorse` is disabled until `Mobile` is unchecked. When `Endorse` is checked, `Mobile` is disabled until `Endorse` is unchecked.

## Architecture

Keep the implementation inside `CheckPrinter.vue` to match the current project structure and avoid introducing a component split for a small feature.

Add a wrapper around the printable preview, for example `check-sheet`, containing:

- `check-front`: the existing front layout.
- `check-cut-gap`: a blank spacing element.
- `check-back`: the back layout using the imported SVG template.

The back layout should position endorsement text over the right-side endorsement area of the SVG template. The template itself should provide the standard border, security text, watermark, and endorsement guide lines.

## Data Model

Add these fields to the local check object:

- `endorsementMode`: one of `none`, `mobile`, or `custom`.
- `endorsementText`: custom endorsement text used only when `endorsementMode` is `custom`.

Existing local storage objects should load as:

- `endorsementMode: 'none'`
- `endorsementText: ''`

History saving should include the new fields so reopening a saved check restores the back-side endorsement state.

## Print Behavior

Replace the current single print path with a target-based print function:

- `Print Front`: prints only the front check region.
- `Print Back`: prints only the back check region.
- `Ctrl+P`: prints the full sheet containing front, gap, and back.

The print CSS should preserve the existing no-margin output behavior and hide the editor controls. It should show only the selected printable region for front/back printing, and only the sheet for Ctrl+P printing.

## Error Handling

No new runtime error UI is required.

If a legacy history entry does not contain endorsement fields, default values are applied during initialization or restore. If the custom endorsement mode is selected with an empty custom text, the back still prints the signature.

## Testing

Because the project has no automated test runner, verification should use the existing project commands:

- `npm run type-check`
- `npm run build`
- `npm run lint`

Manual browser verification should cover:

- Default preview shows front, cutting gap, and back.
- `Print Front` hides the editor and prints only the front.
- `Print Back` hides the editor and prints only the back.
- `Ctrl+P` prints the combined front/back sheet.
- `Mobile` and `Endorse` cannot both be selected.
- `Mobile` prints `For Mobile Deposit Only` plus signature.
- `Endorse` prints custom endorsement text plus signature.
- Neither checkbox selected prints only the signature.
- Saved history restores the endorsement mode and custom endorsement text.
