# Changelog

## v3.0.4 (2026-11-12)

### 白名单换成跨境出海的长清单
- 出厂白名单从 27 个域名扩到 383 个根域名、18 类：谷歌与 YouTube、微软与必应、苹果、Meta、TikTok、X、其他社媒、亚马逊各国站、其他电商平台、独立站与建站、收款与支付、广告联盟与分析、营销客服与评价、物流、办公协作、AI 工具、资讯与接单、CDN 与基础设施。原来 27 个全保留；YouTube（youtube.com、googlevideo.com、ytimg.com、youtu.be）之前不在，装完照书打 YouTube 会被拒
- 白名单只在 `install.sh` 顶上定义一次（`DEFAULT_DOMAINS`），安装时写进 whitelist.txt、xray 路由和 `ai-xray` 的「恢复默认」；原来三处各抄一份
- 安装完成提示和 `whitelist reset` 的提示按实际个数说，不再写死「6 个平台」；文档里加白名单的例子换成 example.com（shopify.com 已在出厂名单里）

### 验证范围
- 真机（Vultr 东京 vc2-1c-1gb Debian 12 全新机，改过的脚本 scp 上去 `bash install.sh --yes`，推送之前）：52 秒装成 v3.0.4；whitelist.txt 383 行、xray 路由 383 条、`ai-xray` 里恢复默认用的列表 383 个、占位符 0 处；`whitelist add example.org` 后 384 条，`whitelist reset` 回 383 条并提示「383 个根域名」；doctor all_ok，公网 https 200
- 本机 Xray 26.11.9 客户端走隧道（同一台机）：youtube.com 200（真实页面 875KB，标题 YouTube）、youtu.be 303、x.com 200、amazon.com 202、paypal.com 302、shopify.com 200、linkedin.com 200、wikipedia.org 200、bing.com 200、tiktok.com 200、canva.com 200、dhl.com 302、chatgpt.com 200、business.facebook.com 302、ads.google.com 302；ebay.com 403（直连也 403，站点挡脚本请求）；reddit.com 走隧道 403、直连 200（Reddit 挡机房 IP，不是白名单的事）；名单外的 baidu.com、ifconfig.me 走隧道连不上、直连正常；断开 5 秒重连仍通；服务器 nginx 日志里本机 25 次连接落在 WS 路径上。机器测完销毁
- 推送后再开一台（同机型 Debian 12 全新机）照 README 那条一行安装 `--yes`：52 秒装成 v3.0.4，白名单 383 条、路由 383 条、`reset` 提示 383、doctor all_ok；本机走隧道 youtube.com 200、x.com 200、amazon.com 200、paypal.com 302、google.com 200，名单外的 baidu.com 连不上。机器测完销毁

## v3.0.3 (2026-11-11)

### 回答更到点
- AGENT.md 加一条：主人问证书/到期，用 `status --json` 的 `cert_days` 答具体天数，只说「正常」不算答（1G Debian 真机上模型答了「一切正常」却没说天数）

### 验证范围
- 真机一行安装（Vultr 东京 vc2-1c-1gb 全新机，照 README 那条 `bash <(curl …)` 在 tty 里跑）：Debian 12 1G 17/17——装机 53 秒、助手答「证书还剩 89 天」、ufw 仍开且放行 443；CentOS Stream 9 1G 18/18——装机 116 秒、助手答「证书还剩 89 天」、firewalld 仍开且放行 https、SELinux 仍 Enforcing。两台都自带 /swapfile：脚本没另建、没登记、卸载没动。CentOS 装依赖阶段 used 峰值 917MB / available 最低 33MB（总 951MB，有 swap 兜底），Debian 同阶段 312MB
- 真机一行安装 Ubuntu（同上机型）：22.04 17/17（81 秒；第一遍约 7 分钟装成但超了测试上限，重跑未复现）、24.04 17/17（85 秒）、26.04 17/17（78 秒）；三台 ufw 仍开且放行 443，自带 swap 未动

## v3.0.2 (2026-11-11)

### 对话框更干净
- picoclaw 日志级别调到 error（config `gateway.log_level`，`ai-xray chat` 再带 `PICOCLAW_LOG_LEVEL=error`）：模型调工具参数写错时的 WRN 不再刷进对话
- AGENT.md 明说调 exec 必须同时带 `action: "run"` 和 `command`（v3.0.1 真机上模型头两次漏了 action，第三次才对）

