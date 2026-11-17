> 如需Clash版本 → [请点击此处](https://github.com/LingJingMaster/Clash-Rules)

---

# 小火箭 Shadowrocket Config

一份开箱即用的 小火箭 Shadowrocket 规则配置
- 支持前沿的苹果智能（Apple Intelligence）
- 在新规下的香港券商访问
- 导入后添加自己的节点或订阅即可使用

## 个人定制

本 Fork 基于 [LingJingMaster/Shadowrocket-Rules](https://github.com/LingJingMaster/Shadowrocket-Rules)，保留原作者 MIT LICENSE 与规则体系。

- 新增独立的 `Muse_Meta.list`，覆盖 Muse 网页、VM 网关、Meta 账号及有代码证据的资源域名。证据分级、可选域名、副作用和测试缺口见 [Muse 网络审计](docs/Muse-Network-Audit.md)。本版本不声称已覆盖所有功能。
- `🎬 Muse / Meta` 默认使用现有 `🇯🇵 日本节点`，可手动切换为 `🚀 节点选择`、`PROXY` 或 `DIRECT`。请确保日本节点组内有可用节点。
- `auth.meta.com` 已被 `DOMAIN-SUFFIX,meta.com` 覆盖，无需重复添加。Facebook / Instagram 仅纳入有认证代码证据的精确主机；这仍会影响访问同一主机的其他应用。
- Shadowrocket → 配置 → `+` → 粘贴 `https://raw.githubusercontent.com/jkl00/Shadowrocket-Rules/main/Shadowrocket.conf` → 下载并启用，再添加自己的节点或订阅。下方二维码也指向此个人版地址。
- 配置自身的 `update-url` 和个人规则指向本 Fork；Google、AI、邮件、银行、券商等远程规则继续引用上游。
- 以后在本仓库 GitHub 首页选择 **Sync fork → Update branch** 同步上游；如有冲突，不要丢弃个人提交。重点保留 `Shadowrocket.conf` 中个人 `update-url`、策略组和 RULE-SET 插入位置，以及此章节、`Muse_Meta.list`、审计文档。个人 Muse 规则插在 HTTPDNS 之后、邮件之前；同步时也请保留个人版二维码 `assets/personal-config-qr.png` 及其引用。
- 本地同步可用 `git fetch upstream`、`git switch main`、`git merge upstream/main`，解决冲突并复核上述入口后再 `git push origin main`；首次克隆 Fork 时用 `git remote add upstream https://github.com/LingJingMaster/Shadowrocket-Rules.git` 建立关联。不要强推或以重置方式丢弃个人修改。

## 快速开始

1. 复制配置文件的 Raw 链接：
   `https://raw.githubusercontent.com/jkl00/Shadowrocket-Rules/main/Shadowrocket.conf`
2. 打开 Shadowrocket → 配置 → 右上角 `+` → 粘贴链接 → 下载
3. 点击已下载的配置，设为使用中（✔️）
4. 首页添加你自己的节点或订阅
5. 连通性测试，选择可用节点连接

或者在 Shadowrocket 中扫描下方**个人版配置二维码**（指向 `jkl00`，不是上游）：

<img width="260" height="260" alt="jkl00 的 Shadowrocket 个人版配置二维码" src="assets/personal-config-qr.png" />

导入后请设为当前使用的配置，并在策略组中确认存在 `🎬 Muse / Meta`。规则列表中它是一条引用 `Muse_Meta.list` 的 `RULE-SET`，远程文件里的域名不一定逐条展开；编辑配置源码可搜索 `Muse_Meta.list` 和 `🎬 Muse / Meta`。如果之前扫过旧二维码，请用上面的个人版 URL 重新下载，单独更新原版配置仍会获取上游版本。

## 分流规则

| 优先级 | 服务 | 默认策略 |
|--------|------|----------|
| 1 | 🧱 DNS 防泄露（HTTPDNS） | REJECT |
| 2 | 🎬 Muse / Meta | 日本节点，可手动切换 |
| 3 | 📧 邮件服务（IMAP / POP3 / SMTP） | PROXY，可切换 DIRECT 或地区节点 |
| 4 | 🔍 谷歌服务（含 Gemini） | 日本节点，可手动切香港节点 |
| 5 | 🤖 AI 服务（ChatGPT、Claude、苹果智能等） | 美国节点 |
| 6 | 📹 油管视频（含 YouTube 翻译 API） | 节点选择 |
| 7 | 🔒 哔哩哔哩 | DIRECT |
| 8 | 🏠 私有网络 / 局域网 | DIRECT |
| 9 | 📲 电报消息 | 节点选择 |
| 10 | 🐱 代码托管（GitHub、GitLab、Atlassian） | 节点选择 |
| 11 | Ⓜ️ 微软服务 | 节点选择 |
| 12 | 🏦 汇丰香港（含 Reward+） | DIRECT |
| 13 | 🏦 其他香港银行 | DIRECT |
| 14 | 📈 券商服务（富途 / moomoo / 长桥 / 老虎 / 雪盈 / 盈透） | 香港节点 |
| 15 | 🍎 苹果推送 | 节点选择 |
| 16 | 🍏 苹果服务 | DIRECT |
| 17 | 🔒 国内服务 | DIRECT |
| 18 | 🌍 非中国（境外流量） | PROXY |
| 19 | GEOIP CN | DIRECT |
| 20 | 🐟 漏网之鱼（兜底） | PROXY |


## 默认策略

| 服务 | 默认策略 | 可选策略 |
|------|----------|----------|
| 🧱 DNS 防泄露 | REJECT | 节点选择、DIRECT |
| 📧 邮件服务 | PROXY | DIRECT、节点选择、日本节点、香港节点 |
| 🔍 谷歌服务 | 🇯🇵 日本节点 | 🇭🇰 香港节点、节点选择、PROXY、DIRECT |
| 🎬 Muse / Meta | 🇯🇵 日本节点 | 节点选择、PROXY、DIRECT |
| 🤖 AI 服务 | 🇺🇸 美国节点 | 节点选择、PROXY、DIRECT |
| 🍎 苹果推送 | 🚀 节点选择 | PROXY、DIRECT |
| 🍏 苹果服务 | DIRECT | 节点选择、PROXY |
| 🏦 汇丰香港 | DIRECT | 🇭🇰 香港节点、节点选择、PROXY |
| 🏦 香港银行 | DIRECT | 香港节点、节点选择、PROXY |
| 📈 券商服务 | 🇭🇰 香港节点 | DIRECT、节点选择、PROXY |
| 🌍 非中国 | PROXY | 节点选择、DIRECT、日本节点 |
| 🐟 漏网之鱼 | PROXY | 节点选择、DIRECT、日本节点 |







## 规则集来源

- [blackmatrix7/ios_rule_script](https://github.com/blackmatrix7/ios_rule_script) — 主要规则集
- [iab0x00/ProxyRules](https://github.com/iab0x00/ProxyRules) — AI 服务补充规则
- `Mail.list` 收录 Apple、Gmail、Outlook、Yahoo、Yandex 的邮件协议端点
- `Apple.list` 基于 blackmatrix7 Apple 规则，并配套加载 `Apple_Domain.list`，补充 iCloud Photos / Apple CDN 直连域名
- `HK_Broker.list` 补充富途 / moomoo / 长桥 / 老虎 / 雪盈 / 盈透 / TradeUP / Schwab 证券域名及交易 IP 段
- `HSBC_HK.list` 与 `HK_Banks_Direct.list` 收录香港银行网站及 App 服务域名

## Star History

[![Star History Chart](https://api.star-history.com/chart?repos=lingjingmaster/shadowrocket-rules&type=date&legend=top-left)](https://www.star-history.com/?repos=lingjingmaster%2Fshadowrocket-rules&type=date&legend=top-left)

## License

MIT
