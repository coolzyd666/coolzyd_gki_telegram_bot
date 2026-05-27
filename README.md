# GKI Kernel Download Bot

<p align="center">
  <strong>基于 Cloudflare Workers 的 Telegram 机器人，用于获取 GKI 通用内核与 OnePlus OKI 内核下载</strong>
</p>

<p align="center">
  <img src="https://img.shields.io/badge/TypeScript-5.3-blue" alt="TypeScript">
  <img src="https://img.shields.io/badge/Cloudflare_Workers-Serverless-orange" alt="Cloudflare Workers">
  <img src="https://img.shields.io/badge/License-GPL--3.0-green" alt="License">
</p>

---

## 目录

- [功能概览](#功能概览)
- [技术架构](#技术架构)
- [前置要求](#前置要求)
- [安装与部署](#安装与部署)
- [环境变量配置](#环境变量配置)
- [机器人命令](#机器人命令)
  - [GKI 内核命令](#gki-内核命令)
  - [OnePlus OKI 内核命令](#oneplus-oki-内核命令)
  - [消息桥接命令](#消息桥接命令)
  - [管理员命令](#管理员命令)
- [支持的内核格式](#支持的内核格式)
- [权限与白名单机制](#权限与白名单机制)
- [HTTP 端点](#http-端点)
- [项目结构](#项目结构)
- [数据来源](#数据来源)
- [常见问题](#常见问题)
- [贡献指南](#贡献指南)
- [许可证](#许可证)
- [致谢](#致谢)

---

## 功能概览

本机器人是一个部署在 Cloudflare Workers 上的无服务器 Telegram Bot，主要提供以下功能：

- **GKI 内核查询与下载**：从 GitHub Release 获取 ReSukiSU + SUSFS 构建的 GKI 内核，支持标准版本与 LTS 长期支持版，提供直链下载和直接上传文件两种方式
- **OnePlus OKI 内核查询与下载**：专门为一加设备提供内核获取功能，支持按设备型号和系统版本模糊匹配，自动选择最新 OS 版本
- **智能版本匹配**：对用户输入的版本号进行规范化处理，支持精确匹配与 LTS 模糊匹配（例如输入 `6.1` 即可匹配 `6.1.X-lts`）
- **群组白名单管理**：基于 Cloudflare KV 的群组访问控制，非白名单群组自动退群，管理员可动态增删白名单
- **多级管理员体系**：硬编码超级管理员 + KV 动态管理员，超级管理员拥有添加/移除普通管理员及一键退群等高权限操作
- **消息桥接**：超级管理员之间可通过 Bot 在私聊中匿名转发消息，回复即可双向通信
- **群组忽略列表**：可设置忽略特定群组的所有消息，Bot 将静默处理，不产生任何回复
- **Telegram 论坛/话题支持**：完全兼容 Telegram 群组的 Topics 功能，回复自动归属到正确的话题线程
- **GitHub API 限流保护**：内置自动重试机制，针对 HTTP 403 限流和 5xx 错误进行指数退避重试

---

## 技术架构

| 组件 | 技术选型 | 说明 |
|------|---------|------|
| 运行时 | Cloudflare Workers | 无服务器边缘计算平台，无需维护服务器 |
| 语言 | TypeScript 5.3 | 全类型安全的单文件架构 |
| 存储 | Cloudflare KV | 用于白名单、管理员列表、忽略列表等持久化存储 |
| 构建工具 | Wrangler 3 | Cloudflare 官方 CLI 工具 |
| API 来源 | GitHub Releases API | 获取 GKI 和 OKI 内核发布信息 |
| 通信协议 | Telegram Bot API (Webhook) | 通过 Webhook 接收和处理消息 |

---

## 前置要求

- [Node.js](https://nodejs.org/) v16 或更高版本
- [Cloudflare 账户](https://dash.cloudflare.com/sign-up)（免费版即可运行 Workers）
- Telegram Bot Token（通过 [@BotFather](https://t.me/BotFather) 创建机器人获取）
- [Wrangler CLI](https://developers.cloudflare.com/workers/wrangler/)（`npm install -g wrangler` 或使用项目本地安装）

---

## 安装与部署

### 1. 克隆仓库

```bash
git clone https://github.com/coolzyd666/coolzyd_gki_telegram_bot.git
cd coolzyd_gki_telegram_bot
```

### 2. 安装依赖

```bash
npm install
```

### 3. 登录 Cloudflare

```bash
npx wrangler login
```

### 4. 创建 KV 命名空间

```bash
npx wrangler kv:namespace create "KV"
```

执行后将输出命名空间 ID，将其填入 `wrangler.toml`：

```toml
[[kv_namespaces]]
binding = "KV"
id = "你的KV命名空间ID"
```

### 5. 设置 Bot Token 密钥

```bash
npx wrangler secret put BOT_TOKEN
```

按提示输入从 @BotFather 获取的 Bot Token。**切勿将 Token 写入代码或配置文件中。**

### 6. 本地开发（可选）

```bash
npm run dev
```

### 7. 部署到 Cloudflare Workers

```bash
npm run deploy
```

### 8. 设置 Webhook

部署完成后，访问以下地址设置 Webhook：

```
https://你的Worker域名/setWebhook
```

### 9. 注册 Bot 命令菜单（可选）

访问以下地址注册 Telegram 命令菜单：

```
https://你的Worker域名/setCommands
```

---

## 环境变量配置

| 变量名 | 说明 | 配置方式 | 必需 |
|--------|------|---------|------|
| `BOT_TOKEN` | Telegram Bot Token，从 @BotFather 获取 | `wrangler secret put BOT_TOKEN` | 是 |
| `KV` | Cloudflare KV 命名空间绑定，用于存储白名单、管理员等数据 | 在 `wrangler.toml` 中配置 `[[kv_namespaces]]` | 是 |

### 超级管理员

Bot 内置了硬编码的超级管理员，无法被移除：

| 用户名 | Telegram ID |
|--------|-------------|
| `mc_zihan` | 7118282988 |
| `coolzyd9107` | 7282448230 |

> 如需修改超级管理员列表，需编辑 `src/index.ts` 中的 `SUPER_ADMINS` 常量并重新部署。

---

## 机器人命令

### GKI 内核命令

| 命令 | 说明 | 示例 |
|------|------|------|
| `/get_gki <版本号>` | 获取 GKI 内核的下载链接 | `/get_gki 5.10.101`、`/get_gki 6.1` |
| `/dl <版本号>` | 直接下载并上传 GKI 内核文件到聊天（受 50MB 限制） | `/dl 6.6.66`、`/dl 6.1` |
| `/list` | 列出当前可用的所有 GKI 内核版本（LTS 和标准版） | `/list` |

**版本号格式说明：**

- 标准版本：输入完整版本号，如 `5.10.101`、`6.6.66`
- LTS 版本：输入 `主.次` 即可自动匹配，如 `6.1` → 匹配 `6.1.X-lts`
- 也可显式指定 LTS：`6.1.X-lts`、`6.1.X`

当版本未找到时，Bot 会列出当前可用的 LTS 版本和最近的若干标准版本供参考。

---

### OnePlus OKI 内核命令

| 命令 | 说明 | 示例 |
|------|------|------|
| `/get_oki <型号> [系统]` | 获取 OnePlus 内核的下载链接 | `/get_oki ace5race`、`/get_oki ACE-5-RACE OOS16` |
| `/oki <型号> [系统]` | 直接下载并上传 OnePlus 内核文件到聊天（受 50MB 限制） | `/oki ace5race`、`/oki ACE-6T OOS16` |

**设备型号匹配规则：**

- 型号匹配不区分大小写，忽略连字符和下划线（`ACE-5-RACE`、`ace5race`、`Ace_5_Race` 均可）
- 优先精确匹配，若无结果则尝试子串匹配
- 系统版本可选，不指定时自动选择最新版本
- 未找到设备时，Bot 会列出所有可用设备供参考

---

### 消息桥接命令

| 命令 | 说明 | 示例 |
|------|------|------|
| `/msg <用户名> <消息内容>` | 向另一位超级管理员发送匿名消息 | `/msg @mc_zihan 你好` |

**使用说明：**

- 仅限超级管理员使用，且仅在**私聊**中生效
- 发送后，Bot 会将你的消息编辑为纯内容，然后转发给目标用户
- 目标用户直接回复转发消息即可返回回复，实现双向匿名通信
- 目标用户必须已与 Bot 开启过对话（先发送 `/start`）

---

### 管理员命令

所有 `/admin` 子命令仅管理员可使用，部分高权限操作仅限超级管理员。

#### 群组管理

| 命令 | 说明 | 权限 |
|------|------|------|
| `/admin list` | 查看所有白名单群组 | 管理员 |
| `/admin add <chatId> [群名]` | 添加群组到白名单 | 管理员 |
| `/admin remove <chatId>` | 从白名单移除群组（Bot 不会自动退出） | 管理员 |
| `/admin leaveall confirm` | 一键退出所有白名单群组并清空白名单 | 管理员 |

> `chatId` 通常为负数，例如 `-1001234567890`，可通过 Telegram API 或群组信息 Bot 获取。

#### 管理员管理

| 命令 | 说明 | 权限 |
|------|------|------|
| `/admin admins` | 查看所有管理员（含超级管理员和 KV 管理员） | 管理员 |
| `/admin addadmin <userId> [用户名]` | 添加普通管理员 | **仅超级管理员** |
| `/admin removeadmin <userId>` | 移除普通管理员（不可移除超级管理员） | **仅超级管理员** |

#### 忽略列表管理

| 命令 | 说明 | 权限 |
|------|------|------|
| `/admin ignorelist` | 查看被忽略的群组列表 | 管理员 |
| `/admin ignore <chatId> [原因]` | 忽略群组消息（Bot 将静默处理，不回复任何内容） | 管理员 |
| `/admin unignore <chatId>` | 取消忽略群组 | 管理员 |

---

## 支持的内核格式

### GKI 内核（来自 ReSukiSU-GKI）

| 格式 | 示例 | 说明 |
|------|------|------|
| 标准版 | `android12-5.10.101-2022-04-AnyKernel3.zip` | 包含完整版本号和日期 |
| LTS 版 | `android14-6.1.X-lts-AnyKernel3.zip` | 长期支持版，始终包含最新安全补丁 |

### OnePlus OKI 内核（来自 huangdihd）

| 格式 | 示例 |
|------|------|
| 命名规则 | `AK3_OP-{型号}_{系统}_android{版本}_ReSukiSU_{构建号}_SuSFS_{SUSFS版本}.zip` |
| 实例 | `AK3_OP-ACE-5-RACE_OOS16_android14-6.1.134_ReSukiSU_34681_SuSFS_v2.1.0.zip` |

---

## 权限与白名单机制

Bot 采用**白名单机制**控制群组访问，确保只在授权群组中提供服务：

1. **自动退群**：当 Bot 被添加到非白名单群组时，会发送提示消息后自动退出该群
2. **白名单存储**：所有白名单数据持久化存储在 Cloudflare KV 中，Worker 重启后不丢失
3. **忽略列表**：即使群组在白名单中，也可通过忽略列表使 Bot 静默处理其消息（不响应任何命令）
4. **管理员分层**：
   - **超级管理员**：硬编码在代码中，不可移除，拥有全部权限
   - **普通管理员**：由超级管理员通过 KV 动态管理，可执行大部分管理操作

---

## HTTP 端点

Bot 部署后提供以下 HTTP 端点：

| 路径 | 方法 | 说明 |
|------|------|------|
| `/` | GET | 健康检查，返回 Bot 状态信息（JSON） |
| `/setWebhook` | GET | 设置 Telegram Webhook，将 Bot 与 Worker 关联 |
| `/setCommands` | GET | 注册 Bot 命令菜单到 Telegram |
| `/` | POST | Telegram Webhook 回调（自动处理，无需手动调用） |

**健康检查返回示例：**

```json
{
  "status": "ok",
  "bot": "GKI Kernel Download Bot",
  "version": "1.1.0"
}
```

---

## 项目结构

```
coolzyd_gki_telegram_bot/
├── src/
│   └── index.ts          # 主程序：所有 Bot 逻辑、命令处理、API 交互
├── package.json          # 项目依赖与脚本配置
├── tsconfig.json         # TypeScript 编译配置（ES2021 + Cloudflare Workers Types）
├── wrangler.toml         # Cloudflare Workers 部署配置（含 KV 绑定）
├── LICENSE               # GPL-3.0 许可证
└── README.md             # 本文件
```

> 注意：本项目采用单文件架构，所有逻辑集中在 `src/index.ts` 中（约 2200 行），包含类型定义、工具函数、命令处理和 Webhook 入口。

---

## 数据来源

Bot 从以下 GitHub 仓库获取内核发布信息：

| 内核类型 | 仓库 | 说明 |
|---------|------|------|
| GKI 内核 | [ReSukiSU-GKI/GKI_KernelSU_SUSFS](https://github.com/ReSukiSU-GKI/GKI_KernelSU_SUSFS) | 集成 ReSukiSU + SUSFS 的 GKI 通用内核 |
| OnePlus OKI | [huangdihd/OnePlus_ReSukiSU_SUSFS](https://github.com/huangdihd/OnePlus_ReSukiSU_SUSFS) | 一加设备专用内核 |

---

## 常见问题

### 1. GitHub API 速率限制

Bot 内置了自动重试机制（最多 2 次），遇到 HTTP 403 限流时会进行指数退避等待。如果频繁遇到限流，可考虑为 GitHub API 请求添加认证 Token。

### 2. 文件过大无法上传

Telegram Bot API 对文件上传有 50MB 的限制。如果内核文件超过此大小，Bot 会自动回退为提供直接下载链接。你可以使用 `/get_gki` 或 `/get_oki` 命令仅获取链接，避免文件上传失败。

### 3. Bot 无响应

请依次检查以下内容：

- Worker 是否成功部署（在 Cloudflare Dashboard 中查看部署状态）
- Webhook 是否正确设置（访问 `/setWebhook` 端点）
- Bot Token 是否正确配置（通过 `wrangler secret put BOT_TOKEN` 设置）
- Bot 是否有群组管理员权限（需要读取消息和发送文件的权限）
- 该群组是否被添加到了忽略列表中

### 4. Bot 被拉入群后自动退出

这是因为该群组不在白名单中。请联系管理员使用 `/admin add <chatId> [群名]` 将群组添加到白名单后重新拉入 Bot。

### 5. KV 命名空间错误

确保 `wrangler.toml` 中的 KV 命名空间 ID 正确，且与你的 Cloudflare 账户中的命名空间匹配。可以通过 `npx wrangler kv:namespace list` 查看账户下所有命名空间。

### 6. 消息桥接不生效

- `/msg` 命令仅在**私聊**中可用，在群组中发送将被忽略
- 仅超级管理员之间可以互相发送消息
- 目标用户必须已与 Bot 开启过对话（先发送 `/start`）

---

## 贡献指南

1. Fork 本仓库
2. 创建功能分支：`git checkout -b feature/amazing-feature`
3. 提交更改：`git commit -m 'Add some amazing feature'`
4. 推送到分支：`git push origin feature/amazing-feature`
5. 发起 Pull Request

---

## 许可证

本项目采用 [GNU General Public License v3.0](LICENSE) 许可证。

---

## 致谢

- [Cloudflare Workers](https://workers.cloudflare.com/) — 无服务器运行平台
- [Telegram Bot API](https://core.telegram.org/bots/api) — Bot 通信框架
- [ReSukiSU](https://github.com/ReSukiSU/ReSukiSU) — 基于内核的 Root 解决方案
- [ReSukiSU-GKI](https://github.com/ReSukiSU-GKI) — GKI 内核构建与发布
- [huangdihd](https://github.com/huangdihd) — OnePlus OKI 内核构建与发布
- [SUSFS](https://github.com/siimsek/susfs) — Root 隐藏解决方案