### 验证范围
- 真机（Vultr 新泽西 512M Debian 12 全新机，照 README 那条 `bash <(curl …)` 在 tty 里跑，pty 驱动按提示作答）两遍：第二遍 15/15——装机 57 秒，自动进对话框，问「服务器正常吗？证书还剩几天」3 秒答对（剩 89 天、续期已登记），对话里 0 行 WRN/ERR，exit 退出码 0；装机峰值 used 179MB；doctor 全绿、公网 https 200、`info --json` 有 vmess、`.security.yml` 600、终端记录和对话历史里 0 处钥匙样式；卸载后六个受管路径全删（第一遍产品侧 14/14 同样全过，唯一失败项是测试驱动自己的收尾竞态）

## v3.0.1 (2026-11-11)

### 修复
- 生成 WebSocket 路径用的 `tr < /dev/urandom | fold | head -n 1` 靠 SIGPIPE 收尾；在 SIGPIPE 被忽略的环境里（例如由 Python 的 pty 拉起、某些面板和自动化工具）GNU fold 写出错不检查，会对着无限输入永远不结束，安装卡在「生成配置...」。改成有限输入 `head -c 512 /dev/urandom | tr -dc | head -c N`，不再依赖信号
- 复现：512M Debian 12 上照 README 一行安装，由 pty 驱动作答，卡死 420 秒以上；`--yes --json` 经 ssh 直跑不受影响（SIGPIPE 默认）

### 验证范围
- 真机（Vultr 新泽西 512M Debian 12 全新机，照 README 那条 `bash <(curl …)` 在 tty 里跑，pty 驱动按提示作答、像真终端一样应答光标查询）：14/14——装机 57 秒，自动进对话框，问「服务器正常吗？证书还剩几天」6 秒答对（剩 89 天、续期已登记），exit 退出码 0；装机峰值 used 171MB；doctor 全绿、公网 https 200、`info --json` 有 vmess、`.security.yml` 600、终端记录和对话历史里 0 处钥匙样式；卸载后六个受管路径全删
- 已知瑕疵：回答前 picoclaw 在终端打了两行 `WRN … missing required property "action" tool=exec`（模型头两次调 exec 漏了 action），答案不受影响
- picoclaw 交互模式用的 readline 画提示符前要终端应答 `ESC[6n`，不答的终端（某些面板网页控制台）提示符不出现；`ai-xray chat -m` 不受影响

## v3.0.0 (2026-11-11)

