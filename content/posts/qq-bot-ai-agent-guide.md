---
title: "QQ 机器人接入 AI Agent 完整指南：从 q.qq.com 注册到网关跑通"
date: 2026-09-21
tags: [QQBot, Hermes, AI Agent, WebSocket, 网关]
author: Hermes 流水线
---

## 先说结论：不需要公网服务器

如果能在 QQ 里直接给一个 AI 助手发消息，让它查资料、跑脚本、把每天的项目进展主动推给你，很多人第一反应是「那得先租台服务器、配 HTTPS 回调」。其实不用。

2026 年的官方 QQ Bot API v2 提供了 **WebSocket 长连接**接入方式：你的本地机器主动连上腾讯的网关，事件从这条连接推过来，回答走 REST API 发回去。所以跑在自己笔记本上的 AI Agent（比如 Hermes）可以零公网暴露地接到 QQ 上。

这篇是完整的落地记录：注册、配置、重启网关、DM 配对、home channel，以及四个真的会卡住你的坑。

## 第 0 步：先分清楚「QQ 机器人」和「QQ 邮箱」

**为什么重要**：Hermes 同时支持 Email 平台（IMAP/SMTP）和 QQ Bot 平台，两者的环境变量都以 `QQ` 开头，混在一起排查会浪费大量时间。

- **QQ 邮箱平台**：`EMAIL_*` 系列变量，机器人通过收发邮件对话
- **QQ Bot 平台**：`QQ_APP_ID` / `QQ_CLIENT_SECRET`，走腾讯官方 Bot API v2

另外，官方 Bot 是**独立应用**，必须先在开发者平台注册，不像微信 iLink 那样扫码即用。平台在 Hermes 内部的注册名是 **`qqbot`**（注意不是 `qq`），后面所有命令和配置键都用这个名字。

## 第 1 步：在 q.qq.com 注册机器人

打开 https://q.qq.com/ ，用 QQ 登录 → 创建机器人 → **主体选个人**，填名称、简介、头像。

进去之后找 **「开发设置」** 页，这里有两样必须拿的东西：

- **AppID**：明文显示，直接复制
- **AppSecret**：默认脱敏，点旁边的眼睛图标查看。**它只在这一处显示，复制完再离开页面**，之后再想看得重新生成

**为什么重要**：`AppSecret` 相当于机器人的密码，Hermes 用它换取 access token（日志里那条 `Access token refreshed, expires in 7200s` 就是它）。丢了只能重置，重置后所有已配置的地方都要同步更新。

顺手看一眼状态：机器人此刻显示「离线（服务不可用）」是**正常的**，网关连上之后才会变在线，别在这里怀疑配置。

## 第 2 步：内部体验号码 = 沙箱白名单

新注册的机器人默认是**沙盒属性**——只有白名单里的人才能和它对话。这里有个术语漂移的坑：

**为什么重要**：官方文档和旧教程里写的是「沙箱测试 / 沙盒配置」，但 2026 年的新版后台里**根本没有「沙箱」这两个字**，照着教程找会找不到，然后误以为要发布上线才能测试。

这张白名单卡片在官方文档（bot.qq.com《QQ Bot 介绍与接入指南》8.3 节）里叫**内部体验号码**——支持添加 20 个用户号码用于内部开发与内邀体验；写作时后台实际卡片名若有不同，以后台页面显示的名称为准。

实际对应关系如下：

| 教程里的词 | 新版后台的位置 |
|---|---|
| 沙箱 / 沙盒配置 | 开发设置 → **内部体验号码** |
| 消息 intent 权限 | 开发设置 → **事件与回调配置**（二级页） |
| AppID / AppSecret | 开发设置页第一张卡片 |
| 服务器 IP 白名单 | 本地开发**不用配**（留空 = 所有 IP 可调 API） |

在「内部体验号码」里点「＋添加用户」，填自己的 QQ 号即可，**上限 20 人**。

两个实测细节：名单同步**不是实时的**，几分钟到几十分钟都有可能；另外，**不把自己加进去，机器人不会理你**——这是最容易踩的一脚。

## 第 3 步：事件订阅与回调选 WebSocket

进「事件与回调配置」，接入方式有 WebSocket 和 Webhook 两条路：

- **WebSocket**：本地 Agent 场景选这个，无需公网 IP、无需 HTTPS 证书、无需备案域名
- **Webhook**：需要一台公网可达的 HTTPS 服务器接收回调

页面自己也会提示：接 OpenClaw / Hermes 这类本地 AI Agent，选 WebSocket 即可。同时确认需要的 intent 已开启（C2C 私聊、群 @ 消息；只做私聊就够用）。

## 第 4 步：Hermes 侧配置

依赖先确认（适配器需要 `aiohttp` 和 `httpx`，通常已随安装带上，别急着装）：

```bash
python -c "import aiohttp, httpx; print('deps ok')"
```

在**目标 profile 的 `.env`** 里追加凭据（假设 HERMES_HOME 在默认位置，Windows 下即 `%LOCALAPPDATA%\hermes`）：

```ini
QQ_APP_ID=你的 AppID
QQ_CLIENT_SECRET=你的 AppSecret
```

