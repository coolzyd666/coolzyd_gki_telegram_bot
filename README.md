# Telegram GKI Bot

A Telegram bot for GKI (Generic Kernel Image) and OnePlus OKI (OnePlus Kernel Image) kernel downloads, deployed on Cloudflare Workers.

一个用于 GKI (通用内核镜像) 和 OnePlus OKI (一加内核镜像) 内核下载的 Telegram 机器人，部署在 Cloudflare Workers 上。

---

## Features | 功能特性

- **GKI Kernel Downloads**: Fetch and download GKI kernels from GitHub releases  
  **GKI 内核下载**：从 GitHub Release 获取并下载 GKI 内核

- **OnePlus OKI Support**: Specialized support for OnePlus device kernels  
  **一加 OKI 支持**：专门支持一加设备内核

- **Version Matching**: Intelligent kernel version matching with LTS support  
  **版本匹配**：智能内核版本匹配，支持 LTS 长期支持版

- **Admin Management**: Built-in admin system with super admins and dynamic admin list  
  **管理员管理**：内置管理员系统，包含超级管理员和动态管理员列表

- **Message Bridging**: Forward messages between users with attribution  
  **消息桥接**：在用户之间转发消息并保留归属信息

- **Forum/Topic Support**: Full compatibility with Telegram forum groups  
  **论坛/话题支持**：完全兼容 Telegram 论坛群组功能

- **Rate Limiting Protection**: Automatic retry logic for GitHub API rate limits  
  **速率限制保护**：针对 GitHub API 速率限制的自动重试逻辑

---

## Prerequisites | 前置要求

