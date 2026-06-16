# Quality Guidelines

> Linting, CI, and code quality standards.

---

## Linting

This project uses **stylelint** with `stylelint-config-standard` and `postcss-less` for LESS syntax support.

### Running locally

```bash
npx stylelint catppuccin.user.less
```

### Configuration

See [`.stylelintrc.json`](../../../.stylelintrc.json). Key overrides from the standard config:

| Rule | Setting | Reason |
|------|---------|--------|
| `selector-class-pattern` | `null` | Discourse uses arbitrary class names we can't control |
| `custom-property-pattern` | `null` | Discourse variables don't follow standard naming |
| `selector-id-pattern` | `null` | May need ID selectors for Discourse elements |
| `property-no-vendor-prefix` | `null` | Userstyles often need vendor-prefixed properties |
| `function-no-unknown` | `[true, { ignoreFunctions: [...] }]` | LESS built-in functions (darken, fade, if, etc.) |

All `null` rules: `selector-class-pattern`, `custom-property-pattern`, `selector-id-pattern`, `rule-empty-line-before`, `comment-empty-line-before`, `at-rule-empty-line-before`, `declaration-empty-line-before`, `custom-property-empty-line-before`, `property-no-vendor-prefix`, `alpha-value-notation`, `color-function-notation`, `hue-degree-notation`, `keyframes-name-pattern`, `no-invalid-position-at-import-rule`, `no-invalid-double-slash-comments`, `font-family-no-missing-generic-family-keyword`, `selector-type-no-unknown`, `function-name-case`, `at-rule-no-vendor-prefix`, `no-descending-specificity`, `selector-not-notation`, `import-notation`

---

## CI Workflows

### Lint (`lint.yml`)

Triggered on:
- PRs that modify `catppuccin.user.less`
- Pushes to `main` that modify `catppuccin.user.less`
- Manual dispatch

Steps:
1. Checkout + Node setup
2. `npm ci`
3. **Validate UserCSS metadata** — parse with `usercss-meta`, check required fields
4. **Compile LESS** — `less.render()` in lint mode with default variable values
5. **Run stylelint**

### Bump Version (`bump-version.yml`)

Triggered on push to `main` that modifies `catppuccin.user.less` (skipped for `github-actions[bot]`).

Steps:
1. Bump `@version` to `$(date -u +%Y.%m.%d)`
2. Commit and push

---

## Forbidden Patterns

### ❌ Hardcoded colors

```less
/* BAD — not tied to the palette, won't change with flavor/accent */
color: #cdd6f4;
background-color: #1e1e2e;
```

```less
/* GOOD — uses palette variables */
color: @text;
background-color: @base;
```

### ❌ Rules outside `#catppuccin()` mixin

```less
/* BAD — won't receive flavor variables, always uses @darkFlavor */
.some-element { color: @text; }
```

```less
/* GOOD — inside the mixin, receives the correct flavor */
#catppuccin(@flavor) {
  .some-element { color: @text; }
}
```

### ❌ Using `!important`

```less
/* BAD — overrides everything, hard to debug */
.d-header { background-color: @mantle !important; }
```

```less
/* GOOD — use specificity or cascade */
.d-header { background-color: @mantle; }
```

### ❌ Duplicating palette definitions

Do not inline Catppuccin color values. Always use `#lib.palette()` and `#lib.defaults()`.

### ❌ Adding build/bundler config

This project has no build step. Do not add webpack, vite, rollup, or any bundler config.

---

## Testing (Manual)

There is no automated testing. Before submitting changes:

1. **Test all four flavors** — change the default flavor in the metadata block and reload
2. **Test light and dark modes** — toggle your OS/browser between light and dark
3. **Test all accent colors** — cycle through a few accent options in Stylus settings
4. **Test the full site** — navigate: homepage, topic list, a topic with replies, user profile, settings, search
5. **Run lint locally**: `npx stylelint catppuccin.user.less`

---

## Common Mistakes

1. **Forgetting to test latte** — most testing happens in dark mode; latte bugs are common
2. **Adding `@import` for additional files** — the single-file design is intentional; extra imports break Stylus auto-update
3. **Changing the `@updateURL`** — must always be the raw GitHub URL on `main`
4. **Committing with lint errors** — CI will fail the PR
