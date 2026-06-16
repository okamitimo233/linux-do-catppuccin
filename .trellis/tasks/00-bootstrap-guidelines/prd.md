# Bootstrap Task: Linux DO Catppuccin — Spec Bootstrap Complete

**This task created project-specific `.trellis/spec/` files.**

## Result

The template specs (frontend React/web-app patterns) were completely wrong for this project, which is a single-file Stylus userstyle. All template files were deleted and replaced with project-specific specs.

### Spec files created

| File | Contents |
|------|----------|
| `.trellis/spec/userstyle/index.md` | Overview, architecture diagram, quick reference |
| `.trellis/spec/userstyle/project-structure.md` | File layout, what goes where, what doesn't belong |
| `.trellis/spec/userstyle/less-style-guide.md` | LESS file organization, mixin sections, selector patterns, color conventions |
| `.trellis/spec/userstyle/metadata-and-config.md` | UserCSS metadata block, Stylus variables, versioning, CI validation |
| `.trellis/spec/userstyle/quality-guidelines.md` | Linting, CI workflows, forbidden patterns, manual testing checklist |

### Files deleted (templates that didn't apply)

All of `.trellis/spec/frontend/` (6 files: directory-structure, component-guidelines, hook-guidelines, state-management, type-safety, quality-guidelines) and `.trellis/spec/guides/` (3 files: index, cross-layer-thinking-guide, code-reuse-thinking-guide).

### Project analysis summary

- **Type**: Stylus userstyle (CSS injection for linux.do)
- **Source**: Single file `catppuccin.user.less` (LESS → CSS)
- **Architecture**: UserCSS metadata → LESS import → `#catppuccin()` mixin called per color scheme → CSS custom properties + Discourse element overrides
- **Dependencies**: `less`, `usercss-meta`, `stylelint`, `postcss-less`
- **CI**: Metadata validation + LESS compilation + stylelint lint; auto version bump

## Completion

Run:
```bash
python3 ./.trellis/scripts/task.py finish
python3 ./.trellis/scripts/task.py archive 00-bootstrap-guidelines
```
