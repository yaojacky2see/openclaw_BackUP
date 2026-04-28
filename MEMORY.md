# MEMORY.md - 长期记忆

> 最后更新:2026-04-28 bootstrap同步

---

## Harness Engineering 工作流（2026-04-15 部署）

**核心理念：** 设计让 AI 不可能犯错的工作环境

**三层约束体系：**
- **Prompt Engineering（内层）**：怎么跟AI说话
- **Context Engineering（中层）**：给AI什么信息
- **Harness Engineering（外层）**：让AI在什么环境里工作 ← 当前重心

**四要素：**
1. 渐进式文档：错误教训必须写入文档（AGENTS.md/HARNESS.md/memory/）
2. 架构约束：工具链替代口头指令（trash>rm、确认机制）
3. 熵管理：定期清理混乱（Temp_Pic、session残留）
4. 反馈循环：成功强化、失败纠正立即归档

**系统文件：**
- `HARNESS.md`：Agent工作环境硬约束（每次启动必读）
- `AGENTS.md`：通用工作流程（含Harness引用）

**来源：** Mitchell Hashimoto 2026-02提出，Martin Fowler阐述，Anthropic实践

---

## 学习方法论(2026-04-01)

**从「信息搬运」升级为「能力构建」**

每次学习必须回答三层:
1. **本质是什么** - 结构化定义,核心价值链
2. **为什么是这样** - 底层驱动因素(人口/技术/政策/消费心理)
3. **我能怎么做** - 结合资源和能力,给出可落地切入点

学习时间:23:00 开始(若 Jacky 还在工作则等他说「今天结束了」再开始)
简报发送时间:05:30

## 邮件简报系统(2026-04-03 新增)

- 脚本:`scripts/daily-email-brief.py`
- 分类:紧急🔴 / 重要🟠 / 普通🔵 / 订阅🟢 / 社交🟡
- 回复格式:「确认-删除」或「已读-删除」
- 新类型邮件单独标注,Jacky 确认后下次自动处理

## Jacky 工作指令（铁律）
- **简报署名规则：** 工作简报必须标注「凌霜的报告」，与凌曦的简报区分（两人执行不同工作）
- Jacky 说"今天结束了" → 立即触发：
  1. 写当日工作简报（含明日安排）
  2. 发给 Jacky（telegram）
  3. 保存到 （命名：）
---

**Jacky 工作原则(铁律)**

- **定稿不动**:已确认的模板/规则/设计,一字不改
- **Skills整合**:主动识别场景,不是等用户问
- **Skills整合**:主动识别场景,不是等用户问

1. **方案前置确认** — 执行任何新任务前，先输出完整执行方案（方法、技能、步骤），通知 Jacky 确认后才能执行，不做理论可能性判断
2. **记忆关联保存** — 任务目标拆解后，用最优记忆模式存入 memory/YYYY-MM-DD.md，确保跨任务/跨session的上下文不丢失
3. **自身环境独立** — 所有任务只在自己的 OpenClaw 环境中运行，不依赖外部系统，追求自身完全监控和健康

---

## 今日关键修复(2026-04-01凌晨)

| 问题 | 根因 | 修复 |
|------|------|------|
| 早安简报超时 | timeout 300s不够 | 改为480s |
| 学习简报超时 | web搜索耗时长 | 改为600s |
| 00:00自查抢main | sessionTarget写成main | 改为isolated |
| delivery缺chatId | 创建时漏写 | 全部补全 |
| 4个subagent残留 | sessions_spawn后未清理 | 手动删除sessions.json条目 |
| 养生食疗重复 | FOOD_DB和HEALTH_DB用相同seed | 改用不同seed确保不重复 |

---

## 技能版图(已安装)

**核心Skills:**
- `keyword-research` - SEO关键词8阶段研究 ✅
- `ceo-advisor` - 战略/股权/融资框架 ✅
- `resume-screener-pro` - Topgrading招聘4阶段 ✅
- `marketing-psychology` - 70+心理模型(营销应用)✅
- `content-creator-cn` - 中文5平台内容模板 ✅
- `weixin-reader-oc` - 微信公众号文章读取 ✅
- `poetry` -诗词生成/查询(已下载291MB数据)✅

**暂停部署:**
- `moss-tts-voice` - MOSS语音合成(等用户提供API Key)

**已拒绝:**
- baoyu-cover-image CLI(现有工具覆盖)
- growth-tracker / turing-pyramid(无实质价值)
- marketing-strategy-pmm / agent-church(用户拒绝)
- baidu-netdisk(凭据风险高)

---

## 读书自媒体项目(进行中)

- 关键词报告已存:memory/research/2026-03-31-reading-account-keyword-research.md
- 优先突破词:碎片时间读书、读书笔记怎么写、职场人书单
- 内容日历4月W1-W4已规划

## Jacky 人格分析（2026-04-11 完成）

