# Metadata & Config

> UserCSS metadata block conventions and versioning.

---

## Metadata Block Location

The metadata block is the `/* ==UserStyle== ... ==/UserStyle== */` comment at the top of [`catppuccin.user.less`](../../../catppuccin.user.less). Stylus reads this block to display the theme name, version, and configuration UI.

---

## Required Fields

All of these are validated by CI (`lint.yml`):

| Field | Example value | Notes |
|-------|---------------|-------|
| `@name` | `Linux DO Catppuccin` | Display name in Stylus |
| `@namespace` | `github.com/okamitimo233/linux-do-catppuccin` | Unique identifier |
| `@version` | `1.0.0` | SemVer; manually bumped |
| `@description` | `Soothing pastel theme for Linux DO` | Short description |
| `@author` | `Catppuccin` | Theme author |
| `@license` | `MIT` | License identifier |
| `@preprocessor` | `less` | Tells Stylus to use LESS |

Reference: CI validation at [`.github/workflows/lint.yml`](../../../.github/workflows/lint.yml) lines 30-37.

---

## Additional Fields

| Field | Value |
|-------|-------|
| `@homepageURL` | `https://github.com/okamitimo233/linux-do-catppuccin` |
| `@updateURL` | `https://github.com/okamitimo233/linux-do-catppuccin/raw/main/catppuccin.user.less` |
| `@supportURL` | `https://github.com/okamitimo233/linux-do-catppuccin/issues` |

The `@updateURL` is critical — Stylus uses it for auto-update. It must always point to the raw file on the `main` branch.

---

## Stylus Configuration Variables

The metadata block defines three user-configurable options using `@var`:

```less
@var select lightFlavor "Light Flavor" ["latte:Latte*", "frappe:Frappé", ...]
@var select darkFlavor "Dark Flavor" ["latte:Latte", ..., "mocha:Mocha*"]
@var select accentColor "Accent" ["rosewater:Rosewater", ..., "mauve:Mauve*", ...]
```

### Variable format

```
@var <type> <name> "<label>" [<options>]
```

- **Type**: `select` (dropdown)
- **Name**: used as the LESS variable `@lightFlavor`, `@darkFlavor`, `@accentColor`
- **Label**: shown to the user in Stylus settings
- **Options**: `["value:DisplayName*", ...]` — `*` marks the default

### Available flavors

`latte`, `frappe`, `macchiato`, `mocha`

### Available accent colors

`rosewater`, `flamingo`, `pink`, `mauve`, `red`, `maroon`, `peach`, `yellow`, `green`, `teal`, `blue`, `sapphire`, `sky`, `lavender`, `subtext0`

---

## Versioning

### Strategy

Version bumps are **manual**. Edit the `@version` field directly in [`catppuccin.user.less`](../../../catppuccin.user.less). There is no CI auto-bump — the `bump-version.yml` workflow has been removed.

### Rules

- **Version format**: SemVer `MAJOR.MINOR.PATCH` (e.g. `1.0.0`). This is compatible with Stylus update checks (numeric segment comparison) and passes `usercss-meta` validation.
- **Increment `PATCH`** for bug fixes and minor style tweaks.
- **Increment `MINOR`** for new features, new `@var` options, or substantive restyling.
- **Increment `MAJOR`** for breaking changes (e.g. removing or renaming `@var` options that users may have customized).
- **The `@updateURL` must remain** `raw/main/catppuccin.user.less` — do not version-pin the URL.
- **Keep `@namespace` and `@name` unchanged** — changing them causes Stylus to treat the theme as a different style, producing duplicates.

---

## Metadata Validation in CI

The lint workflow (`lint.yml`) uses `usercss-meta` to parse and validate:

1. Parse the metadata block with `usercss-meta`'s `parse()`
2. Check for parse errors
3. Verify all required fields are present
4. Extract default variable values for LESS compilation
5. Compile LESS with `less.render()` in lint mode
6. Run `stylelint`

If you add a new required field, update the CI check's `required` array.

---

## Common Mistakes

1. **Changing `@preprocessor`** — must always be `less`; changing it breaks Stylus
2. **Removing `@updateURL`** — disables auto-update for all users
3. **Changing `@namespace`** — Stylus treats it as a different style, causing duplicates
4. **Alphabetizing metadata fields** — the order doesn't matter, but reordering just the CI `required` list without reordering the metadata is fine
5. **Adding `@var` without bumping `@version`** — bump `MINOR` manually when adding new config options so Stylus detects the update
