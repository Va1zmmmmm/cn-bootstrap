# CN Bootstrap

中国大陆网络环境一键初始化工具。适用于全新安装的 Claude Code —— 自动配置所有包管理器的国内镜像源、GitHub 代理、API 中转方案，让 Claude Code 在国内网络下开箱即用。

## 覆盖范围

| 层级 | 工具 | 镜像策略 |
|------|------|----------|
| JS/TS | npm / yarn / pnpm | 淘宝 npmmirror |
| Python | pip / uv / conda | 清华 tuna |
| Git | clone / release / raw | kkgithub / gh-proxy |
| Go | go modules | goproxy.cn |
| Rust | cargo / rustup | 中科大 |
| Ruby | gem / bundler | Ruby China |
| macOS | Homebrew | 清华 |
| Docker | pull / daemon | 中科大 / 阿里云 |
| Claude Code | API 中转 | DeepSeek API |

## 使用方式

安装为 Claude Code Skill 后，触发词自动激活：

- "初始化国内环境"
- "装工具太慢了"
- "下载又被墙了"
- npm/pip/git clone 失败或超时时自动介入

## 执行原则

- 自动检测 → 自动配置 → 验证每步 → 汇报结果
- 镜像 1 不通自动降级到镜像 2、3
- 不永久修改配置（除非用户明确要求）

## 安装

```
npx skills add Va1zmmmmm/cn-bootstrap -g
```

## 许可证

MIT
