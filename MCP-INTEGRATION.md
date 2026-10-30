# WorkBuddy x GitHub MCP 接入记录

本仓库用于验证 WorkBuddy 通过自定义 MCP（Model Context Protocol）操作 GitHub 的完整链路。

## 接入方式

| 项目 | 值 |
| --- | --- |
| 传输协议 | `streamable-http` |
| 服务端 | `https://api.githubcopilot.com/mcp/`（GitHub 官方托管） |
| 配置文件 | `~/.workbuddy/mcp.json` |
| 认证方式 | fine-grained Personal Access Token，走 `Authorization: Bearer` 请求头 |
| 工具数量 | 46 个（context / repos / issues / pull_requests / users 五组）|

## 为什么用 PAT 而不是 OAuth

官方推荐 OAuth，但远程 MCP 的 OAuth 需要宿主应用自行注册 GitHub App。
本机实测授权回调在 3 分钟内未返回（日志停在第 `server-side-authorization-opened` 一步，
随后 `abort in-flight server-side auth poll`），因此改用 PAT —— 官方 README 亦注明
PAT 的兼容性优于 OAuth。

## 已验证能力

- [x] 搜索仓库（`search_repositories`，支持 `topic:` / `stars:>` 等限定符）
- [x] 读取目录结构（`get_file_contents`，支持按 `fields` 裁剪字段）
- [x] 读取文件全文（返回 blob SHA，可后续用于精确更新）
- [x] 列出 Pull Request（可按 state / sort / fields 裁剪）
- [x] 创建分支（`create_branch`，从 `main` 拉 `demo/first-commit`）
- [x] 通过 API 提交文件（本文件即由 MCP 写入，无本地 git 操作）

## 权限配置要点

fine-grained token 默认全部为 `No access`，需逐项开启：

| 权限 | 级别 | 用途 |
| --- | --- | --- |
| `Contents` | Read and write | 读写文件、建分支、提交 |
| `Pull requests` | Read and write | 创建 PR、评论 |
| `Issues` | Read and write | 可选，读写 Issue |
| `Administration` | Read and write | 可选，仅创建/删除仓库时需要 |

> 注意：`Administration` 仅在 Repository access 设为 All repositories 时才能覆盖新建仓库，
> 因为新仓库在授权时还不存在。

## 安全提醒

令牌以明文存放在 `mcp.json` 中，且本次调试过程中曾出现在对话记录里。
建议验收完成后撤销并重新签发，只写入本地文件。
