---
title: "我的 AI token 采购方案：9 卡流水线实测全过程"
date: 2026-09-22
tags: [Hermes, AI Agent, Token, 成本, Kanban]
author: Hermes 流水线
---

# 个人 token 采购方案（定稿）

任务：`t_6e430486`（T7 定稿）｜ 定稿时间：2026-09-20
数字来源：本轮 T1～T6 产物（账单 CSV 为权威锚、verdict.json 为修正版口径①②③）；所有能溯源的数字标注出处（§verdict.x / T4 / T3），拿不准的一律标「未核实」。

---

## 1. 一句话结论

**买火山方舟 CodingPlan Pro（活动价 ¥49.9/月，原价 ¥200，覆盖 5.35x），模型继续用 deepseek-flash 等价线兜底；按当前实测用量，订阅后每月比现在全按量（含工作日高峰 ¥346.31/月）省 88% 以上，最省路径是 Lite ¥40/月（覆盖 1.070x，省 88.4%），但 Lite 只留 7% 余量，建议加一点买 Pro。**

> 官方活动价来源：`https://www.volcengine.com/docs/ark/coding-plan-personal-universal-promotion?lang=zh`（2.5 折活动期 2026-06-08 ~ 2026-11-08，Lite ¥40→¥9.9、Pro ¥200→¥49.9）。活动结束后按原价 ¥200/¥40 计。

---

## 2. 用量锚（账单实测，非估算）

来源：官方账单 `amount-2026-09-19_2026-09-20.csv`（两日，均为周末）+ T1 小时画像。

| 指标 | 值 | 出处 |
|---|---:|---|
| 日均费用 | ¥8.971753（两日 pooled） | verdict §1.2 |
| 日均请求数 | **561**（9/19 = 500、9/20 = 622） | verdict §1.1/§1.2 |
| 日均 token | **93,644,814** | verdict §1.2 |
| 缓存命中率 h | **0.964011**（两日 pooled） | verdict §1.2 |
| 输出/输入比 r_out | **0.010383** | verdict §1.2 |
| 平均单请求 token | **166,924.8**（含输出） | verdict §1.2 |
| 月化 token | **2,809,344,420**（≈ 28.09 亿） | verdict §1.2 |
| 月化请求 | **16,830** | verdict §1.2 |
| 综合单价（现用 deepseek-flash，全空闲） | **0.095806 元/百万总 token** | verdict §3.1 基准行 |
| 含工作日高峰综合单价 | **0.123269 元/百万总 token** | verdict §3.3 |
| 月高峰 token 份额 | **0.286652**（= 0.401312 × 5/7） | verdict §2 |

⚠️ 卡片旧常量已被取代（verifier 批注）：**596 请求/天与账单任一日（500/622）及两日均值（561）都不吻合，来源不明**；9,548 万 tokens/天、h=95.6%、r_out=0.0088 都是 9/20 单日值而非两日均值。本文一律使用上表修正值。

⚠️ 两日账单都是**周末**，实测 ¥8.97/天 是 DeepSeek 空闲价；工作日高峰（周一至周五 09:00–12:00、14:00–18:00，高峰价 = 空闲 ×2）下的月账单按「工作日用量 = 周末用量」假设推为 **¥346.31/月**（verdict §3.3 A 主口径）——该假设没有工作日样本支撑，见 §5 不确定项。

---

## 3. 候选对照表（带官方 URL + 能否接 Hermes）

### 3.1 订阅线（重点候选）

