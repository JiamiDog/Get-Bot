# Get Bot

跨境电商 & AI 生产力加速器

为跨境从业者解决海外平台访问慢、不稳定的问题。出厂预置 Google Ads、Meta、TikTok、Pinterest 等业务平台白名单，一行命令部署，开箱即用。

## 解决什么问题

跨境电商从业者每天要用 Google Ads 投广告、Meta Business 管主页、TikTok Ads 跑素材、Pinterest 做选品调研。这些平台从国内访问慢、掉线、打不开，严重影响工作效率。

Get Bot 在你的海外服务器上部署一条加速通道，专门加速这些业务平台。默认只加速白名单内的平台，不是通用网络工具。

## 一行安装

```bash
bash <(curl -fsSL https://raw.githubusercontent.com/JiamiDog/Get-Bot/main/install.sh)
```

回车用 IP 签证书（有域名就填域名）→ 贴一把 DeepSeek 钥匙（可跳过）→ 一分钟装好 → 复制连接信息到客户端 → 开始工作。装好以后直接进对话框，出了问题跟助手说一句。

不想回答问题：`bash install.sh --yes`（域名用 `<ip>.sslip.io`，不装助手）；给自动化用：`--json` 只输出一段 JSON，钥匙走环境变量 `AI_XRAY_KEY`。全部选项见 `bash install.sh --help`。

## AI 助手

