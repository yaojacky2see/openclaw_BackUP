# Errors

Append structured entries:
- ERR-YYYYMMDD-XXX for command/tool/integration failures
- Include symptom, context, probable cause, and prevention

---

## ERR-20260428-001: memory-lancedb-pro v1.0.32 TypeScript 未编译

**症状：** `Cannot find module './cli.js'`
**根因：** npm 包只有 TypeScript 源码，无编译产物
**教训：** 第三方 npm skill 需确认是否有 dist/ 或 build/，否则需 esbuild 手动编译
**预防：** 安装 skill 前检查 package.json main 字段，.ts 源码需要编译

## ERR-20260428-002: esbuild 编译后 stringEnum 工具注册失败

**症状：** `TypeError: (0 , _pluginSdk.stringEnum) is not a function`
**根因：** OpenClaw SDK 版本与 skill 期望的 `stringEnum` 函数签名不匹配
**教训：** Jacky fork 版本（v1.1.0-beta.10）自带 stringEnum，完美兼容
**预防：** 优先使用 Jacky 维护的 fork 版本

## ERR-20260428-003: GitHub 推送失败（fastgit.xyz 被墙）

**症状：** `error: no DAV locking support on https://hub.fastgit.xyz/`
**根因：** ~/.gitconfig 全局配置把所有 GitHub 访问重定向到 fastgit.xyz 镜像（该镜像被墙）
**教训：** 移除 ~/.gitconfig 中的 `insteadOf` 规则，或在 repo 级别覆盖
**预防：** 以后遇到 git 推送失败，先 `cat ~/.gitconfig` 检查重定向规则

## ERR-20260428-004: 嵌套 git 仓库导致 submodule 警告

**症状：** `warning: adding embedded git repository`
**根因：** 直接 clone 一个 git repo 到 workspace
**教训：** clone 后立即 `rm -rf .git` 避免嵌套
**预防：** `git clone` 到 workspace 后立即检查是否有 .git 目录

## ERR-20260428-005: ClawhHub 429 限流

**症状：** `ClawhHub /api/v1/download failed (429): Rate limit exceeded`
**根因：** ClawhHub 对未认证请求限流
**教训：** 网络安装失败时改用 npm 或 GitHub raw URL
**预防：** 准备多个安装源（npm、GitHub、直接下载）
