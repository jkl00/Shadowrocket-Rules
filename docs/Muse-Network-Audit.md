# Muse 网络依赖审计

审计日期：2026-11-17（Asia/Shanghai）。基线：上游 `091d94e826ae2973bc0884aec53b97ec94949bac`。
本文件记录当前公开材料能够支持的规则，**不是 Muse 全功能连通性证明**。

## 范围与输入缺口

用户补充要求提到“20 条扩展规则”，但消息没有逐行附出这 20 条原文。因此下文将明确点名的域名和所描述的 GraphQL、Facebook 认证、OHTTP 端点作为核查起点，不声称复现了未提供的原始 20 行。最终启用规则由证据与最小分流原则决定，不以凑足 20 条为目标。

A = 已确认的核心服务依赖（服务代码/官方 SDK 或公开网页直接资源支持；不等于本次登录实测）。
B = 有当前服务代码证据的候选或条件依赖（默认启用的有限补充，尚未实测功能）。
C = 仅 CSP、域名清单、通用辅助代码或第三方资料可见，缺乏 Muse 实际调用证据（可选，默认不启用）。

## 证据来源与交叉核查

| ID | 来源 | 本次核查范围与限制 |
|---|---|---|
| S1 | [Muse 官网](https://muse.ai/) | HTTPS GET 返回 200，响应标记 `/oidc/logged-out`；检查当前 HTML、CSP、49 个直接脚本资源。不是已登录会话，也未捕获完整运行期 HAR。 |
| S2 | [官方 Muse Gadget SDK](https://github.com/facebookincubator/muse-gadget-sdk/tree/481878879e5c22cd8af9726d7576e1d162884e6a) | 版本 `481878879e5c22cd8af9726d7576e1d162884e6a`；`linux/src/musegadget/muse_api.py` 的 `API_BASE`、`service.py` 的 `DEFAULT_NOISE_HOST` 以及 `link_client.py` 的 WebSocket 构造代码。 |
| S3 | [nikships/muse-cli PROTOCOL.md](https://github.com/nikships/muse-cli/blob/b62adbabb77abf688d26a74f93d405b6667dbc9f/docs/PROTOCOL.md) | 版本 `b62adbabb77abf688d26a74f93d405b6667dbc9f`；作者记录的是其自身实测，不是本次实测。网关和 API 路径与 S2/S1 交叉验证；文件上传、生成媒体、VM 下载路径作为补充证据。 |
| S4 | [MetaCubeX Meta 域名集](https://raw.githubusercontent.com/MetaCubeX/meta-rules-dat/meta/geo/geosite/meta.list) | 核查 `muse.ai`、`metaaivm.com`、`metaaiusercontent.com`、`meta.ai`、`meta.com`、`llama.com`、`atmeta.com` 等归类。此文件还含大量其他 Meta 服务/商标域名，未整表导入。SHA-256：`5f6cdf31b559fa8c07b9386dcf6329d8d4e44e0f6580807e70c7131790c09b5a`。 |
| S5 | [Meta 账号登录公开页](https://auth.meta.com/) | GET 返回公开 HTML，图标和脚本指向 `static.xx.fbcdn.net`；检查直接公开脚本。裸 `/oidc` 返回 404，不据此判断真实 OAuth 登录失效。 |
| S6 | [Meta 官方 Muse 介绍](https://about.fb.com/news/2026/09/introducing-muse-personal-ai-agent/) | 说明个人 VM、云端浏览器、连接第三方服务与 Stripe Link 购物能力；不提供固定完整域名表，不能据此推定每个第三方请求都从 iPhone 发出。 |

S1 的关键公开脚本（文件名可能随发布变化）：

- `3cmpldvl_h-8p.js`：`.metaaivm.com` VM 地址处理；生产域 `muse.ai` / `production.museai.com`；`voice.museai.com`；亦含旧域/开发环境和允许列表，不能把全部字符串视为实际请求。
- `169rnupgd87zi.js`：明确处理 `.s.metaaiusercontent.com`、`.s.ecto1usercontent.com` 应用预览来源。
- `0fw1h171o30gr.js`、`2_plgp0oeq6g1.js`：`auth.meta.com/oidc`，接受 `www.facebook.com` / `www.instagram.com` 的 `/auth_platform` 认证重定向。
- `2f2xzv4_d_jsf.js`：`https://nest-static.fbcdn.net/v1/hatch-web/` 资源加载路径。
- `241xaz9jd44ky.js`：`api.muse.ai/patrique...` API 路径，生产 origin 和账号辅助链接。
- `111hi1ey48q2j.js`：Stripe 付款 iframe 识别；不是支付网络成功证据。
- `01uwshq62l429.js`：Facebook Pixel / Vercel 测量代码。未因为测量代码出现就将它们纳入核心分流。

这些公开脚本可由 `https://muse.ai/_next/static/immutable/chunks/<文件名>` 查看。只记录短域名/用途和摘要哈希，不在仓库复制完整第三方脚本、账号信息、Cookie、签名 URL、HAR 或 Token。

## 最终启用规则（与 Muse_Meta.list 一致）

| 类型 | 域名 | 分类 | 用途与证据 | 副作用/边界 |
|---|---|---|---|---|
| DOMAIN-SUFFIX | muse.ai | A | 官网、`api.muse.ai`、SDK/设备入口；S1/S2/S3 | 覆盖所有子域，包括 `auth.muse.ai`；不表示该子域一定存在或本次已登录。 |
| DOMAIN-SUFFIX | metaaivm.com | A | `hatch.metaaivm.com` Noise WebSocket、按用户 VM 分配的媒体地址；S1/S2/S3 | 包含其他用户 VM 地址；仅对本机发出的连接生效。 |
| DOMAIN-SUFFIX | meta.ai | A | Muse 接口/生产 origin `agent.meta.ai` 等；S1/S4 | 其他 Meta AI 网页/应用也会走此策略。 |
| DOMAIN-SUFFIX | meta.com | A | OIDC 账号认证 `auth.meta.com`；S1/S5 | 其他 Meta/Quest 账号和网站流量也会走此策略；用户明确要求覆盖整个域。 |
| DOMAIN-SUFFIX | metaaiusercontent.com | B | 应用/生成内容预览 origin；S1 的预览处理代码与 CSP 相互印证 | 同域其他 Meta AI 内容也会走此策略；具体图片/视频链路未实测。 |
| DOMAIN-SUFFIX | ecto1usercontent.com | B | 预览 origin 的兼容后缀；S1 预览处理代码及 CSP | 兼容资源是否实际使用取决于账号/版本。 |
| DOMAIN | production.museai.com | B | S1 当前代码的生产 origin 集 | 未将整个 `museai.com` 后缀加入。 |
| DOMAIN | voice.museai.com | B | S1 当前代码的语音 origin 集 | 语音功能未测试；不能覆盖未知 UDP 中继。 |
| DOMAIN | www.facebook.com | B | S1 对此主机 `/auth_platform` 认证跳转的明确校验 | 所有发往此主机的 Facebook 浏览/登录/上报也会受影响，域名规则无法按 URL 路径区分。 |
| DOMAIN | www.instagram.com | B | S1 对此主机 `/auth_platform` 认证跳转的明确校验 | 影响该主机的 Instagram 浏览/登录；未加入整个 `instagram.com`。 |
| DOMAIN | static.xx.fbcdn.net | A | S5 公开登录 HTML 直接引用图标和多个脚本 | Facebook/Instagram 等复用此主机的静态资源也会走此策略。 |
| DOMAIN | nest-static.fbcdn.net | B | S1 明确构造的 Muse 资源加载路径 | 其他复用此 CDN 主机的应用也可能受影响。 |

规则文件只有 `DOMAIN` / `DOMAIN-SUFFIX` 与域名两列，无策略组名。
不重复添加 `auth.meta.com`、`api.muse.ai`、`hatch.metaaivm.com` 等已被后缀覆盖的精确规则。

## 初始扩展项与其他候选的处理

| 项目 | 分类 | 处理及原因 |
|---|---|---|
| muse.ai / metaaivm.com / meta.ai / meta.com | A | 按上表启用后缀，覆盖登录、Hatch/VM 和 Meta AI。 |
| metaaiusercontent.com / ecto1usercontent.com | B | 有预览处理代码，启用后缀。 |
| meta-agents-apps.workers.dev | C | S1 `frame-src` 出现 `*.meta-agents-apps.workers.dev`；本次 49 个直接脚本中未确认实际访问。暂不启用；有连接日志后可加入 `DOMAIN-SUFFIX,meta-agents-apps.workers.dev`，不可加入整个 `workers.dev`。 |
| llama.com / atmeta.com | C | S4 的 Meta 归属规则，不构成 Muse 调用证据；未启用。 |
| graph.facebook.com / b-graph.facebook.com / graph.meta.ai | C | 按“Meta GraphQL 主机”描述核查的候选，未获取本次 Muse 请求证据；未猜测必需性。`graph.meta.ai` 已被 `meta.ai` 后缀覆盖，不另加重复行。 |
| www.facebook.com | B | 启用精确主机；Muse 当前认证校验代码支持。 |
| web.facebook.com / m.facebook.com | C | 前者在 S1 CSP 的 form-action 中，二者可见于 S5 通用 Meta 域名注册表；未确认 Muse 认证实际跳转到它们，未启用。 |
| connect.facebook.net | C | S1 Pixel 脚本/追踪入口；未启用为 Muse 功能依赖。 |
| fbcdn.net | A/B（已观察到两个精确主机） | 用上表两个精确 DOMAIN 替代整个后缀，减少其他 Meta 应用的影响。S1 CSP 的 `*.fbcdn.net` 不能确定地区 CDN 完整列表。 |
| fbsbx.com | C | S1 CSP 允许媒体/iframe；S5 通用辅助代码也识别该域，但不是 Muse 实际媒体请求证据。暂不启用整个后缀，也不猜测 `lookaside` / `attachment` 必需。 |
| meta-ohttp-config-prod.fastly-edge.com | C | 用户点名的两个 Meta OHTTP 主机之一；公开检索可定位名字，本次所检查的 Muse/登录直接脚本未确认 Muse 使用。不默认启用；若日志确认，使用精确 DOMAIN。 |
| meta-ohttp-relay-prod.fastly-edge.com | C | 同上；不能由“Meta 通用 OHTTP 服务存在”推出“Muse 必需”。不可将整个 `fastly-edge.com` 导入。 |
| js.stripe.com / hooks.stripe.com / m.stripe.network / api.stripe.com | C（条件支付能力） | CSP 和付款 iframe 处理代码；尚无支付请求实测。保留可选，未启用。支付页面及购买保护是 S6 描述的能力，不代表订阅和购物共用完全相同链路。 |
| link.com | C | 当前代码中有 Stripe Link provider 标识；未确定登录/支付子域，未添加整个后缀。 |
| cdninstagram.com / cdn.whatsapp.net / oculuscdn.com | C | CSP 中图片/媒体允许来源，未确认 Muse 使用的具体主机，未启用。 |
| fonts.googleapis.com / gstatic.com / www.google.com | C | CSP 的字体/图标来源；未新增规则，继续由上游 Google 规则处理。 |
| youtube.com / youtube-nocookie.com | C | CSP 嵌入能力，未纳入 Muse，原视频规则保留。 |
| hatch.ecto1.ai / hatch.club | C | 生产/兼容 origin 集中的其他名称，未确认当前用户实际使用；未加入。 |
| willow606.com / exe.xyz / par-msl.dev / localhost | C | VM 地址辅助或开发允许列表；不是请求观测，未启用。 |
| x-marketing.fbthirdpartypixel.com / va.vercel-scripts.com | C | 营销/测量代码，未把测量流量作为功能依赖。 |

C 类不等于无用或被屏蔽；它们仍按上游原有规则分流。上线后如发现它们与 Muse 主链路出口不一致导致问题，再根据日志有针对性地补入。

## 功能、链路与实际验证状态

| 功能 | 公开技术证据 | 本次实际测试 |
|---|---|---|
| Muse 首页 | S1 HTML、脚本和 CSP | **已测试：匿名 HTTPS 页面返回 200，49 个直接脚本下载成功**；未证明所有浏览器交互正常。 |
| 登录 | `auth.meta.com` OIDC、Facebook/Instagram 认证回跳代码 | **未测试**：未在 Muse 完成账号登录；已有 Chrome 的 Muse 标签页显示空白内容区，无可交互的登录/对话控件，无法继续端到端操作。GitHub 登录不等于 Muse 登录。 |
| 普通对话 | S2/S3 的 API、Hatch Noise WebSocket 与订阅协议 | **未测试**：无已登录 Muse 测试会话。 |
| 图片生成 | S3 记录生成媒体事件与 VM 下载；S1 预览 origin | **未测试**。 |
| 视频生成 | S3 视频附件路径；S1 媒体/预览允许来源 | **未测试**，账号功能资格亦未验证。 |
| 文件上传/下载/预览 | S3 workspace 上传和 `/media/raw/workspace/` 下载，S1 预览域 | **未测试**：未上传任何用户文件。 |
| Agent 工作区/云端浏览器/Secure VM | S2 网关协议、S3 事件与 S6 架构说明 | **未测试**。云端 VM 对第三方的请求通常发生在服务器，iPhone 规则只控制设备本地发出的连接，不能为云端浏览器设定出口。 |
| Meta/其他账号关联、Connector | S1 provider/连接处理代码、S6 功能说明 | **未测试**：没有授权或新增第三方权限。OAuth 页面可在本机打开，Connector API 也可能从云端发出；不可据 provider 名称把全部第三方域名加入。 |
| 支付/订阅 | S1 Stripe iframe/CSP、S6 Stripe Link 能力说明 | **未测试**：未购买、开通订阅或更改支付设置。 |
| 各地区 CDN 与实时音视频 | CSP 和固定主机线索 | **未测试**：当前 HTTP 响应不能代表其他地区/账号。 |

未运行 iPhone Shadowrocket，未执行其真实解析器或规则诊断。静态域名测试只能证明按 DOMAIN/DOMAIN-SUFFIX 语义匹配预期，不证明网络链路、账号地区、日本节点质量或全部功能正常。

## 主配置的优先级与保留项

新增策略组：

```ini
🎬 Muse / Meta = select,🇯🇵 日本节点,🚀 节点选择,PROXY,DIRECT,policy-select-name=🇯🇵 日本节点
```

个人 RULE-SET 在上游 HTTPDNS 防泄露之后，邮件、Google、AI、Global 之前。此前的规则保留原顺序；未发现与启用域名相交的前置显式域名规则。局域网、私有 IP、WebRTC 特殊策略仍然优先；没有把 Muse 规则移到它们前面。

**既有例外仍然成立：**前置 UDP WebRTC 的 3478/5349/19302–19309 端口规则，以及 TCP 5223/特定 IP 规则、skip-proxy/tun 排除可先于域名分流生效。上游 WebRTC 默认拒绝非中国相关 UDP，可能影响 Muse 实时音视频/浏览器功能；本次保持原样，不能保证所有 Muse 连接都命中个人策略。远程 HTTPDNS 集也会更新；当前版本需继续核查冲突。

Google/Gemini、AI、邮件、IBKR/券商等上游远程 URL 与规则均保留。LICENSE 原样保留；未新增 DNS、MITM、广告规则或工作流。上游本来就有的 `[MITM]` 和 URL Rewrite 原样保留。

## iPhone Shadowrocket 验收与补漏

1. 使用 [个人配置](https://raw.githubusercontent.com/jkl00/Shadowrocket-Rules/main/Shadowrocket.conf) 下载并启用；更新远程规则。确认 `🎬 Muse / Meta` 为 `🇯🇵 日本节点`，日本节点组有可用节点。现有手动选择可能保留，配置默认值不保证覆盖客户端旧选择。
2. 在 Shadowrocket 的规则测试/域名匹配功能（如当前版本提供）分别输入 `muse.ai`、`auth.muse.ai`、`auth.meta.com`、`meta.ai`；确认命中 `Muse_Meta.list` 并选择 `🎬 Muse / Meta`。`auth.muse.ai` 即使不提供服务，后缀匹配也应成立，不能用页面是否打开判断规则对错。
3. 清空/标记一段连接记录，断开旧 Muse 连接后重新打开网页；在 Shadowrocket 连接日志中逐条查看目标域名、命中规则、策略组和最终节点。界面菜单名随版本变化；不要为抓日志开启 MITM。
4. 逐项测试登录、普通对话、图片/视频生成、上传一个无隐私的小文件并下载预览、工作区/云端浏览器；再自行授权一个确实需要的第三方 Connector，只做只读请求。支付/订阅可检查页面加载，实际付款由本人决定。
5. 对每项记录：时间、功能、成功/失败、主机、策略链、可见错误。特别观察 `.metaaivm.com` WebSocket 是否建立、预览域、动态 `scontent-...fbcdn.net`、`*.fbsbx.com`、Agent 应用子域和 OAuth 回调。不要把 URL query、Cookie、Token、聊天内容或文件上传进公开仓库。
6. 对确认为 Muse 所需且目前出口错误的共享主机，先补 `DOMAIN,精确主机`；服务自有、子域动态变化时才考虑限定后缀。不要因云端访问第三方网站失败就把整个第三方服务导入本机策略。
7. 若候选主机在日志出现且功能需要，复核后可加 `DOMAIN,graph.facebook.com`、精确 Fastly OHTTP 主机，或 `DOMAIN-SUFFIX,meta-agents-apps.workers.dev`。`graph.meta.ai` 已覆盖，无需再加。新增 CDN 精确主机通常比整个 `fbcdn.net` / `fbsbx.com` 更少影响其他应用；如账号频繁更换地区 CDN，评估覆盖与副作用后再扩大。
8. 测试切换为 `🚀 节点选择`/`PROXY`/`DIRECT` 后新连接是否随组变化；随后恢复预期出口。规则正确而功能失败时，还需检查节点、账户地区、权限、WebRTC 前置规则和服务端错误。

## 本次静态验证结果

- 通过区块保留、策略组引用与默认值、所有规则策略引用检查。
- 16 个应命中与 14 个不应命中的域名案例通过，包括 `muse.ai`、`auth.muse.ai`、`auth.meta.com`、`meta.ai`；也验证未扩大到整个 Facebook、Instagram、WhatsApp、Workers、Fastly 和 Google APIs。
- 12 条启用规则均为两列格式，无重复或被其他个人后缀规则冗余覆盖的条目。
- 逐字比较确认主配置只变更 update-url、添加一个组和一条 RULE-SET；所有原规则/策略组、区块、远程来源保持不变。检查当前前置显式域名及 HTTPDNS 集，没有发现与启用个人域名交集；端口/IP 例外仍保留。
- LICENSE 和其余上游已跟踪文件字节一致；仅计划提交四个目标文件。敏感模式扫描和人工 diff 检查未发现订阅、节点、密码、私钥、GitHub Token 或账号会话数据。
- 上述是自编静态校验，不是 Shadowrocket 官方解析器或真实客户端测试。Raw URL 可访问性须在推送后另行检查。

## 维护与回滚

GitHub Fork 首页 **Sync fork → Update branch** 是最简同步方式；若提示冲突，克隆后合并 `upstream/main`，保留个人入口与规则，重新做静态和 iPhone 验证。不要点丢弃个人提交的同步方式，不要 force push。
本地 `origin` 指个人 Fork，`upstream` 指原作者；远程配置属于各自克隆的本地 Git 配置，不随 Git 提交传播。其他机器重新克隆后需要添加 upstream。
回滚可对个人功能提交执行 `git revert <commit>` 并正常推送；这保留历史，也会还原个人配置入口，请在回滚后重新核对导入地址。

## 关键公开脚本的证据摘要

| 文件 | SHA-256 |
|---|---|
| `3cmpldvl_h-8p.js` | `c086f0a7cc0d48704eb2daa22ed05413f3e65340861ab8214c0b5098b07cb049` |
| `169rnupgd87zi.js` | `5fa0e3f41c2adffc8d03c2875d347d575c9800e319747136b9e451a5de5a035e` |
| `0fw1h171o30gr.js` | `76861407a2b3a29b9b4c27a66282640be69e391c8bcfc4bee3e81acb7c190a23` |
| `2f2xzv4_d_jsf.js` | `9f9d184495382dd8938b0b89ada42d3f14a9e9fdd8ee47eaad9d8e0fb4bdf297` |
| `241xaz9jd44ky.js` | `10e32f4362c9a513817bc73f65895b083c4839a55aa56de28b9f2dcd1be25e05` |
| `111hi1ey48q2j.js` | `2c4a69e36ce8da642115e64c219cd3af64c93184baec5dcfbabc9d00b7bb9b00` |