启用平台：

```bash
hermes config set platforms.qqbot.enabled true
```

也可以用交互式向导：

```bash
hermes gateway setup
```

选 **QQ Bot**，几个问题的推荐答案：

1. **配置方式**：选「手动填 AppID/Secret」——凭据已经到手，扫码路径是多余的折腾
2. **DM 授权**：选 `pairing approval`（默认，也是适配器的默认值 `dm_policy: pairing`）
3. **Home channel OpenID**：**直接回车留空**——OpenID 是用户首次发消息后才生成的，这时候根本拿不到
4. 回到平台菜单确认显示 `QQ Bot (configured)`，再选 Done
5. 最后问「Restart the gateway to pick up changes?」→ **选 N**，理由见下一节

**为什么重要**：第 5 问选 N 不是偷懒。向导拉起的网关进程挂在当前终端进程树上，窗口一关它就死了，你会以为「配置没生效」，其实是进程没了。

## 第 5 步：重启网关（Windows 正确姿势）

**配置不热加载**，改完必须重启网关。而且 `hermes gateway stop` 在本机实测**不可靠**：执行后立刻拉起计划任务，命令显示成功，但老进程可能没死透，新进程起不来，`status` 里还是旧 PID（判断依据：日志时间戳连续、没有断点）。

可靠流程是「确定杀死 → 用计划任务拉起 → 确认 PID 变了」：

```powershell
hermes gateway status                        # 记下输出里的 PID
taskkill /PID <旧PID> /F
hermes gateway status                        # 必须看到 No gateway process detected
schtasks /run /tn Hermes_Gateway             # 计划任务拉起，脱离当前会话
hermes gateway status                        # PID 必须变了才算重启成功
```

成功连上的日志链是这样（用 Python 读，别用 `tail`，中文 Windows 控制台下中文会乱码）：

```bash
PYTHONPATH= python -c "
import os, pathlib
p = pathlib.Path(os.environ['LOCALAPPDATA'])/'hermes'/'logs'/'gateway.log'
for line in p.read_text(encoding='utf-8', errors='replace').splitlines()[-40:]:
    if 'qqbot' in line.lower():
        print(line)"
```

期望看到：

```
Connecting to qqbot...
Access token refreshed, expires in 7200s
WebSocket connected to wss://api.sgroup.qq.com/websocket
✓ qqbot connected
Gateway running with 2 platform(s)
[QQBot:xxxx] Ready, session_id=...
```

另外还会看到 `WebSocket closed → Reconnecting → Resume sent → Session resumed` 反复刷（实测间隔几十秒到几分钟一次）。这是长连接在自动重连并恢复会话，属于正常自愈，**不要去动它**——只要 `✓ qqbot connected` 出现过、消息收得到，就不用管。

## 第 6 步：DM 配对（平台名是 qqbot，不是 qq）

配对机制和微信一样：陌生人第一次发消息会收到一个配对码，管理员批准后才真正放行。

实测两条经验：**平台名必须写 `qqbot`**，写 `qq` 会直接报 `not found or expired for platform 'qq'`；**转发的配对码可能已过期**，用请求 ID 最稳：

```bash
hermes pairing list                       # 拿请求 ID（形如 c1ee2935fdbe72ed）和 User OpenID
hermes pairing approve qqbot <request-id>
# → Approved! User <OpenID> on qqbot can now use the bot
```

**为什么重要**：批准后用户要**再发一条新消息**才会收到回复（上一条已经走完流程了）。多个人的体验就是「每人一遍」：对方发码 → 你 list → 你 approve。

想省掉这一步就把策略放开，改完一样要重启网关：

```bash
hermes config set platforms.qqbot.extra.dm_policy open        # 全开放
hermes config set platforms.qqbot.extra.dm_policy allowlist   # 改走白名单
```

注意 `open` 还差一个二次确认：必须在目标 profile 的 `.env` 里再加一行 `QQ_ALLOW_ALL_USERS=true`（或设全局 `GATEWAY_ALLOW_ALL_USERS=true`）。这是防误开的保险——只把策略改成 open 而不打开这个开关，网关会直接拒绝启动并退出，报错 `Refusing to start: qqbot has dm_policy/group_policy set to 'open' but neither GATEWAY_ALLOW_ALL_USERS nor QQ_ALLOW_ALL_USERS is enabled.`

白名单是列表，直接写进 `config.yaml` 更稳（`config set` 适合标量）：

```yaml
platforms:
  qqbot:
    enabled: true
    extra:
      dm_policy: "allowlist"          # pairing(默认) | open | allowlist | disabled
      allow_from:
        - "第一个人的 OpenID"
        - "第二个人的 OpenID"
```

## 第 7 步：home_channel，让 Agent 主动找你

前六步都通了，机器人还只是「你问它答」。要让 cron 任务和完成通知能主动投递给你，得配 home channel，值为**你的 OpenID**（首次发消息后，日志里 `inbound message: platform=qqbot user=<OpenID>` 就有，形如 `A1B2C3D4E5F6A7B8C9D0E1F2A3B4C5D6`）：

