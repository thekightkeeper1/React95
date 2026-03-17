# React95 v6 Upgrade Technical Notes

These notes summarize the incompatibility between **React95 v4.0.0** and **styled-components v6**, which caused the scrollbar rendering issues and other potential UI bugs.

## 1. The Core Issue: Missing `&` Prefix
In `styled-components` v6 (which uses Stylis v4), any CSS selector that does **not** start with an ampersand (`&`) is strictly treated as a **descendant selector**.

### Example of Failure:
*   **Source Code:** `::-webkit-scrollbar { ... }`
*   **v5 Behavior (Loose):** Often correctly styled the element itself.
*   **v6 Behavior (Strict):** Compiles to `.sc-component-id ::-webkit-scrollbar`, targeting only scrollbars of *children*.

### Specific Files Identified:
*   `node_modules/react95/dist/common/index.js`: The `createScrollbars` utility lacks the `&` prefix for all `::-webkit-scrollbar` pseudo-elements.
*   `node_modules/react95/dist/TreeView/TreeView.js`: `:focus` selectors are missing the `&`.
*   Various components (e.g., `Digit`, `TextInput`) use pseudo-elements (`:before`, `:after`) without the `&` prefix.

## 2. The Migration Checklist (v6 Compatibility)
To fully upgrade the `react95` library for v6, the following tasks are required:

### A. Recursive Selector Prefixing
Every instance of a pseudo-selector or pseudo-element must be prefixed with `&` if it is intended to style the component itself:
- `&::-webkit-scrollbar`
- `&:hover`, `&:active`, `&:focus`
- `&:before`, `&:after`
- `&::placeholder`

### B. Transient Props (`$`)
`styled-components` v6 is more restrictive about passing non-standard HTML attributes to the DOM.
- All custom props used for styling (e.g., `variant`, `shadow`, `active`) should be converted to **transient props** (prefixed with `$`).
- Example: Change `props.active` to `props.$active` in both the React component and the styled-component definition.
- *Reference:* See React95 PR #398 for existing work on this.

### C. Dependency Updates
- Update `package.json` `peerDependencies` to `styled-components: >= 6.0.0`.
- Update `devDependencies` to the latest `styled-components` and `@types/styled-components`.

## 3. Reference Material
- **Main Tracking Issue:** [react95-io/React95 Issue #390](https://github.com/react95-io/React95/issues/390) - "Bump styled-components to v6"
- **Active PR:** [react95-io/React95 PR #398](https://github.com/react95-io/React95/pull/398) - "refactor: use transient props for styled-components v6+"

---
*Created on 2026-03-17 to guide the manual upgrade of the React95 fork.*
