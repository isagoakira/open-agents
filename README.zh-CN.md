# Open Agents

[![Deploy with Vercel](https://vercel.com/button)](https://vercel.com/new/clone?project-name=open-agents&repository-name=open-agents&repository-url=https%3A%2F%2Fgithub.com%2Fvercel-labs%2Fopen-agents&demo-title=Open+Agents&demo-description=Open-source+reference+app+for+building+and+running+background+coding+agents+on+Vercel.&demo-url=https%3A%2F%2Fopen-agents.dev%2F&env=POSTGRES_URL%2CBETTER_AUTH_SECRET%2CNEXT_PUBLIC_VERCEL_APP_CLIENT_ID%2CVERCEL_APP_CLIENT_SECRET%2CNEXT_PUBLIC_GITHUB_CLIENT_ID%2CGITHUB_CLIENT_SECRET%2CGITHUB_APP_ID%2CGITHUB_APP_PRIVATE_KEY%2CNEXT_PUBLIC_GITHUB_APP_SLUG%2CGITHUB_WEBHOOK_SECRET&envDescription=Neon+can+provide+POSTGRES_URL+automatically.+Generate+BETTER_AUTH_SECRET+yourself%2C+then+add+your+Vercel+OAuth+and+GitHub+App+credentials+for+a+full+deployment.&products=%255B%257B%2522type%2522%253A%2522integration%2522%252C%2522protocol%2522%253A%2522storage%2522%252C%2522productSlug%2522%253A%2522neon%2522%252C%2522integrationSlug%2522%253A%2522neon%2522%257D%252C%257B%2522type%2522%253A%2522integration%2522%252C%2522protocol%2522%253A%2522storage%2522%252C%2522productSlug%2522%253A%2522upstash-kv%2522%252C%2522integrationSlug%2522%253A%2522upstash%2522%257D%255D&skippable-integrations=1)

Open Agents 是一个用于在 Vercel 上构建和运行后台编码 Agent 的开源参考应用。它包含 Web UI、Agent 运行时、沙箱编排，以及从提示到代码变更所需的 GitHub 集成，无需使用您的笔记本电脑。

本仓库旨在被 fork 和定制，而不是被当作黑盒使用。

## 概述

Open Agents 是一个三层架构系统：

```text
Web -> Agent 工作流 -> 沙箱 VM
```

- Web 应用处理身份认证、会话、聊天和流式 UI。
- Agent 在 Vercel 上作为持久化工作流运行。
- 沙箱是执行环境：文件系统、Shell、Git、开发服务器和预览端口。

### 核心架构决策：Agent 不是沙箱

Agent 不在 VM 内部运行。它在沙箱外部运行，通过文件读取、编辑、搜索和 Shell 命令等工具与沙箱交互。

这种分离是项目的主要亮点：

- Agent 执行不绑定到单个请求生命周期
- 沙箱生命周期可以独立休眠和恢复
- 模型/提供商选择和沙箱实现可以独立演进
- VM 保持为纯执行环境，而不是变成控制平面

## 当前功能

- 支持文件、搜索、Shell、任务、技能和 Web 工具的聊天驱动编码 Agent
- 基于 Workflow SDK 的持久化多步骤执行，支持流式传输和取消
- 基于快照恢复的隔离式 Vercel 沙箱
- 在沙箱内克隆仓库和分支工作
- 可选的运行成功后的自动提交、推送和 PR 创建
- 通过只读链接分享会话
- 可选的通过 ElevenLabs 转录的语音输入

## 运行时说明

以下是理解当前实现需要关注的一些细节：

- 聊天请求启动工作流运行，而不是内联执行 Agent。
- 每个 Agent 轮次可以跨越多个持久化工作流步骤继续执行。
- 可以通过重新连接到现有工作流的流来恢复活动运行。
- 沙箱使用基础快照，暴露端口 `3000`、`5173`、`4321` 和 `8000`，并在不活动后休眠。
- 支持自动提交和自动 PR，但它们是偏好驱动的功能，不是始终开启的行为。

## 环境变量

请参阅 `apps/web/.env.example` 获取完整列表。概要：

### 最低运行时要求

```env
POSTGRES_URL=
BETTER_AUTH_SECRET=
```

### 登录所需（Vercel OAuth）

```env
NEXT_PUBLIC_VERCEL_APP_CLIENT_ID=
VERCEL_APP_CLIENT_SECRET=
```

### GitHub 仓库访问、推送和 PR 所需

```env
NEXT_PUBLIC_GITHUB_CLIENT_ID=
GITHUB_CLIENT_SECRET=
GITHUB_APP_ID=
GITHUB_APP_PRIVATE_KEY=
NEXT_PUBLIC_GITHUB_APP_SLUG=
GITHUB_WEBHOOK_SECRET=
```

### 可选

```env
REDIS_URL=                                      # 技能元数据缓存（回退到内存）
KV_URL=                                         # Vercel KV 缓存（回退到内存）
VERCEL_PROJECT_PRODUCTION_URL=                  # 规范生产 URL
NEXT_PUBLIC_VERCEL_PROJECT_PRODUCTION_URL=      # 公共规范生产 URL
VERCEL_SANDBOX_BASE_SNAPSHOT_ID=                # 覆盖默认沙箱快照
ELEVENLABS_API_KEY=                             # 语音转录
```

## 在 Vercel 上部署您自己的副本

1. Fork 这个仓库。
2. 将仓库导入 Vercel。如果您使用上面的部署按钮，Neon Postgres 会自动配置。
3. 生成会话签名的密钥：

   ```bash
   openssl rand -base64 32   # BETTER_AUTH_SECRET
   ```

4. 在 Vercel 项目设置中添加环境变量：

   ```env
   POSTGRES_URL=
   BETTER_AUTH_SECRET=
   ```

5. 首次部署以获取稳定的生产 URL。
6. 创建回调 URL 为以下地址的 Vercel OAuth 应用：

   ```text
   https://YOUR_DOMAIN/api/auth/callback/vercel
   ```

7. 添加以下环境变量并重新部署：

   ```env
   NEXT_PUBLIC_VERCEL_APP_CLIENT_ID=
   VERCEL_APP_CLIENT_SECRET=
   ```

8. 如果您需要完整的 GitHub 启用编码 Agent 流程，请使用以下信息创建 GitHub App：

   - 主页 URL：`https://YOUR_DOMAIN`
   - 回调 URL：`https://YOUR_DOMAIN/api/auth/callback/github`
   - 设置 URL：`https://YOUR_DOMAIN/api/github/app/callback`

   在 GitHub App 设置中：
   - 使用 GitHub App 的 Client ID 和 Client Secret 作为 `NEXT_PUBLIC_GITHUB_CLIENT_ID` 和 `GITHUB_CLIENT_SECRET`
   - 如果您希望组织安装能够正常工作，请将应用设为公开

9. 添加 GitHub App 环境变量并重新部署。
10. 可选择添加 Redis/KV 和规范生产 URL 变量。

## 本地设置

1. 安装依赖：

   ```bash
   bun install
   ```

2. 创建您的本地环境文件：

   ```bash
   cp apps/web/.env.example apps/web/.env
   ```

3. 填写 `apps/web/.env` 中所需的值。
4. 启动应用：

   ```bash
   bun run web
   ```

如果您已有链接的 Vercel 项目，可以使用 `vc env pull` 在本地拉取环境变量。

## OAuth 和集成设置

### Vercel OAuth

身份认证由 [Better Auth](https://www.better-auth.com/) 处理，Vercel 和 GitHub 作为社交提供商。所有认证路由都从 `/api/auth/[...all]` 捕获路由提供。

创建一个 Vercel OAuth 应用并使用以下回调：

```text
https://YOUR_DOMAIN/api/auth/callback/vercel
```

对于本地开发，使用：

```text
http://localhost:3000/api/auth/callback/vercel
```

然后设置：

```env
NEXT_PUBLIC_VERCEL_APP_CLIENT_ID=...
VERCEL_APP_CLIENT_SECRET=...
```

### GitHub App

您不需要单独的 GitHub OAuth 应用。Open Agents 使用 GitHub App 的 OAuth 凭据作为 Better Auth 社交提供商，加上 App 的安装令牌以访问仓库。

创建一个 GitHub App 用于基于安装的仓库访问并配置：

- 主页 URL：`https://YOUR_DOMAIN`
- 回调 URL：`https://YOUR_DOMAIN/api/auth/callback/github`
- 设置 URL：`https://YOUR_DOMAIN/api/github/app/callback`
- 如果您希望组织安装能够正常工作，请将应用设为公开

对于本地开发，使用 `http://localhost:3000` 作为主页 URL，`http://localhost:3000/api/auth/callback/github` 作为回调 URL，`http://localhost:3000/api/github/app/callback` 作为设置 URL。

然后设置：

```env
NEXT_PUBLIC_GITHUB_CLIENT_ID=...   # GitHub App Client ID
GITHUB_CLIENT_SECRET=...           # GitHub App Client Secret
GITHUB_APP_ID=...
GITHUB_APP_PRIVATE_KEY=...
NEXT_PUBLIC_GITHUB_APP_SLUG=...
GITHUB_WEBHOOK_SECRET=...
```

`GITHUB_APP_PRIVATE_KEY` 可以存储为转义换行符的 PEM 内容，或作为 base64 编码的 PEM。

## 有用的命令

```bash
bun run web                # 运行开发服务器
bun run check              # lint + 格式检查
bun run fix                # lint + 格式修复
bun run typecheck          # 类型检查所有包
bun run ci                 # 完整 CI：检查、类型检查、测试、迁移检查
bun run sandbox:snapshot-base  # 刷新沙箱基础快照
```

## 仓库布局

```text
apps/web         Next.js 应用、工作流、身份认证、聊天 UI
packages/agent   Agent 实现、工具、子 Agent、技能
packages/sandbox 沙箱抽象和 Vercel 沙箱集成
packages/shared  共享工具
```