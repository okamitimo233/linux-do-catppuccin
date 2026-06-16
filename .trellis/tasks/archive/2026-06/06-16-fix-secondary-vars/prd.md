# Fix secondary CSS variable mapping causing black labels on user profile

## Goal

修复 Catppuccin 主题中 `--secondary-*` 衍生变量（medium/high/very-high）错误映射到 `@crust` 导致用户信息界面标签文字（如 "加入日期："、"信任级别："）在暗色背景上不可读的问题。

## 根因

Discourse 用户信息页面的 `dt` 标签（"加入日期："、"信任级别："等）使用 `var(--secondary-medium)` 作为文字颜色（来源：`app/assets/stylesheets/common/base/user.scss`）。

我们的主题把 `--secondary-medium` 设为 `@crust`（#11111b），这是 Catppuccin 色板中最暗的颜色。在 moka 等暗色 flavor 下，`@base`（#1e1e2e）为背景，`@crust` 比背景更暗，导致标签文字几乎不可见（黑色 on 深色）。

相反，`dd` 标签（值：如 "2025年9月21日"、"活跃用户"）使用 `var(--primary)`，我们正确映射到了 `@text`（#cdd6f4，亮色），所以值正常显示。

问题出在对 `secondary-*` 衍生变量语义的理解不足：这些变量应该是**比 `@base` 更亮**的中间色调（如边框、次要文字），而不是更暗的颜色。`@crust` 是色板底部最暗的颜色，用于最深层背景，不应用作文本颜色。

## Requirements

1. `--secondary-medium` 改为 `@overlay0`（#6c7086），使 `dt` 标签在暗色背景下可读
2. `--secondary-high` 改为 `@overlay2`（#9399b2）
3. `--secondary-very-high` 改为 `@subtext1`（#bac2de）
4. 在 `less-style-guide.md` 中添加 gotcha #8 记录此教训

## Acceptance Criteria

- [x] 用户信息页面 `dt` 标签文字（"加入日期："、"信任级别："等）在暗色 flavor 下可读
- [x] `--secondary-medium` 在页面上不再为 #11111b
- [x] 不影响已有的其他 UI 元素样式

## Out of Scope

- 其他 CSS 变量的调整
- 非 `secondary-*` 衍生变量的改动
