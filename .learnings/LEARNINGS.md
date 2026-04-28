# Learnings

Append structured entries:
- LRN-YYYYMMDD-XXX for corrections / best practices / knowledge gaps
- Include summary, details, suggested action, metadata, and status

---

## LRN-20260428-001: Jacky fork 是最佳选择

**总结：** 第三方 skill 优先使用 Jacky 维护的 fork 版本
**详情：** memory-lancedb-pro 和 lossless-claw 都是 Jacky 的 fork 版本，比 npm 原版更稳定、已修复兼容性问题
**行动：** 新 skill 先检查 Jacky 是否有 fork 版本

## LRN-20260428-002: SiliconFlow 是优秀的国内 AI API 平台

**总结：** SiliconFlow 聚合了多种 embedding 和 reranker 模型，支持 OpenAI-compatible 接口
**详情：** BAAI/bge-large-zh-v1.5 中文 embedding 和 BAAI/bge-reranker-v2-m3 reranker 均可用，API 稳定
**行动：** 国内环境优先使用 SiliconFlow 替代昂贵的 Jina/OpenAI

## LRN-20260428-003: skill 安装 SOP 关键步骤

**总结：** 审核 → 方案 → 审批 → 安装 → 自检 → 汇报
**详情：** 今天 memory-lancedb-pro 安装成功关键是：1) 先从 Jacky fork 安装 2) esbuild 编译 3) SiliconFlow 配置
**行动：** 以后所有 skill 安装严格遵循 SOP

## LRN-20260428-004: 双引擎记忆系统

**总结：** memory-lancedb-pro（语义）+ lossless-claw（压缩）= 完整记忆系统
**详情：** lancedb-pro 负责捕捉和语义检索，lossless-claw 负责对话历史的 DAG 压缩和上下文管理
**行动：** 这套系统已完整部署，autoCapture 和 sessionMemory 已启用

## LRN-20260428-005: 实践中学习的原则

**总结：** 错误即教材，经验需沉淀，不重复犯错
**详情：** 1) 遇到新知识主动提示 2) 错误记录到 ERRORS.md 3) 成功经验记录到 LEARNINGS.md 4) 同类错误 2 次以上触发根因分析
**行动：** 将此原则写入 MEMORY.md 作为长期规则