### AI 助手：出了问题一句话修
- 装机时可贴一把 DeepSeek 钥匙（`read -s` 收，不进命令行、不进对话），装完直接进对话框；以后 `ai-xray chat`
- 底座是 [picoclaw](https://github.com/sipeed/picoclaw) v0.3.1（钉版 + sha256 校验，x86_64 / arm64），放在 `/etc/ai-xray/assistant/`；钥匙单独存 `.security.yml`（600）
- 三道闸把模型限在 `ai-xray` 上：配置里只留 exec + read_file、远程通道关；`AGENT.md` 的 `tools:` 白名单；`approve_tool` 进程钩子 `gate.py` 做命令级白名单（fail-closed），只放行 `ai-xray <子命令>` 和 `free -m`、`df -h`、`uptime`，分号/管道/换行/后台一律拒
- 运维剧本 `skills/ai-xray/SKILL.md`（I-Lang，过校验器）：查 → 修 → 确认 → 用人话汇报；清空白名单、卸载要用户说「确定」
- `ai-xray assistant setup|remove` 补装或移除

### 512M 内存向下兼容
- 装前检测：内存 <700MB 且系统是 CentOS / Ubuntu 22.04+ 这类重系统 → 停下来叫用户重装 Debian 12（`--force-os` 可硬装）
- 内存 <1G 且无 swap → 自动建 512MB swapfile，登记进 fstab 和 managedPaths，卸载时撤掉

### 子命令（给助手和自动化用）
- `doctor [--json]`：11 项体检，每项 ok/false 带提示
- `fix`：重建 nginx / xray 配置（参数从 info.json 来，新增 `xrayPort`）、放行防火墙、补续期任务、续签证书、重启服务，最后再体检一遍
- `status --json`、`info --json`、`logs [n]`、`restart`、`whitelist list|add|remove|reset|clear --yes`、`chat`、`assistant`、`uninstall --yes`
- 菜单加 `10) 体检并自动修复`、`11) 跟 AI 助手说话`

### 安装脚本参数
- `--domain`、`--site`、`--yes`、`--json`、`--no-ai`、`--no-swap`、`--force-os`、`--help`；环境变量 `AI_XRAY_KEY`
- 没有域名直接回车：用 `<ip>.sslip.io` 签证书（Let's Encrypt 实测签得出）
- `--json` 模式 stdout 只有一段 JSON

### 验证范围
- 本地：bash -n、shellcheck 0 warning；gate.py 11 条放行/拒绝用例；SKILL.md 过 I-Lang 校验器 0 错 0 警告；管理脚本本机功能测试 32 项
- 真机（Vultr 新泽西 512M Debian 12，钥匙经 stdin，`--yes --json`，两遍，第二遍 18/18）：装机 60 秒、峰值 used 159–176MB、available 最低 274MB；`--json` 模式 stdout 只有 JSON；助手文件/闸/配置核对全过；对话「服务器正常吗」3 秒答对；故意停 nginx、删续期任务、写坏 xray 配置、停 xray 后，一句「手机连不上了」10 秒内全修回来（doctor 全绿）；对话加白名单生效；让它 `rm -rf` 被拒且文件还在；要连接信息给出 vmess；每轮 RSS 峰值 32–34MB；历史文件里无钥匙；卸载后含助手、swap、picoclaw 的受管路径全删
- 未做：1G 以上机器和 CentOS 9 没重跑 v3.0；arm64 包只校验了哈希没装过

## v2.2.0 (2026-11-11)

### 证书续期（真正修好）
- 之前 acme.sh 只是被下载到 /usr/local/bin，从没跑过 `--install`，而 acme.sh 的续期定时任务只有 `--install` 才会装：装出来的机器约 90 天后证书到期、443 失效。现在安装时写入 `/etc/cron.d/ai-xray-acme`，每天 03:23 跑 `acme.sh --cron`，续期沿用 v2.1.1 的 pre/post-hook 和 reloadcmd
- 证书改装到 `/etc/nginx/ssl/ai-xray/`（SELinux 开着也能读）；管理菜单新增 `9) 手动续期证书`；`查看状态` 显示证书到期日和自动续期是否登记

### 站点层重做
- 移除对 `aixray.fluxrouter.net/generate` 和模板池仓库 `ai-xray-sites` 的依赖（前者模型已停、返回 503，后者不存在，结果是每台机都显示同一张 "Site Maintenance" 页）。改为三套随脚本内置的静态模板（文档站、加密工具站、学生福利导航站），安装时随机选标题和构建号本地渲染，不再联网

### 防火墙与 SELinux
- 不再关闭 firewalld / 禁用 SELinux（v2.1.0 的 CHANGELOG 这么写了，代码没做到）。改为：firewalld 或 ufw 开着就只放行 80/443；SELinux 保持开启，只打开 `httpd_can_network_connect` 让 nginx 连本机 Xray 端口

### 白名单管理
- 路由重建从 python3 改为 jq（python3 不在依赖里，CentOS 最小安装没有）
- 添加域名先清洗（去协议、路径、端口、www，转小写）并校验格式；新配置先写临时文件、`xray run -test` 通过才替换并重启，失败则撤回这次添加，原配置不动
- 删除编号校验为数字

### 其他
- 版本号统一为 2.2.0（原脚本五处写 v2.0、info.json 写 2.0.0）
- 死链 `ssr.dedyn.io` 移除，客户端导入说明改指 README
- 卸载补删 BBR 的 sysctl 片段、续期定时任务、模板目录、证书目录；明确不删 Xray 内核、nginx、acme.sh
- nginx 反代加 `proxy_read_timeout` / `proxy_send_timeout` 300s；有 IPv6 时同时监听 `[::]:443`
- 证书申请失败时能打出原因提示（原来 `set -e` 直接退出）
- apt 锁最多等 120 秒
- `ai-xray` 支持子命令：`render` `renew` `status` `info` `version`
- `info.json` 权限改为 600
- shellcheck：0 error 0 warning（v2.1.1 为 1 error 9 warning）

### 验证范围
- 本地：bash -n、shellcheck 0.11.0；管理脚本的模板渲染、jq 路由重建 + `xray run -test`（Xray 26.11.9）、域名清洗、证书到期计算、子命令分发
- 真机（Vultr 东京 vc2-1c-1gb，域名用 sslip.io，证书 Let's Encrypt 真签）：Debian 12 全过 27 项、CentOS Stream 9（SELinux Enforcing、firewalld 开启）全过 29 项。项目：安装脚本退出 0；xray/nginx 运行；证书在 `/etc/nginx/ssl/ai-xray/`；续期任务登记；`ai-xray status` 显示到期日；首页有随机标题；公网能打开站点、80 跳 443；本机客户端经 VMess 实连：x.com、google.com 通，白名单外 example.com 被挡；菜单加白后 example.com 通、带协议路径的输入清洗成 shopify.com、坏域名被拒；`ai-xray renew` 成功且 nginx 仍在；卸载后六个受管路径全删
- 未做：CentOS 7/8、Debian 10/11、Ubuntu 各版本没有重新跑

## v2.1.1 (2026-06-24)

### 路由修复
- inbound 启用 sniffing（destOverride http/tls/quic，routeOnly），routing domainStrategy 改 IPIfNonMatch：修复客户端发送 IP（本地解析 DNS）时白名单规则匹配不到、流量被误 block 的问题
- 默认白名单按平台补全关联域名（CDN/接口/网页域名，如 chatgpt.com、gstatic.com、googleapis.com、fbcdn.net 等）：修复白名单内平台的资源/接口被挡、页面打不全的问题

### 证书续期修复
- acme.sh 增加 --pre-hook/--post-hook 停启 nginx：修复无人值守续期时 standalone 无法占用 80 端口（nginx 占用）、证书约 90 天后到期失效的问题

### 清理
- 移除 rebuild_routing 中无效的 killall -SIGHUP xray fallback（Xray 不支持 SIGHUP 热重载）
- 默认白名单移除 claude.ai（共 6 个平台）

## v2.1.0 (2026-05-08)

### 安全加固（17项审计修复）
- Xray-install: pin commit + SHA256 校验
- acme.sh: pin 版本 + SHA256 校验，下载脚本不管道执行
- 依赖安装: 逐包安装，失败即退出，保留 stderr
- 端口检测: ss -ntlp 探测 + sleep 2 + shuf 随机端口循环验证空闲
- BBR: 写入 /etc/sysctl.d/99-bbr.conf drop-in，不再直接修改 sysctl.conf
- 卸载: 读取 info.json managedPaths，仅删除脚本创建的文件
- 证书续期: --reloadcmd 自动 reload nginx
- 新增 test_configs: nginx -t + xray run -test 部署前验证
- IP 获取: 多源 fallback
- Worker 响应校验
- tar 解压错误处理
- nginx repo: https + gpgcheck
- preflight 预检 + 配置备份机制
- 全局防火墙/SELinux 不关闭（仅处理阻止通信的规则）

### 白名单 + TOS
- 出厂默认 7 个跨境电商/AI 域名白名单
- Xray routing 白名单外流量全部 block
- TOS.txt 条款，删除域名需阅读全文并输入 YES 确认
- 管理菜单新增「白名单管理」选项（添加/删除/全部删除/恢复默认）

### 兼容性修复
- CentOS 7: BBR 检测从 grep bbr 改为 tcp_available_congestion_control
- CentOS 8: 补充 tar 依赖
- set -e 下 ping / grep / curl / systemctl 容错处理
- nginx 配置统一使用 conf.d，避免 sites-enabled 双重加载
- VMess 链接 add 字段从 IP 改为域名
- BBR 重复写入检测

### 文字修正
- 「伪装站」→「站点」
- README 定位重写为「跨境电商 & AI 生产力加速器」，移除敏感词
- CHANGELOG / NOTICE / install.sh 中 camouflage 术语替换

### 已验证系统
- CentOS 7 / 8 / Stream 9
- Debian 10 / 11 / 12
- Ubuntu 18.04 / 20.04 / 22.04 / 24.04

## v2.0.0 (2026-05-08)

### Rewritten from scratch
- Single `install.sh` replacing all previous scripts
- VMESS + WS + TLS only (removed Reality, VLESS, Trojan, XTLS, KCP, gRPC)
- AI-powered site generation with 3-layer fallback:
  - Level 1: Real-time AI generation via Cloudflare Worker
  - Level 2: Local rendering from open-source template pool
  - Level 3: Temporary redirect
- Simplified VPS quality check (3 tests instead of 8)
- Single-file management command `ai-xray`
- Clean repo structure: 7 files total

### Removed
- All multi-protocol support
- `install-simple.sh`, `install-full.sh`
- Legacy README variants
- `_worker.js`, `wrangler.toml`
- All test reports and work logs

### Reference
- `docs/reference/xray-v1-reference.sh` retained as historical reference
