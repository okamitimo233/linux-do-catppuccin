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
| `@version` | `2026.06.16` | Date-based; auto-bumped |
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

### Auto-bump workflow

The `bump-version.yml` workflow runs on every push to `main` that touches `catppuccin.user.less` (excluding pushes from the bot itself):

```bash
TODAY=$(date -u +%Y.%m.%d)
sed -i -E "s/@version[[:space:]]+.*/@version         ${TODAY}/" catppuccin.user.less
```

Reference: [`.github/workflows/bump-version.yml`](../../../.github/workflows/bump-version.yml)

### Rules

- **Version format**: `YYYY.MM.DD` (UTC date)
- **Never set `@version` manually** — the CI will overwrite it on push
- **The `@updateURL` must remain** `raw/main/catppuccin.user.less` — do not version-pin the URL

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
5. **Adding `@var` without updating `@version`** — let the CI handle it