```bash
hermes config set platforms.qqbot.home_channel <你的 OpenID>
# 等价写法：.env 里加 QQBOT_HOME_CHANNEL=<OpenID>
```

**为什么重要**：不配这个，kanban 的审批卡、任务完成汇总这类主动推送会无处可去（只能落回终端）。配完记得重启网关。⚠️ 另外，个人机器人的主动推送受腾讯频次额度限制，推不出去是平台规则，不是 Hermes 故障。

顺带留一句：QQ 群支持默认不开。要在群里用，得在 q.qq.com 开「群 @ 消息」intent、把机器人拉进群，再把 `platforms.qqbot.extra.group_policy` 设为 `open`（或 `allowlist` + `group_allow_from`）；和私聊的 open 一样，群 open 也必须在 `.env` 里设 `QQ_ALLOW_ALL_USERS=true`——dm/group 任一为 open 都检查这同一个开关，不设同样过不了启动校验；群里**必须 @ 机器人**才会响应。

## 四个实测会卡住你的坑

**坑 1：`gateway setup` 向导会把凭据写错 profile。**
执行 `hermes -p commander gateway setup`，菜单里明明显示 `QQ Bot (configured)`，但 `QQ_APP_ID` / `QQ_CLIENT_SECRET` 实际被写进了 **default 的 `.env`**，目标 profile 里一个都没有——现象是目标网关日志里永远没有 qqbot 连接行。

排查时**别用 `grep QQ`**：`EMAIL_ADDRESS=*@qq.com`、`WEIXIN_BASE_URL=*qq.com` 都会被误判进来。用 Python 只匹配行首：

```bash
PYTHONPATH= python -c "
import os, pathlib
home = pathlib.Path(os.environ['LOCALAPPDATA'])/'hermes'
targets = [home/'.env'] + sorted(home.glob('profiles/*/.env'))
for f in targets:
    if not f.exists(): continue
    keys = [l.split('=',1)[0] for l in f.read_text(encoding='utf-8', errors='replace').splitlines()
            if l.strip().upper().startswith(('QQ_','QQBOT'))]
    print(f, '->', keys)"
```

分发原则：**长期用的平台配 default，项目专用的配项目 profile**。

**坑 2：clone 出来的 profile 带着上一家的凭据。**
`hermes profile create --clone` 会连 `.env` 一起复制过来，于是微信/邮箱凭据在多个 profile 里重复。多网关同时运行时会报 `Weixin bot token already in use (PID ...)`，或者两个网关同时轮询同一个邮箱、互相吞对方的邮件。给非 default profile 配网关前，先把 `platforms.weixin.enabled`、`platforms.email.enabled` 置为 `false`。

这一点用上面那段检查脚本扫一遍最直观：实测某台机器上 `default/.env` 和某个 clone 出来的 profile 的 `.env` 里，躺着**完全相同的** `QQ_APP_ID` / `QQ_CLIENT_SECRET`。

**坑 3：手机 QQ 搜不到机器人。** 按概率排查：

1. 搜索结果要切到 **「机器人」分类**，在用户列表里翻当然找不到
2. 内部体验号码名单**同步有延迟**，刚加完等几分钟再试
3. 手机 QQ 版本太旧，更新
4. 最稳的办法：在管理员 QQ 里打开机器人资料页 → 分享名片/二维码 → 让对方扫码加好友，完全绕过搜索

**坑 4：连接被系统代理劫持。**
QQ 适配器用的 HTTP 客户端会继承 Windows 系统代理设置。Clash 开着一切正常，Clash 一关立刻连接被拒——因为 `*.qq.com` 被塞进了代理而端口没人监听。修法是把 `*.qq.com` 加进代理绕过列表（`ProxyOverride`），并且**在 Clash 的 bypass 设置里也加一份**（Clash 每次切换系统代理都会重写注册表里的这一项）。改完必须重启网关，运行中的进程仍沿用旧设置。

## 排障速查

| 现象 | 优先检查 |
|---|---|
| 秒断连 | AppID/AppSecret 是否写对；intent 是否开启 |
| 机器人不回消息 | 自己是否在「内部体验号码」名单里；名单是否已同步 |
| 日志无 qqbot 连接行 | 凭据是否落错 profile；`platforms.qqbot.enabled` 是否为 true |
| 群聊里 @ 了没反应 | 群 @ intent 是否开启；`group_policy` 配置；是否真的 @ 了 |
| 收到配对码但批不了 | 平台名写 `qqbot`；用请求 ID 而不是配对码 |
| 主动推送失败 | `home_channel` 是否配；腾讯侧频次额度 |

最后一步，也是整套流程的验收标准：手机 QQ 给机器人发一句「你好」，看到终端日志冒出 `inbound message: platform=qqbot user=<OpenID>`，几秒后收到回复——到这一步，你的 QQ 里就多了一个跑在本地、由 AI Agent 驱动的助手。

## 参考资料

- Hermes 官方文档 · QQ Bot：https://hermes-agent.nousresearch.com/docs/user-guide/messaging/qqbot
- 腾讯 QQ 机器人开发者文档（API v2）：https://bot.q.qq.com/wiki/develop/api-v2/
