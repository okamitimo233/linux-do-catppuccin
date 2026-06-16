# Project Structure

> File layout and organization for this userstyle project.

---

## Top-Level Layout

```
linux-do-catppuccin/
├── catppuccin.user.less     ← Main source: the entire theme
├── package.json             ← Node metadata + devDependencies
├── package-lock.json        ← Locked dependency tree
├── .stylelintrc.json        ← stylelint configuration
├── .gitignore               ← Ignores node_modules/
├── LICENSE                  ← MIT
├── README.md                ← English documentation
├── README_zh.md             ← Chinese documentation
├── .github/
│   └── workflows/
│       ├── lint.yml         ← CI: metadata validation + LESS compile + stylelint
│       └── bump-version.yml ← CI: auto-bump @version on push to main
└── .trellis/                ← Trellis project management (do not edit manually)
```

---

## Key Files

### `catppuccin.user.less`

The single source file for the userstyle. Contains:

1. **UserCSS metadata block** (`/* ==UserStyle== ... ==/UserStyle== */`) — Stylus configuration, version, and options
2. **LESS import** of the shared Catppuccin palette library
3. **`@-moz-document domain("linux.do")`** — scopes the style to linux.do
4. **Light/dark detection** via `prefers-color-scheme` and Discourse theme classes
5. **`#catppuccin()` mixin** — the core theme: CSS custom properties + element overrides

### `package.json`

Private package. Dev dependencies only:
- `less` — LESS compiler
- `postcss-less` — stylelint syntax adapter for LESS
- `stylelint` + `stylelint-config-standard` — CSS/LESS linting
- `usercss-meta` — UserCSS metadata parser

### `.stylelintrc.json`

Extends `stylelint-config-standard` with custom syntax `postcss-less`. Relaxes several rules that conflict with LESS conventions (vendor prefixes, class patterns, etc.) and adds `function-no-unknown` with a whitelist of LESS built-in functions.

See [`quality-guidelines.md`](./quality-guidelines.md) for linting details.

---

## What Doesn't Belong

- **Do not add build scripts** — the LESS file is served raw; no bundling step
- **Do not add TypeScript/React files** — this is a CSS-only project
- **Do not add server-side code** — the userstyle runs entirely in the browser via Stylus
- **Do not add additional CSS/LESS files** — the single-file design is deliberate for Stylus auto-update simplicity

---

## When Adding New Features

- All changes go in `catppuccin.user.less` — there is nowhere else
- Update both `README.md` and `README_zh.md` if the feature changes user-facing behavior
- The CI linter validates the file on every PR and push to main