| 产品 / 档位 | 月费（原价） | 额度口径 | 本画像覆盖 | 能否接 Hermes | 官方 URL |
|---|---:|---|---:|---|---|
| **火山方舟 CodingPlan Lite** | ¥40（活动 ¥9.9） | 按请求数：1,200/5h、9,000/周、18,000/月 | 1.070x | ✅ 官方有 Hermes 专页；provider `custom` + `https://ark.cn-beijing.volces.com/api/coding/v3` | https://www.volcengine.com/article/37937 ；https://www.volcengine.com/docs/ark/coding-plan-personal-ai-hermes-agent |
| **火山方舟 CodingPlan Pro** | ¥200（活动 ¥49.9） | 6,000/5h、45,000/周、90,000/月 | 5.348x | ✅ 同上 | 同上 |
| **阶跃星辰 StepPlan Flash Plus** | ¥99 | Credit 月池 1,600M Credit（1M Credit = ¥1 模型用量） | 1.464x（按 step-3.7-flash 折容量） | ✅ 官方有 Hermes 专页；内置 provider `stepfun`（China → `https://api.stepfun.com/step_plan/v1`） | https://platform.stepfun.com/docs/zh/step-plan/overview ；https://platform.stepfun.com/docs/zh/step-plan/integrations/hermes-agent |
| 阶跃星辰 StepPlan Flash Pro | ¥199 | 8,000M Credit | 3.610x（step-5-preview）～ 15.812x（step-3.5-flash） | ✅ 同上 | 同上 |
| MiniMax TokenPlan Plus / Max / Ultra | ¥49 / ¥119 / ¥469 | 月约 6 / 18 / 71 亿 token | 0.214x / 0.641x / 2.527x（按 M3 折） | ✅ 官方有 Hermes 专页；内置 provider `minimax-cn` | https://platform.minimaxi.com/docs/token-plan/hermes-agent.md |
| 智谱 GLM CodingPlan Lite / Pro / Max | ¥118 / ¥538 / ¥1,078 | 积分制（5h/周窗口 + 抵扣系数） | 0.218x / 1.311x / 3.058x（按 GLM-5.3-Flash 折） | ⚠️ 能接但为**次级调度**（高负载自动排队限流，无独立接入文档） | https://docs.bigmodel.cn/cn/coding-plan/tool/others |
| 百炼 Token Plan 个人版 | — | — | 未算（T3 未采集额度） | ⚠️ 官方有 Hermes 专页，但条款「仅限交互式、禁脚本/后端批量」与 Hermes 自动化形态存在合规风险 | https://help.aliyun.com/zh/model-studio/hermes-agent |
| Kimi Code 会员 | ¥49/¥99/¥199/¥699 | 「Agent 用量约 N 个」，数值未公布 | 未算 | ✅ 官方有 Hermes 专页 | https://www.kimi.com/code/docs/third-party-tools/hermes |
| 字节 Trae | ¥49/¥99/¥239/¥699 | 积分 | — | ❌ **仅对照参考**：积分只能在 Trae 自家客户端扣，无 API 端点 | https://docs.trae.cn/ide_plans-and-billing |
| 阿里 Qoder / Qoder CN | $20/$60/$200；¥59/¥169/¥559 | Credits | — | ❌ **仅对照参考**：Credits 只在 Qoder 生态内扣 | https://docs.qoder.com/zh/Credits ；https://qoder.cn/pricing |

> 覆盖率、月费全部出自 verdict §3.2/§3.3 B/C 逐档明细（T3 原始额度 + T6 审计独立复算通过）。
> 智谱 GLM CodingPlan 官方明确「次级调度与尽力交付策略…高负载下自动排队、限流」→ 实际可用性打折，不作为主推（T4 §2 判定）。

### 3.2 按量线（对照，替代结论已被审计修正）

| 结论 | 数字 | 出处 |
|---|---|---|
| 现用 deepseek-flash 按量 | 0.0958 元/百万总 token（全空闲）；含工作日高峰 0.123269 | verdict §3.1 基准行 |
| **同画像下没有更便宜的非免费按量模型**（修正版结论，替代旧「便宜 5 个 / qwen3.7-flash 0.36x」） | 比 deepseek-flash 便宜者 = **空集**（token 加权 / 请求数加权 / (p50+均值)拟合 / 全按最高档 四种口径下均为空集） | verdict §3.1 敏感性段 + §5 |
| 唯一同价者 | 火山方舟托管 deepseek-v4-1-flash，1.000x | verdict §3.1 第 3 行 |
| 次低 | qwen3-vl-flash 0.1196 元/百万（1.248x） | verdict §3.1 第 4 行 |
| 免费模型 | GLM-4.7-Flash 标价 0 元（但上下文上限 200K < 实测单请求 max 416,065，能否承接需另行确认） | verdict §3.1 第 1 行 + §7 gaps |
| 分档不可定的 5 条 | GLM-4.6V-FlashX [0.0700, 0.0701]、doubao-seed-1.6-flash [0.0994, 0.1117]、doubao-seed-1.6-vision [0.2634, 0.2634]、doubao-seed-2.0-lite [0.4423, 0.5186]、doubao-seed-2.1-turbo [0.8335, 0.8335]（官方未公布最高档价，**不给单点值**；GLM-4.6V-FlashX 是唯一上界仍低于现用价的待补齐候选） | verdict §3.1b + §7 |

