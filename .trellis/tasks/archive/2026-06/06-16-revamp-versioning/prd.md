# 重写主题的版本号管理机制，支持同日多版本区分并防止Stylus重复安装

## Goal

改用能在同一天内区分多个发布版本的版本号方案，同时确保 Stylus 更新时不会重复安装主题。

## 已确认事实（代码库调研）

### 当前版本号机制

- `@version` = `2026.06.16`（纯日期格式 `YYYY.MM.DD`）
- CI 自动在每次 push 到 main 时用 `sed` 替换为当天 UTC 日期
- 工作流：`.github/workflows/bump-version.yml`
- 同一天多次 push 版本号不变 → Stylus 不会检测到更新

### Stylus 版本比较行为

- Stylus 支持"任意数量的点分隔数字"和"0-padded 的 CalVer"
- 版本比较是数值比较：逐段比对，`2026.06.16.2` > `2026.06.16.1` 是合法的
- `usercss-meta` 的 `version` 字段用 semver-regex 校验（会 strip 前导 `v` 或 `=`）
- `usercss-meta` 的 semver-regex 来自 `semver` npm 包，接受格式 `X.Y.Z[-prerelease][+build]`，但 Stylus 本身在更新检查时不强制 semver

### Stylus 重复安装原因

- `@name` + `@namespace` 组合决定唯一性
- 当前 `@namespace` = `github.com/okamitimo233/linux-do-catppuccin`，稳定不变
- 重复安装最可能的原因是：`@namespace` 变化、或 Stylus 自身 bug（如 #2136）
- 如果 `@namespace` 和 `@name` 不变，版本号变更只会触发更新，不会重复安装

### 涉及文件

| 文件 | 角色 |
|------|------|
| `catppuccin.user.less` | 版本号所在位置（UserCSS 元数据块） |
| `.github/workflows/bump-version.yml` | CI 自动 bump 版本号 |
| `.github/workflows/lint.yml` | CI 用 `usercss-meta` 校验元数据 |

## Requirements

- [ ] 版本号能在同一天区分多个发布的先后顺序
- [ ] 更新后 Stylus 不会产生重复安装
- [ ] 版本号格式兼容 Stylus 的更新检查机制
- [ ] 版本号修改后 CI lint 通过（`usercss-meta` parse 不报错）
- [ ] 保持 `@namespace` 和 `@name` 不变（避免触发重复安装）

## Acceptance Criteria

- [ ] 同一天 push 两次，两次的 `@version` 不同且正确反映顺序
- [ ] 在 Stylus 中更新已安装的主题时，不会出现重复条目
- [ ] `usercss-meta` parse 不报错
- [ ] `bump-version.yml` 已删除，不再自动覆盖手动设置的版本号
- [ ] `@namespace` 和 `@name` 与当前值一致

## Decisions

1. **版本号格式** → 纯 SemVer（`MAJOR.MINOR.PATCH`，如 `1.0.0`），舍弃日期
2. **Bump 策略** → 手动改 `@version`，删除 `bump-version.yml` CI 工作流
3. **流程文档化** → 更新 `.trellis/spec/userstyle/metadata-and-config.md` 的版本号章节

## Open Questions

（已全部解答）