- [Node.js](https://nodejs.org/) (v16 or higher | v16 或更高版本)
- [Cloudflare account](https://dash.cloudflare.com/sign-up) (Cloudflare 账户)
- [Telegram Bot Token](https://core.telegram.org/bots#how-do-i-create-a-bot) (Telegram 机器人令牌)
- [wrangler](https://developers.cloudflare.com/workers/wrangler/) CLI tool

---

## Installation | 安装

1. **Clone the repository | 克隆仓库**:
   ```bash
   git clone <repository-url>
   cd telegram-gki-bot
   ```

2. **Install dependencies | 安装依赖**:
   ```bash
   npm install
   ```

3. **Set up your Telegram bot token as a secret | 设置 Telegram 机器人令牌为密钥**:
   ```bash
   wrangler secret put BOT_TOKEN
   ```

4. **Update `wrangler.toml` with your KV namespace ID | 在 `wrangler.toml` 中更新 KV 命名空间 ID**:
   ```toml
   [[kv_namespaces]]
   binding = "KV"
   id = "your-kv-namespace-id"
   ```

   **Or create a new KV namespace | 或创建新的 KV 命名空间**:
   ```bash
   wrangler kv:namespace create "KV"
   ```

---

## Configuration | 配置

### Environment Variables | 环境变量

| Variable | Description | Required |
|----------|-------------|----------|
| `BOT_TOKEN` | Your Telegram bot token from @BotFather | Yes |
| `BOT_TOKEN` | 从 @BotFather 获取的 Telegram 机器人令牌 | 是 |
| `KV` | Cloudflare KV namespace binding | Yes |
| `KV` | Cloudflare KV 命名空间绑定 | 是 |

### Super Admins | 超级管理员

The bot has hardcoded super admins that cannot be removed:  
机器人内置了无法移除的硬编码超级管理员：

- `mc_zihan` (ID: 7118282988)
- `coolzyd9107` (ID: 7282448230)

---

## Usage | 使用方法

### 1. Bind Bot to Telegram | 绑定 Bot 到 Telegram

After deployment, you need to add the Bot to your Telegram group or channel:  
部署完成后，您需要将 Bot 添加到您的 Telegram 群组或频道中：

1. **Open Telegram | 打开 Telegram**:  
   Search for your Bot username (e.g., `@YourBotName`).  
   搜索您创建的 Bot 用户名（例如 `@YourBotName`）。

2. **Start the Bot | 启动 Bot**:  
   Click the `/start` button to ensure the Bot is active.  
   点击 `/start` 按钮，确保 Bot 处于激活状态。

3. **Add to Group | 添加到群组**:
   - Enter the target group or channel.  
     进入目标群组或频道。
   - Click group name > **Add Members** > Select your Bot.  
     点击群组名称 > **添加成员** > 选择您的 Bot。
   - **Important | 重要**: Set the Bot as **Administrator** so it can read messages and send files.  
     将 Bot 设置为**管理员**，以便它能读取消息和发送文件。
     - **Recommended permissions | 权限建议**: Enable "Delete Messages", "Manage Topics" (if forum), "Send Files", etc.  
       开启"删除消息"、"管理话题"（如果是论坛）、"发送文件"等权限。

---

### 2. Use Bot in Chat | 在聊天中使用 Bot

The Bot supports direct use in groups, supergroups, and forums (Topics).  
Bot 支持在群组、超级群组（Supergroup）和论坛（Forum/Topics）中直接使用。

#### Basic Usage | 基础用法

- **Trigger Kernel Search | 触发内核搜索**:  
  Send the kernel version number directly in the chat, no prefix command needed.  
  直接在聊天中发送内核版本号即可，无需前缀命令。
  - **Examples | 示例**: `6.1`, `5.10.101`, `6.1.X-lts`

- **OnePlus Device Lookup | OnePlus 设备查询**:  
  Send device codename or model.  
  发送设备代号或型号。
  - **Examples | 示例**: `ACE`, `OnePlus 11`

#### Forum/Topic Mode | 论坛/话题模式

This Bot fully supports Telegram's forum feature. When Topics are enabled in a group, the Bot will automatically identify the current topic ID and reply to the corresponding topic thread without cross-talk.  
本 Bot 完美支持 Telegram 的论坛功能。当群组开启 Topics 后，Bot 会自动识别当前所在的话题 ID，并将回复发送到对应的话题线程中，不会串频。

#### Admin Features | 管理员功能

- **Add Regular Admins | 添加普通管理员**:  
  Super admins can add admins by replying with specific commands in the group (depending on code logic).  
  超级管理员可在群组中回复特定指令添加（具体指令视代码逻辑而定）。

- **Message Bridging | 消息桥接**:  
  The Bot supports anonymous forwarding of user messages, protecting privacy while facilitating communication.  
  Bot 支持匿名转发用户消息，保护隐私的同时促进交流。

---

### Development | 开发

Run the bot in development mode:  
以开发模式运行机器人：

```bash
npm run dev
```

### Deployment | 部署

Deploy to Cloudflare Workers:  
部署到 Cloudflare Workers：

```bash
npm run deploy
```

---

## Bot Commands | 机器人命令

The bot responds to various triggers and commands in Telegram groups:  
机器人在 Telegram 群组中响应各种触发器和命令：

- **Kernel Search | 内核搜索**: Request kernels by version (e.g., "6.1", "5.10.101", "6.1.X-lts")  
  按版本号请求内核（例如 "6.1"、"5.10.101"、"6.1.X-lts"）

- **Device Lookup | 设备查询**: Search for OnePlus device kernels by model  
  按型号搜索一加设备内核

- **Admin Commands | 管理员命令**: Manage admin list and permissions  
  管理管理员列表和权限

- **Message Forwarding | 消息转发**: Bridge messages between users anonymously  
  在用户之间匿名转发消息

---

## Supported Kernel Formats | 支持的内核格式

### GKI Kernels

- Standard format | 标准格式: `android12-5.10.101-2022-04-AnyKernel3.zip`
- LTS format | LTS 格式: `android14-6.1.X-lts-AnyKernel3.zip`

### OnePlus OKI Kernels

- Format | 格式: `AK3_OP-{MODEL}_{OS}_android{VERSION}_ReSukiSU_*.zip`
- Example | 示例: `AK3_OP-ACE-5-RACE_OOS16_android14-6.1.134_ReSukiSU_34681_SuSFS_v2.1.0.zip`

---

## Project Structure | 项目结构

```
telegram-gki-bot/
├── src/
│   └── index.ts          # Main bot logic | 主机器人逻辑
├── package.json          # Dependencies and scripts | 依赖和脚本
├── tsconfig.json         # TypeScript configuration | TypeScript 配置
├── wrangler.toml         # Cloudflare Workers configuration | Cloudflare Workers 配置
└── README.md             # This file | 本文件
```

---

## GitHub Repositories | GitHub 仓库

The bot fetches kernels from these repositories:  
机器人从以下仓库获取内核：

- **GKI Kernels**: [ReSukiSU-GKI/GKI_KernelSU_SUSFS](https://github.com/ReSukiSU-GKI/GKI_KernelSU_SUSFS)
- **OnePlus OKI**: [huangdihd/OnePlus_ReSukiSU_SUSFS](https://github.com/huangdihd/OnePlus_ReSukiSU_SUSFS)

---

## License | 许可证

This project is licensed under the MIT License - see the [LICENSE](LICENSE) file for details.  
本项目采用 MIT 许可证 - 详见 [LICENSE](LICENSE) 文件。

---

## Contributing | 贡献

1. Fork the repository | 叉取仓库
2. Create your feature branch | 创建你的功能分支 (`git checkout -b feature/amazing-feature`)
3. Commit your changes | 提交你的更改 (`git commit -m 'Add some amazing feature'`)
4. Push to the branch | 推送到分支 (`git push origin feature/amazing-feature`)
5. Open a Pull Request | 发起拉取请求

---

## Troubleshooting | 故障排除

### Common Issues | 常见问题

1. **GitHub API Rate Limiting | GitHub API 速率限制**:  
   The bot implements automatic retry logic. If you encounter frequent rate limits, consider adding authentication.  
   机器人实现了自动重试逻辑。如果遇到频繁的速率限制，请考虑添加身份验证。

2. **KV Namespace Errors | KV 命名空间错误**:  
   Ensure your KV namespace ID in `wrangler.toml` is correct and matches your Cloudflare account.  
   确保 `wrangler.toml` 中的 KV 命名空间 ID 正确并与您的 Cloudflare 账户匹配。

3. **Bot Not Responding | 机器人无响应**:  
   Verify that | 请确认：
   - The bot token is correctly set via `wrangler secret put BOT_TOKEN`  
     机器人令牌已通过 `wrangler secret put BOT_TOKEN` 正确设置
   - The bot is added to the group/channel with appropriate permissions  
     机器人已以适当权限添加到群组/频道
   - The Worker is deployed and active  
     Worker 已部署并处于活动状态

---

## Acknowledgments | 致谢

- [Cloudflare Workers](https://workers.cloudflare.com/) for the serverless platform
- [Telegram Bot API](https://core.telegram.org/bots/api) for the bot framework
- [ReSukiSU-GKI](https://github.com/ReSukiSU-GKI) for GKI kernels
- [huangdihd](https://github.com/huangdihd) for OnePlus OKI kernels