装机时贴一把你自己的 DeepSeek 钥匙（[platform.deepseek.com](https://platform.deepseek.com) 注册，充 10 块，复制 API key），服务器上就多一个助手。以后 ssh 上来输 `ai-xray chat`，用人话说：

- 「手机连不上了，帮我看看」→ 它跑体检、修、再确认，告诉你结果
- 「把 shopify.com 加进白名单」
- 「把连接信息发我」
- 「证书还剩几天」

它底下是 [picoclaw](https://github.com/sipeed/picoclaw)（Go 单二进制，一轮对话约 30MB 内存），**只能**通过一道命令闸运行 `ai-xray` 的子命令和 `free -m`、`df -h`、`uptime`，别的命令一律拒绝；清空白名单和卸载要你明确说「确定」。钥匙存在 `/etc/ai-xray/assistant/.security.yml`（600），不进对话。没装或装机时跳过了：`ai-xray assistant setup`；不要了：`ai-xray assistant remove`。

## 512M 内存也能跑

装前检测：内存不到 700MB 又是 CentOS / Ubuntu 22.04+ 这类重系统，脚本会停下来叫你去面板重装 Debian 12（硬要装加 `--force-os`）；内存不到 1G 且没有 swap，自动建 512MB swap。实测 512M 的 Debian 12：装机峰值约 160–270MB，装完常驻约 120MB，助手一轮对话 32–34MB。

## 默认业务白名单

安装完成后，仅以下业务平台可通过加速通道访问：

| 平台 | 域名 | 业务场景 |
|------|------|----------|
| Google | google.com | Ads投放、Analytics分析、Search Console、Merchant Center |
| Meta | facebook.com | Business Suite、广告投放、主页管理 |
| TikTok | tiktok.com | Ads Manager、店铺管理、素材投放 |
| X (Twitter) | x.com | 广告投放、社媒运营 |
| Pinterest | pinterest.com | 广告投放、选品调研 |
| OpenAI | openai.com | ChatGPT |

白名单含各平台及其 CDN/接口域名。白名单外的流量不经过加速通道，可通过管理菜单添加其他业务平台。

## 特性

**业务专用。** 默认白名单锁定跨境电商和AI生产力平台。这是一台专用的工作加速器，不是通用网络工具。

**协议稳定。** VMESS + WebSocket + TLS，经过Cloudflare CDN实战验证的组合，稳定性优先。

**内置站点模板。** 三套静态模板随脚本内置（文档站、加密工具站、学生福利导航站），安装时随机标题和构建号本地渲染，不依赖任何外部服务。

**证书自动续期。** 安装时登记每日续期任务；`ai-xray` 里能看到证书到期日，也能手动续期。

**AI 助手。** 贴一把 DeepSeek 钥匙，出了问题跟它说一句，它查、它修、它回话；命令闸限住它只能动 `ai-xray`。

**多发行版，512M 起。** CentOS Stream 9、Debian 10/11/12、Ubuntu 18.04~24.04；小内存机器装前检测、自动 swap。

## 系统兼容性

| 发行版 | 版本 | 状态 | 备注 |
|--------|------|------|------|
| CentOS | 7 | ⚠️ | 系统已停服，默认软件源不可用，需先指向 vault 源；BBR自动跳过（内核3.10） |
| CentOS | 8 | ⚠️ | 系统已停服，默认软件源不可用，需先指向 vault 源 |
| CentOS | Stream 9 | ✅ | |
| Debian | 10 | ✅ | |
| Debian | 11 | ✅ | |
| Debian | 12 | ✅ | |
| Ubuntu | 18.04 | ✅ | |
| Ubuntu | 20.04 | ✅ | |
| Ubuntu | 22.04 | ✅ | |
| Ubuntu | 24.04 | ✅ | |
| Ubuntu | 26.04 | ✅ | |

v3.0.3 用上面那条一行安装在 Debian 12（512M 和 1G）、CentOS Stream 9（SELinux Enforcing、firewalld 开启）、Ubuntu 22.04 / 24.04 / 26.04 上跑过完整安装、公网证书签发、助手对话和卸载（v2.2.0 另做过客户端实连）；Debian 10/11、Ubuntu 18.04/20.04 的 ✅ 是 v2.1.0 时的结果，之后没有重跑。

## 客户端

安装完成后，将连接信息导入客户端即可使用：

| 平台 | 推荐客户端 |
|------|------------|
| iOS | Shadowrocket |
| Android | v2rayNG |
| Windows | v2rayN |
| macOS | V2rayU |

导入步骤：

1. 在服务器上运行 `ai-xray`，选 `1` 查看连接信息，复制 `vmess://` 开头的那一整行。
2. 打开客户端，选「从剪贴板导入」（v2rayN：服务器 → 从剪贴板导入批量URL；v2rayNG：右上角 + → 从剪贴板导入；Shadowrocket：右上角 + → 粘贴）。
3. 选中这个节点，打开连接，访问白名单里的平台（比如 x.com）确认能通。

## Cloudflare CDN（推荐）

域名DNS管理中开启橙色云朵（Proxied），SSL/TLS模式选Full (strict)。通过Cloudflare全球网络加速业务访问，提升稳定性。

## 管理

服务器上的命令和目录名仍叫 `ai-xray`。

```bash
ai-xray
```

| 选项 | 功能 |
|------|------|
| 1 | 查看连接信息 |
| 2 | 重新生成站点 |
| 3 | 更新Xray内核 |
| 4 | 重启服务 |
| 5 | 查看日志 |
| 6 | 查看状态 |
| 7 | 白名单管理 |
| 8 | 卸载 |
| 9 | 手动续期证书 |
| 10 | 体检并自动修复 |
| 11 | 跟 AI 助手说话 |

子命令（助手和自动化用的就是这些）：

| 命令 | 做什么 |
|------|--------|
| `ai-xray status [--json]` | 服务、端口、证书剩余天数、续期任务、助手 |
| `ai-xray doctor [--json]` | 体检 11 项，每项 ok/false 带提示 |
| `ai-xray fix` | 按顺序自动修：重建坏配置、放行防火墙、补续期任务、续签证书、重启服务 |
| `ai-xray info [--json]` | 连接信息（vmess 链接） |
| `ai-xray whitelist list / add 域名 / remove 域名 / reset / clear --yes` | 白名单 |
| `ai-xray logs [行数]`、`renew`、`update`、`restart` | 日志、续期、更新内核、重启 |
| `ai-xray chat`、`ai-xray assistant setup / remove` | AI 助手 |
| `ai-xray uninstall [--yes]` | 卸载，只删自己建的东西 |

证书自动续期任务登记在 `/etc/cron.d/ai-xray-acme`，每天 03:23 检查。

## 推荐VPS

| 商家 | 特点 | 链接 |
|------|------|------|
| 搬瓦工 DC8ZNET | CN2 GIA，中国直连最快 | [bandwagonhost.cloud](https://bandwagonhost.cloud) |
| DMIT | 香港/洛杉矶，大带宽 | [dmit.io](https://www.dmit.io/aff.php?aff=3138) |
| Vultr | 按小时计费，随开随关 | [vultr.com](https://www.vultr.com/?ref=8910412) |

## 免责声明

Get Bot 是为跨境电商从业者设计的业务平台加速工具。默认配置仅加速预置白名单内的业务平台。用户对自行修改白名单后的所有网络行为承担全部法律责任。开发者不对任何超出原始业务加速用途的使用承担责任。

## 致谢

基于 [Xray-core](https://github.com/XTLS/Xray-core) / [acme.sh](https://github.com/acmesh-official/acme.sh) / [Nginx](https://nginx.org/) / [Let's Encrypt](https://letsencrypt.org/) 构建。

## 许可证

MIT © JiamiDog