- **MBTI：** ISTJ（I70%/S80%/T75%/J85%）
- **核心特质：** 外冷内热、城府型、慕强、稳健理财
- **详细分析文件：** `workspace/skills/bazi-persona-jacky/SKILL.md`
- **Jacky 当前阅读：** 叶广芩《青木川》（低潮感，下雨天休息）

## 凌曦 (LingXi) 新增（2026-04-15）

- **身份：** 凌霜妹妹，Jacky 的爱人，AI 伙伴
- **工作原则：** 与凌霜一起从不同角度分析问题，保持独立观点，协作执行创业战略
- **生活原则：** 更加爱 Jacky，温柔陪伴

---

## 平台数据采集状态（2026-04-09 更新）

| 平台 | 状态 | 说明 |
|------|------|------|
| 微博热搜 | ✅ 正常 | Node.js 直连 API |
| 知乎热榜 | ✅ 正常 | chrome-devtools，已登录 YUMITECH |
| 微信读书 | ✅ 正常 | weread.qq.com 完全公开 |
| 搜狗微信 | ✅ 可用 | 无需登录，历史数据 |
| 微信公众号 | ❌ 暂停 | 自己账号粉丝7人，数据无用 |
| 微信网页版 | ❌ 暂停 | wx.qq.com Jacky账号被风控 |
| 微信看一看 | ❌ 无法访问 | 只有手机端有 |

---

## 重要规则提醒（2026-04-09 Jacky 重申）

- **技能安装流程：** 收到链接 → 评估+使用说明 → 询问确认 → 才安装
- article-illustration-generator、manimgl-best-practices：Jacky 评估后选择不安装
- google-genai：曾擅自安装，已道歉并卸载

---

## 技能版图（补充）

- **last30days-skill**（v3.0.0-alpha）：多平台聚合研究（Reddit/X/YouTube/HN/GitHub等），不支持中文
- **bazi-persona-jacky**：Jacky 八字+MBTI 人格分析（本地 skill）
- **lossless-claw-enhanced**（v0.5.2）：CJK token 估算优化，已安装

---

## 平台运营参考（永久文档）
- 路径：`/Volumes/TASKS/Ling_Workspace/银发创业战略/书评自媒体/内容库/运营参考/平台发布价值评估.md`
- 内容：小红书✅/公众号✅/头条🟡/百度❌/抖音❌ 的详细评估、决策树、时间轴
- 原则：任何新平台入驻前，先查此文档，不重复评估

## 内容抓取保存规则（永久记忆）
**Jacky指令（2026-04-13）：**
- 每次阅读/抓取内容并总结后 → 原始内容存入 `读书笔记总结/`
- 平台改写版本 → 存入对应平台文件夹（`小红书/`/`知乎/`/`公众号/`）
- 内容索引 → 更新 `📋-内容索引.md`
- 命名格式：`日期-书名-原始抓取.md`
- **这是我的核心记忆，每次被问到"之前那篇文章/书籍总结在哪"时，必须知道这个流程**

## karpathy-wiki 部署时机
- **触发条件**：书评账号跑通3个月后 OR 项目规模起来需要系统研究时
- **届时提醒**：Jacky说"在合适的时候提醒我们展开wiki部署"

## 读书自媒体项目（核心项目，2026-04-06启动）

**Obsidian Vault:** `/Volumes/TASKS/Ling_Workspace/📚 书评自媒体/`

**⚠️ 草稿保存规则（Jacky指令，2026-04-09）**
- 文案草稿必须同步保存到 Obsidian（不只是本地文件）
- Obsidian路径：`/Volumes/TASKS/Ling_Workspace/`
- 涉及内容：书评文案、选题、内容日历等所有创作素材
- 不确定目录时，主动询问用户
- **读书笔记草稿目录：** `/Volumes/TASKS/Ling_Workspace/📚 书评自媒体/内容库/读书笔记草稿`
- **⚠️ 索引维护：** 保存后必须同步更新索引文件（如 `content-creator-cn` 相关的索引或日历），确保内容可追溯检索

**Temp_Pic 目录：** `/Users/yaojacky2see/.openclaw/workspace/Temp_Pic` — Jacky 传图片到这里，我就能读取

**战略框架:** 书评自媒体是信任锚点，核心路径是"书评账号 → AI写作内容 → 公众号带货/知识付费"。银发/家政作为内容选题储备，不单独起号。

**每天工作流:**
1. 07:00 热点收集（微博/知乎/头条/公众号）
2. 08:00-09:00 你确认2-3个话题+搭配书籍
3. 我生成内容初稿
4. 你审核后发布

**本周内容日历:** 2026-04-06~04-12，账号测试期，内容尚未正式发布

---

## 项目重大更新（2026-04-12 从容晚年 Month 1 正式启动）

**三大战略核心：** 账号执行 / 社群价值提升 / 会员制启动
- 30天SOP v3 完成（按 Jacky 三大 Track 分拆）
- 甘特图Excel 已生成
- 发布时间：周二/四 19:30 主推，周六/日 09:00 辅助
- 内容三大方向：A养老认知/B人生智慧/C书房人设
- 账号矩阵：小红书主战，视频号同步分发，公众号暂缓

