# 覆写 Discourse --*-rgb CSS 变量，修复用户信息区域背景色泄漏

## Goal

修复 Catppuccin 主题中缺失的 `--*-rgb` CSS 变量覆写，确保 Discourse 使用 `rgba(var(--*-rgb), ...)` 模式渲染的元素也显示 Catppuccin 色板，而非回退到原始 Discourse 暗色主题的颜色。

## 背景 / 根因分析

通过 Browser Lens 对 linux.do 页面 (`/u/lyz215328/activity`) 的分析：

1. Discourse 在 CSS 中大量使用 `rgba(var(--secondary-rgb), 0.85)` 这样的半透明颜色模式
2. Catppuccin 主题只覆写了 `--secondary: @base`（#1e1e2e），但**未覆写** `--secondary-rgb: 34, 34, 34`
3. 因此 `rgba(var(--secondary-rgb), ...)` 展开后使用的是原始 Discourse 暗色 (#222222)，而非 Catppuccin #1e1e2e
4. `.about` 用户信息区域的背景正是通过这种模式渲染的，导致泄漏原始背景色

影响范围：以下 10 个 `-rgb` 变量均未覆写（Catppuccin mocha 色板对应的值）：

| 变量 | 当前值（Discourse 原始） | 应覆写为（Mocha） |
|------|------------------------|-------------------|
| `--primary-rgb` | `221, 221, 221` | `red(@text)`, `green(@text)`, `blue(@text)` → `205, 214, 244` |
| `--secondary-rgb` | `34, 34, 34` | `red(@base)`, ... → `30, 30, 46` |
| `--tertiary-rgb` | `9, 157, 215` | `red(@accent)`, ... → 取决于 accent 选择 |
| `--primary-low-rgb` | `49, 49, 49` | `red(@surface0)`, ... → `49, 50, 68` |
| `--primary-very-low-rgb` | `40, 40, 40` | `red(@surface0)`, ... → `49, 50, 68` |
| `--header_background-rgb` | `17, 17, 17` | `red(@mantle)`, ... → `24, 24, 37` |
| `--highlight-rgb` | `168, 113, 55` | `red(@yellow)`, ... → `249, 226, 175` |
| `--success-rgb` | `28, 165, 81` | `red(@green)`, ... → `166, 227, 161` |
| `--chat-skeleton-animation-rgb` | `40, 40, 40` | `red(@surface0)`, ... → `49, 50, 68` |
| `--always-black-rgb` | `0, 0, 0` | 保持不变 (`0, 0, 0`) — 语义上永远黑色 |

## Requirements

1. 在 `#catppuccin()` mixin 的 "Core Discourse colour variables" 区域中添加 `-rgb` 变量覆写
2. 使用 LESS 内置 `red()`/`green()`/`blue()` 函数从 Catppuccin 色板变量动态生成 RGB 分量
3. `--always-black-rgb` 保持 `0, 0, 0` 不变
4. 所有 flavor（latte/frappe/macchiato/mocha）都应生成正确的 RGB 值

## Acceptance Criteria

- [ ] 在 linux.do 用户活动页面，`.about` 区域和所有相关元素显示 Catppuccin 色板颜色，无原始 Discourse 颜色泄漏
- [ ] 所有 9 个需覆写的 `-rgb` 变量在 DOM 中显示为对应 Catppuccin 色板的 RGB 值
- [ ] `--always-black-rgb` 保持 `0, 0, 0`
- [ ] 不影响已有的 non-rgb 变量覆写
- [ ] Mocha 和 Latte flavor 切换时 RGB 值同步更新

## Out of Scope

- 其他 Discourse CSS 变量的额外覆写
- 非 linux.do 站点的适配

## Implementation Approach

采用**方案 B：封装本地 mixin** 生成 `-rgb` 变量，避免 LESS 3.x+ 参数化 mixin 的作用域限制，同时减少重复。

```less
// 在 #catppuccin() 内部定义
.rgb-var(@name, @color) {
  --@{name}-rgb: red(@color), green(@color), blue(@color);
}

// 调用
.rgb-var(primary, @text);
.rgb-var(secondary, @base);
.rgb-var(tertiary, @accent);
.rgb-var(primary-low, @surface0);
.rgb-var(primary-very-low, @surface0);
.rgb-var(header_background, @mantle);
.rgb-var(highlight, @yellow);
.rgb-var(success, @green);
.rgb-var(chat-skeleton-animation, @surface0);
```

`--always-black-rgb` 保持 `0, 0, 0` 不变，不通过 mixin 生成。

## Notes

- lib.less 提供了 `#lib.rgbify(@color)` mixin，但因其参数化 mixin 的变量作用域限制（LESS 3.x+），不直接使用
- LESS 的 `red()`/`green()`/`blue()` 函数在编译时求值，无运行时开销