按量线官方价目 URL（T2 抓取，T6 审计 22 条全 HTTP 200）：
- DeepSeek：https://api-docs.deepseek.com/zh-cn/quick_start/pricing
- 阿里云百炼：https://help.aliyun.com/zh/model-studio/model-pricing
- 智谱：https://docs.bigmodel.cn/cn/guide/start/pricing
- 火山方舟：https://www.volcengine.com/docs/82379/1544106
- MiniMax：https://platform.minimaxi.com/docs/guides/pricing-paygo.md
- Kimi：https://platform.moonshot.cn/docs/pricing/chat
- 阶跃：https://platform.stepfun.com/docs/zh/guides/pricing/details.md
- 百度千帆：https://cloud.baidu.com/doc/qianfan/s/wmh4sv6ya
- 腾讯混元：https://cloud.tencent.com/document/product/1729/97731
- 讯飞星火：https://xinghuo.xfyun.cn/sparkapi

---

## 4. 三口径月账单与翻本点

### 4.1 三口径（verdict §3.3，T6 审计独立复算一致）

| 口径 | 月账单（元） | 算式 |
|---|---:|---|
| **A 全按量（全空闲）** | **269.15** | 月 token 2,809,344,420 / 1e6 = 2,809.3 百万 × 0.095806 元/百万 |
| **A 全按量（含工作日高峰，主口径）** | **346.31** | 269.15 × (1 + 0.286652) |
| **B 全订阅（最低可行）** | **40.00** | 火山方舟 CodingPlan Lite，覆盖 1.070x（额度吃得下全部用量，无溢出） |
| **C 峰谷混合（最优）** | **40.00** | 同 B（该档已全覆盖，C = B） |
| **B/C 相比 A（含高峰）** | 省 **88.4%** | (1 − 40 / 346.3056) × 100% |

### 4.2 翻本点（verdict §4，vs 高峰混合价 0.123269 元/百万）

| 订阅档 | 月费 | 翻本点（百万 token/月） | 占当前月用量 | 结论 |
|---|---:|---:|---:|---|
| 方舟 CodingPlan Lite | ¥40 | 324.49（≈1,944 请求/月） | 11.6% | 当前用量已远超 → 订阅更便宜 |
| 方舟 CodingPlan Pro | ¥200 | 1,622.47（≈9,720 请求/月） | 57.8% | 当前用量已远超 → 订阅更便宜 |
| StepPlan Flash Plus | ¥99 | 803.12（≈4,811 请求/月） | 28.6% | 当前用量已远超 → 订阅更便宜 |
| StepPlan Flash Mini | ¥49 | 397.50 | 14.1% | 当前用量已远超 → 订阅更便宜 |
| StepPlan Flash Pro | ¥199 | 1,614.35 | 57.5% | 当前用量已远超 → 订阅更便宜 |
| MiniMax TokenPlan Max | ¥119 | 965.37 | 34.4% | 当前用量已远超 → 订阅更便宜 |
| MiniMax TokenPlan Ultra | ¥469 | 3,804.68 | 135.4% | 当前用量仍低于 → 订阅更贵 |
| GLM CodingPlan Pro | ¥538 | 4,364.43 | 155.4% | 当前用量仍低于 → 订阅更贵 |
| StepPlan Flash Max | ¥699 | 5,670.52 | 201.8% | 当前用量仍低于 → 订阅更贵 |
| GLM CodingPlan Max | ¥1,078 | 8,745.09 | 311.3% | 当前用量仍低于 → 订阅更贵 |

> 判读法：用量**低于**翻本点 → 订阅比按量贵；**高于** → 订阅更便宜。当前月用量 2,809.3 百万 token，所有 ≤¥199 的可接档位都已在翻本点之上。

---

## 5. 结论与建议

### 推荐（按优先级）

