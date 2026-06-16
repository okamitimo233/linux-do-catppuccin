# LESS Style Guide

> How the `catppuccin.user.less` file is organized and written.

---

## File Organization

The source file follows a strict section order. New code must follow the same layout:

1. **UserCSS metadata block** (`/* ==UserStyle== ... ==/UserStyle== */`)
2. **LESS `@import`** — the Catppuccin palette library
3. **`@-moz-document domain("linux.do")`** — scoping rule
4. **Light/dark detection** — `prefers-color-scheme` media queries + Discourse theme class overrides
5. **`#catppuccin()` mixin** — the core theme definition
   - CSS custom property declarations (Discourse color variables)
   - Element-level overrides (organized by Discourse component)
   - Flavor-specific tweaks

Reference: [`catppuccin.user.less`](../../../catppuccin.user.less)

---

## The `#catppuccin()` Mixin

The mixin is called with a single argument — the flavor variable (`@lightFlavor` or `@darkFlavor`). Inside:

```less
#catppuccin(@flavor) {
  #lib.palette();     // Load Catppuccin color definitions
  #lib.defaults();    // Load default helpers

  /* Expose Catppuccin colors as CSS custom properties */
  #lib.css-variables();

  /* ... theme rules ... */
}
```

### Mixin Sections (in order)

| Section | Purpose | Reference lines |
|---------|---------|-----------------|
| 1. Core colour variables | Maps Catppuccin colors to Discourse `--primary`, `--secondary`, `--tertiary`, etc. derivatives | Lines ~70-200 |
| 2. Code/syntax highlighting | `--hljs-*` variables | Lines ~205-220 |
| 3. Shadows | `--shadow-*` variables using Catppuccin surface colors | Lines ~225-260 |
| 4. Element-level overrides | Component-by-component style rules | Lines ~265-450 |
| 5. SVG/logo filters | Placeholder for inline SVG recoloring | Lines ~455-460 |
| 6. Flavor-specific tweaks | Latte contrast adjustments, dark-mode shadow removal | Lines ~465-480 |

---

## Color Variable Conventions

### Using Catppuccin palette colors

Always use the palette variables provided by `#lib.palette()`:
- Backgrounds: `@base`, `@mantle`, `@crust`, `@surface0`, `@surface1`, `@surface2`
- Text: `@text`, `@subtext0`, `@subtext1`, `@overlay0`, `@overlay1`, `@overlay2`
- Accent: `@accent` (set by user via Stylus config)
- Semantic: `@red`, `@green`, `@yellow`, `@blue`, `@pink`, `@mauve`, `@peach`, `@teal`

### Mapping to Discourse variables

Discourse uses its own CSS custom property naming convention. Map Catppuccin colors to Discourse variables in section 1 of the mixin:

```less
--primary: @text;
--secondary: @base;
--tertiary: @accent;
--header_background: @mantle;
// etc.
```

### Creating translucent variants

Use the LESS `fade()` function:

```less
--tertiary-very-low: fade(@accent, 10%);
--danger-low: fade(@red, 15%);
```

**Anti-pattern**: Do not use hardcoded rgba values like `rgba(203, 166, 247, 0.10)`. Always reference the palette variable with `fade()`.

### Latte-specific adjustments

Latte is a light theme — some colors need increased contrast. Use the flavor guard:

```less
& when (@flavor = latte) {
  --content-border-color: @surface1;
  --input-border-color: @surface2;
}
```

---

## Selector Patterns

### Discourse component selectors

Target Discourse's existing CSS classes. Never introduce new class names — the theme applies to a site we don't control.

Common selectors (from the source file):

| Component | Selector |
|-----------|----------|
| Header | `.d-header` |
| Sidebar | `.sidebar-section-wrapper`, `.sidebar-section-link` |
| Topic list | `.topic-list-item`, `.topic-title` |
| Posts | `.topic-body` |
| Buttons | `.btn-primary`, `.btn-danger`, `.btn-default` |
| Inputs | `input[type="text"]`, `textarea`, `select` |
| Modals | `.modal-inner-container`, `.modal-header` |
| Alerts | `.alert.alert-success`, `.alert.alert-error` |
| Menu panels | `.menu-panel` |
| Composer | `.composer-container`, `.reply-area` |

### Selector specificity

- Prefer single-class selectors (`.d-header`) over deeply nested selector chains
- Only increase specificity when Discourse's own styles require it
- Avoid `!important` — use specificity or cascade order instead

---

## Light/Dark Detection

The theme supports both `prefers-color-scheme` (automatic) and Discourse's explicit theme classes.

```less
/* Fallback: system preference */
:root {
  @media (prefers-color-scheme: light) { #catppuccin(@lightFlavor); }
  @media (prefers-color-scheme: dark)  { #catppuccin(@darkFlavor); }
}

/* Override: Discourse theme classes on :root */
:root[data-theme="dark"],
:root.theme-dark,
:root.dark {
  #catppuccin(@darkFlavor);
}

/* Override: Discourse theme classes on <html> */
html[data-theme="dark"],
html.theme-dark,
html.dark {
  #catppuccin(@darkFlavor);
}
```

**Rule**: The `:root` rules must come first (fallback), then `:root` attribute/class overrides, then `html` attribute/class overrides. This ensures Discourse's explicit theme always wins over system preference.

---

## Common Mistakes

1. **Using hardcoded colors instead of palette variables** — always use `@text`, `@surface0`, etc.
2. **Adding `!important`** — fix specificity instead
3. **Forgetting the latte flavor guard** — dark-theme contrast choices may look wrong in latte; test both light and dark
4. **Placing rules outside `#catppuccin()`** — all themed rules must be inside the mixin to receive the correct flavor
5. **Adding selectors without testing on the live site** — Discourse's DOM structure can vary between instances
6. **Removing or reordering metadata block fields** — CI validates the metadata block; see [metadata-and-config.md](./metadata-and-config.md)
7. **Overriding `--name` but forgetting `--name-rgb`** — Discourse uses `rgba(var(--*-rgb), <alpha>)` for semi-transparent elements (e.g. `.about` background). If you override `--secondary` to Catppuccin `@base` but leave `--secondary-rgb` unchanged, the `rgba()` call falls back to the original Discourse color and leaks through. Always generate matching `-rgb` variants using `red(@color), green(@color), blue(@color)`.
