# Userstyle Development Guidelines

> Conventions for the Linux DO Catppuccin Stylus userstyle.

---

## Overview

This project is a single-file [Stylus](https://github.com/openstyles/stylus) userstyle that applies the Catppuccin theme to [linux.do](https://linux.do) (a Discourse forum). The source is a LESS file that compiles to CSS and uses the shared Catppuccin palette library.

---

## Guidelines Index

| Guide | Description |
|-------|-------------|
| [Project Structure](./project-structure.md) | File layout — what goes where |
| [LESS Style Guide](./less-style-guide.md) | How the `.user.less` file is organized and written |
| [Metadata & Config](./metadata-and-config.md) | UserCSS metadata block, versioning, Stylus variables |
| [Quality Guidelines](./quality-guidelines.md) | Linting, CI checks, forbidden patterns |

---

## Quick Reference

### Source file

```
catppuccin.user.less    ← The entire theme (LESS → CSS)
```

### Key dependencies

| Tool | Purpose |
|------|---------|
| `less` | Compiles LESS to CSS |
| `usercss-meta` | Parses and validates UserCSS metadata |
| `stylelint` + `postcss-less` | Lints the LESS file |
| `@import "https://userstyles.catppuccin.com/lib/lib.less"` | Catppuccin palette and helpers |

### Architecture at a glance

```
UserCSS metadata (Stylus config vars)
  ↓
LESS import → Catppuccin palette lib (#lib.palette(), #lib.defaults())
  ↓
#catppuccin() mixin — called once per detected color scheme
  ├── CSS custom property declarations (Discourse --primary, --tertiary, etc.)
  ├── Element-level overrides (.d-header, .sidebar-section-link, etc.)
  └── Flavor-specific tweaks (latte vs dark)
```
