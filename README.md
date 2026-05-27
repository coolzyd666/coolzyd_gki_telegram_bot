# Telegram GKI Bot

A Telegram bot for GKI (Generic Kernel Image) and OnePlus OKI (OnePlus Kernel Image) kernel downloads, deployed on Cloudflare Workers.

## Features

- **GKI Kernel Downloads**: Fetch and download GKI kernels from GitHub releases
- **OnePlus OKI Support**: Specialized support for OnePlus device kernels
- **Version Matching**: Intelligent kernel version matching with LTS support
- **Admin Management**: Built-in admin system with super admins and dynamic admin list
- **Message Bridging**: Forward messages between users with attribution
- **Forum/Topic Support**: Full compatibility with Telegram forum groups
- **Rate Limiting Protection**: Automatic retry logic for GitHub API rate limits

## Prerequisites

- [Node.js](https://nodejs.org/) (v16 or higher)
- [Cloudflare account](https://dash.cloudflare.com/sign-up)
- [Telegram Bot Token](https://core.telegram.org/bots#how-do-i-create-a-bot)
- [wrangler](https://developers.cloudflare.com/workers/wrangler/) CLI tool

## Installation

1. Clone the repository:
   ```bash
   git clone <repository-url>
   cd telegram-gki-bot
   ```

2. Install dependencies:
   ```bash
   npm install
   ```

3. Set up your Telegram bot token as a secret:
   ```bash
   wrangler secret put BOT_TOKEN
   ```

4. Update `wrangler.toml` with your KV namespace ID:
   ```toml
   [[kv_namespaces]]
   binding = "KV"
   id = "your-kv-namespace-id"
   ```

   Or create a new KV namespace:
   ```bash
   wrangler kv:namespace create "KV"
   ```

## Configuration

### Environment Variables

| Variable | Description | Required |
|----------|-------------|----------|
| `BOT_TOKEN` | Your Telegram bot token from @BotFather | Yes |
| `KV` | Cloudflare KV namespace binding | Yes |

### Super Admins

The bot has hardcoded super admins that cannot be removed:
- `mc_zihan` (ID: 7118282988)
- `coolzyd9107` (ID: 7282448230)

## Usage

### Development

Run the bot in development mode:
```bash
npm run dev
```

### Deployment

Deploy to Cloudflare Workers:
```bash
npm run deploy
```

## Bot Commands

The bot responds to various triggers and commands in Telegram groups:

- **Kernel Search**: Request kernels by version (e.g., "6.1", "5.10.101", "6.1.X-lts")
- **Device Lookup**: Search for OnePlus device kernels by model
- **Admin Commands**: Manage admin list and permissions
- **Message Forwarding**: Bridge messages between users anonymously

## Supported Kernel Formats

### GKI Kernels
- Standard format: `android12-5.10.101-2022-04-AnyKernel3.zip`
- LTS format: `android14-6.1.X-lts-AnyKernel3.zip`

### OnePlus OKI Kernels
- Format: `AK3_OP-{MODEL}_{OS}_android{VERSION}_ReSukiSU_*.zip`
- Example: `AK3_OP-ACE-5-RACE_OOS16_android14-6.1.134_ReSukiSU_34681_SuSFS_v2.1.0.zip`

## Project Structure

```
telegram-gki-bot/
├── src/
│   └── index.ts          # Main bot logic
├── package.json          # Dependencies and scripts
├── tsconfig.json         # TypeScript configuration
├── wrangler.toml         # Cloudflare Workers configuration
└── README.md             # This file
```

## GitHub Repositories

The bot fetches kernels from these repositories:

- **GKI Kernels**: [ReSukiSU-GKI/GKI_KernelSU_SUSFS](https://github.com/ReSukiSU-GKI/GKI_KernelSU_SUSFS)
- **OnePlus OKI**: [huangdihd/OnePlus_ReSukiSU_SUSFS](https://github.com/huangdihd/OnePlus_ReSukiSU_SUSFS)

## License

This project is licensed under the MIT License - see the [LICENSE](LICENSE) file for details.

## Contributing

1. Fork the repository
2. Create your feature branch (`git checkout -b feature/amazing-feature`)
3. Commit your changes (`git commit -m 'Add some amazing feature'`)
4. Push to the branch (`git push origin feature/amazing-feature`)
5. Open a Pull Request

## Troubleshooting

### Common Issues

1. **GitHub API Rate Limiting**: The bot implements automatic retry logic. If you encounter frequent rate limits, consider adding authentication.

2. **KV Namespace Errors**: Ensure your KV namespace ID in `wrangler.toml` is correct and matches your Cloudflare account.

3. **Bot Not Responding**: Verify that:
   - The bot token is correctly set via `wrangler secret put BOT_TOKEN`
   - The bot is added to the group/channel with appropriate permissions
   - The Worker is deployed and active

## Acknowledgments

- [Cloudflare Workers](https://workers.cloudflare.com/) for the serverless platform
- [Telegram Bot API](https://core.telegram.org/bots/api) for the bot framework
- [ReSukiSU-GKI](https://github.com/ReSukiSU-GKI) for GKI kernels
- [huangdihd](https://github.com/huangdihd) for OnePlus OKI kernels