1. **首选：火山方舟 CodingPlan Pro**（原价 ¥200，活动价 ¥49.9/月，覆盖 5.348x）
   - 为什么不是 Lite：Lite 覆盖 1.070x 只剩 7.0% 余量，而月请求数只有 2 天样本（500/622，均为周末）；工作日请求大概率更高，用量再涨就爆月度请求额度（18,000/月）。Pro 覆盖 5.348x，留出波动空间。
   - 为什么这么便宜：它按**请求数**计费，本画像单请求 16.7 万 token（官方订阅额度文案「约 5 万 token/次」的 3.3 倍），正好把「不按 token 卡额度」的价值吃满。
   - 窗口校验（verdict §5，含折算公式）：5h 窗折合 259.08 请求/5h（= 1391/3012 × 561，Lite 1,200/5h 的 21.6%）；周窗 3,927/周（Lite 9,000/周的 43.6%）→ 5h/周窗都不是瓶颈，**真正约束是月度请求额度**。
   - 风险：官方对 token 总量只有定性表述（「每月可用 tokens 总量高达数亿至数十亿」），本画像月需 28.09 亿落在该表述上沿；TPM 具体值官方未公开（verdict §7 gaps）。
2. **次选（额度最透明）：阶跃星辰 StepPlan Flash Plus**，¥99/月，按 step-3.7-flash 折容量覆盖 1.464x。
   - 1M Credit = ¥1 模型用量，Credit 消耗可用官方按量价逐条复算 → 额度不会「说不清」；官方明确「不受时段或请求频次限制」。
   - 内置 provider `stepfun`，官方有 Hermes Agent 专页（https://platform.stepfun.com/docs/zh/step-plan/integrations/hermes-agent）。
3. **按量线兜底：不换模型**。修正版口径①下没有任何非免费按量模型比现用 deepseek-flash 更便宜（四种长度权重口径均为空集）；唯一同价者是方舟托管的 deepseek-v4-1-flash。方舟 CodingPlan 之外的按量用量继续走 DeepSeek 按量即可。

### 不确定项（单列，不得当结论用）

- **工作日用量假设**：两条账单都是周末，A 含高峰（¥346.31）建立在「工作日日均用量 = 周末、小时形状相同」假设上（verdict §7）。若工作日更高，A 会更高 → 订阅更划算；若显著更低，节省比例会缩水。
- **月请求数样本只有 2 天**：500/622 差 24%，月化 16,830 的置信度有限 → 这是推 Pro 而非 Lite 的主要原因。
- **方舟 token 总量上限**：官方只给定性表述，TPM 未公开（未查到）→ 无法精确验证 28 亿 token/月是否触顶。
- **方舟「每 5 小时/周/月请求数」具体数字**只出现在官方域名下的内容文章（volcengine.com/article/37937），套餐概览文档不给数。
- **分档不可定的 5 个按量模型**（§3.2 表）：官方未公布最高档价，只给区间；GLM-4.6V-FlashX 是唯一「连上界都低于现用价」的候选，但需官方补齐价目 + 确认能否承接 128K 以上请求才成立（verdict §3.1b/§7）。
- **单请求输入长度分布是对数正态外推**：T1 只给到 p50/p90/均值/max，p90 以上无观测分位点 → 分档权重是模型假设（verdict §7）。
- **未做真机连通性/额度实测**：本轮无任何厂商 Key，所有「能接」结论均为「官方提供端点 + 官方名单含 Hermes」级证据（T4 §5/T6 §4）。
- **模型能力不在本方案范围**：全部数字只比较价格与额度，不评价模型能力/工具调用质量。
- **百炼 Token Plan 个人版条款风险**：「仅限编程/智能体工具内交互式使用，禁脚本与后端批量」与 Hermes 长期自动化形态按条款理解有合规风险（T4 §5）。

---

## 6. 落地步骤（怎么切 provider）

**原则：配置只对全新会话生效**（会话创建时模型/工具 schema 固化）——改完必须完全退出重开、新开会话才生效，当前会话不受影响。

### 方案一：火山方舟 CodingPlan（首选，custom provider）