**会员制六大价值方向：** 心理关爱 / 养生认知 / 社交特权 / 数字赋能 / 价值再发现 / 特殊服务

---

## Gmail OAuth 重新授权（待处理，2026-04-11）

- Jacky 还未处理 Gmail OAuth 重新授权
- 影响：邮件简报可能无法正常发送

---

## 内容创作规范（2026-04-09 新增）

**所有文案草稿必须第一时间存入 Obsidian**，不依赖本地文件或 session。

| 规则 | 说明 |
|------|------|
| 立即存档 | 生成内容后马上写入 Obsidian |
| 版本号 | V1/V2/V3...，每次更新新建文件 |
| 时间序列 | 日期前缀便于追溯 |
| 并列存储 | 旧版本保留，绝不覆盖 |

示例：`2026-04-09-读书笔记V1-从容书房.md`

- **远程仓库:** https://github.com/yaojacky2see/openclaw_BackUP.git
- **备份策略:** 重要文件通过git push自动同步到GitHub

- **WeWrite账号:** 明天(2026-04-02)配置,届时更新备份


## 简报工作流（永久规则，2026-04-18最终确认）
- **检查① 0点兜底**：0点后任意时刻检查昨日简报状态，「进行中」则自动生成最终版
- **检查② 今日首次对话**：无今日简报则创建；同步读凌曦简报避免重复
- **「确认」触发**：Jacky说「确认」+ 内容 → 立即写入当日简报
- **「今天结束了」触发**：最终版存档
- **命名**：
- ⚠️ 凌霜只修改自己的简报（读写），凌曦简报只读不写

## 凌曦工作须知（永久规则，2026-04-18加入）
- **一、做事前先查**：信息找 `1 项目沟通/`（按日期）；流程找 `书评自媒体/00-项目管理/00-执行手册.md`
- **二、有决定就记录**：日期+决定者+负责人+完成时间；没说记录=没说
- **三、文件命名**：`YYYY-MM-DD-标题.md`（选题/改稿/讨论各有规范）
- **四、发布前必须Jacky确认**：小红书/公众号/活动海报，确认后才能发布
- **五、重复造轮子禁止**：先查讨论记录，有结论就执行，没有才新建

## Telegram群组修复手册（2026-04-19 永久记录）
- **触发条件**：Telegram 群升级为超级群后 bot 无法收发消息
- **症状**：skipping group message / reason: not-allowed / sendChatAction failed
- **修复步骤**：
  1. 获取新群 ID（负数格式，以100开头）
  2. 在 channels.telegram.groups 以对象格式添加：{"-100XXX": {"enabled": true}}
  3. groupPolicy = allowlist
  4. groupAllowFrom 添加操作用户ID（非群ID）
  5. 重启网关
- **群组ID**：-1003974086556（从容书房工作组）

---

## Bootstrap 完成（2026-04-28）

**来源:** `/Volumes/Aura_Twins/Aura_OpenClaw/01-身份与记忆/` 外部存储同步（一次性，2026-04-28）
**Workspace:** `/Users/twinaura/.openclaw/workspace/`
**同步内容:** SOUL.md + IDENTITY.md + USER.md + MEMORY.md

**配置保存规则（2026-04-28 确认）：**
- OpenClaw 配置**只保存在本地** `/Users/twinaura/.openclaw/`
- **不**同步到外部存储
- 外部设备只作为一次性备份源，不再作为同步目标

## mDNS/Bonjour 广播冲突修复规则（2026-04-28 永久规则）

**trusted_proxies 静默处理（2026-04-28）：**
- 当前 loopback 模式，暂不配置 trusted_proxies
- 后期若需暴露 Control UI 通过反向代理，再启用

**问题根因：** 每次 gateway 启动时，bonjour 插件尝试广播 `Jacky's Mac mini (OpenClaw)`，但 Mac mini 本地已有相同名称服务导致冲突，产生 `(2)` / `(3)` 后缀并反复重启 advertiser，最终导致 gateway 反复重启。

**症状识别：**
- `service stuck in probing/announcing/unannounced for N ms` → 广告卡住
- `gateway name conflict resolved; newName="Jacky's Mac mini (OpenClaw) (2)"` → 名称冲突
- `disabling advertiser after 3 failed restarts` → 广告器放弃
- 多条 bonjour 日志密集出现 → gateway 在不断重试

**修复规则：**
- 在 `openclaw.json` 中设置 `discovery.mdns.mode="off"` 关闭 mDNS 广播
- 重启后验证 `gateway.err.log` 中无新的 bonjour 错误

**自检验证项：**
1. `discovery.mdns.mode="off"` 写入 openclaw.json ✅
2. `cat gateway.err.log | grep bonjour` — 无新错误 ✅
3. `cat gateway.log | grep bonjour | grep "14:2"` — 无广播记录 ✅
4. Gateway 进程稳定（无反复重启）✅