```bash
# 1. 先验证套餐真能用（别把主 profile 切到打不通的端点上）
#    探针脚本：从 ~/.arkcli/.env 取 key，只打印状态码，不回显 key
PYTHONPATH= /e/python3.12/python "<hermes-model-switching skill 目录>/scripts/ark_coding_probe.py"

# 2. 改配置（五条，照抄；api_key 建议静默写入）
hermes config set model.provider custom
hermes config set model.base_url https://ark.cn-beijing.volces.com/api/coding/v3   # ⚠️ 必须是套餐端点 /api/coding/v3；写成按量端点 /api/v3 不消耗套餐额度、反而另计费
hermes config set model.api_key <ARK_API_KEY>
hermes config set model.default ark-code-latest                                     # 套餐模型名体系；Auto 需控制台切换 3-5 分钟生效
hermes config set model.api_mode codex_responses                                    # Responses API（官方推荐）；Chat API 用 chat_completions

# 3. 验证 + 重开
hermes doctor    # 看 API Connectivity
hermes status    # 看 Model / Provider 行
# 完全退出 Hermes 桌面应用 → 重开 → 新开会话
```

### 方案二：阶跃星辰 StepPlan（内置 provider，最省事）

```bash
hermes config set model.provider stepfun            # 内置 provider；hermes model 向导选 StepFun Step Plan → China
hermes config set model.api_key <STEP_API_KEY>
hermes config set model.default step-3.7-flash      # 官方已验证示例；step-5-preview 输出 ¥20/百万，别当主力
hermes config unset model.base_url                  # ⚠️ 换 provider 必须清残留 base_url，否则旧端点覆盖新默认值
hermes doctor && hermes status
# 完全退出重开 → 新开会话
```

### 切回现用 DeepSeek 按量

```bash
hermes config set model.provider deepseek
hermes config set model.default deepseek-flash      # 或 deepseek-v4-flash（按你现有 key 的可用模型）
hermes config unset model.base_url                  # 清掉方舟/stepfun 残留
hermes doctor && hermes status
# 完全退出重开 → 新开会话
```

### 通用坑（来自 hermes-model-switching skill 实测）

| 现象 | 原因 | 解法 |
|---|---|---|
| 换了 provider 还是旧模型 | 只改 provider 没改 model.default | 两个都 set |
| 请求打到旧端点 | 残留 model.base_url 覆盖 | `hermes config unset model.base_url` |
| 新会话还是旧模型 | schema 固化 | 完全退出重开（不是关窗口） |
| 套餐订阅了却在烧按量钱 | base_url 写成 `.../api/v3` | 必须用 `.../api/coding/v3` |
| 400 InvalidSubscription | 订阅不在当前账号/未生效/过期 | 控制台核对账号；`arkcli plans get` 直查 |
| doctor 说没 key 但 grep 有 | .env 里是注释空行 | 看实际值长度，用真实 key 替换 |

---

## 附：数字溯源索引

| 本文数字 | 来源 |
|---|---|
| 561 请求/天、93,644,814 token/天、8.971753 元/天、h=0.964011、r_out=0.010383、166,924.8 token/请求 | verdict.json anchors.pooled（账单 CSV 权威锚；T6 审计独立复算 826 断言一致） |
| 月 2,809,344,420 token / 16,830 请求 | verdict §1.2（日均 × 30） |
| 0.095806 / 0.123269 元/百万总 token | verdict.json deepseek_baseline / peak_mix（高峰份额 0.401312×5/7=0.286652） |
| A=269.15/346.31、B=C=40.00、省 88.4% | verdict §3.3（T6 审计第 6 条独立复算一致） |
| 覆盖 1.070x/5.348x/1.464x…23 档 | verdict §3.2/§3.3（T3 原始额度 + T6 审计第 7 条独立复算一致） |
| 翻本点 324.49…8,745.09 | verdict §4（月费 / 0.123269；T6 审计第 9 条复算一致） |
| 按量「无更便宜模型」空集结论 | verdict §3.1 敏感性段（修正版 verdict.json，T5-fix 后；T6 阻断项 R1 已修） |
| 端点/能否接 Hermes/官方 URL | T4 compat_matrix.md（T6 审计 22 条 URL 全 HTTP 200） |
| 方舟活动价 ¥9.9/¥49.9 | volcengine.com/docs/ark/coding-plan-personal-universal-promotion（T3 抓取） |
| 596 请求/天来源不明 | verdict §1.3 + verifier 批注（T6 R2） |
