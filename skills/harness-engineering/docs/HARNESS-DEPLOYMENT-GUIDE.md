# Hermes Harness Engineering — Agent 部署指南

> 本文档指导 Agent 在任意项目中独立部署 Harness Engineering 系统。
> 包含 6 个核心概念、完整执行步骤、常见问题与修复方案。
>
> **适用对象：** 具备 Shell + Git 能力的 AI Agent
> **前置条件：** 项目已初始化 Git，skills 目录存在

---

## 目录

1. [概念速览](#1-概念速览)
2. [执行流程总览](#2-执行流程总览)
3. [Step 0：准备](#step-0准备)
4. [Step 1：建地图（C1）](#step-1建地图c1)
5. [Step 2：写 Linter（C2）](#step-2写-linterc2)
6. [Step 3：写生成器（C3）](#step-3写生成器c3)
7. [Step 4：Agent 可读性（C4）](#step-4agent-可读性c4)
8. [Step 5：背压评分（C5）](#step-5背压评分c5)
9. [Step 6：状态文档（C6）](#step-6状态文档c6)
10. [常见问题与修复](#常见问题与修复)
11. [最终文件清单](#最终文件清单)
12. [快速命令速查](#快速命令速查)

---

## 1. 概念速览

| # | 概念 | 核心原则 | 落地工具 |
|---|------|---------|---------|
| C1 | 地图而非手册 | 渐进式披露，入口简洁，细节在子地图 | AGENTS.md（多级） |
| C2 | 机械化执行 | lint 规则 = 修复指令，规则必须可执行 | validate-skills.sh |
| C3 | 熵管理 | 文档自动生成，消除手工维护造成的漂移 | generate-agents.sh |
| C4 | Agent 可读性 | 根文件 ≤60 行，约束越严自主性越强 | AGENTS.md 行数限制 |
| C5 | 吞吐量改变合并 | 小步快跑，合并门控最小化，背压=质量门禁 | 背压评分系统 |
| C6 | Harness 精确定义 | Guide×Sensor 矩阵，计算性+推理性双重验证 | HARNESS.md |

**2×2 Guides × Sensors 矩阵**

| | 计算性（确定性） | 推理性（语义） |
|--|---------|---------|
| **引导器（前馈）** | bootstrap脚本 | AGENTS.md、Skills |
| **传感器（反馈）** | linter、类型检查 | AI code review、LLM-as-judge |

---

## 2. 执行流程总览

```
Step 0: 准备环境
    ↓
Step 1: 建入口 AGENTS.md（Map）
    ↓
Step 2: 写 validate-skills.sh（Linter）
    ↓
Step 3: 写 generate-agents.sh（Auto-gen）
    ↓
Step 4: 验证并修复 → 达到 100%
    ↓
Step 5: 精简根 AGENTS.md ≤60 行
    ↓
Step 6: 建 HARNESS.md 状态文档
    ↓
Git commit + push
```

---

## Step 0：准备

### 目标
确认环境具备基本条件：Git 仓库、skills 目录结构。

### 操作

```bash
# 确认在 Git 仓库根目录
cd ~/.hermes/hermes-agent

# 确认 skills 目录存在
ls -la skills/

# 确认 Git 状态
git status
git log --oneline -3
```

### 预期结果
- `skills/` 目录存在
- Git 仓库已初始化
- 至少有一些 skill 目录

---

## Step 1：建地图（C1）

### 目标
建立多级 AGENTS.md 系统：入口地图 + 子地图。

### 原则
- **入口（根）AGENTS.md**：≤60 行，按任务类型路由到子地图
- **子地图**：按分类组织，每个分类一个 AGENTS.md
- **渐进式披露**：不把所有 skill 都塞进入口

### 操作

#### 1.1 创建子目录 AGENTS.md

每个有多个 skill 的分类目录，都创建 `AGENTS.md`：

```bash
# 创建子地图（例如 mlops/）
mkdir -p skills/mlops
echo "# MLOps" > skills/mlops/AGENTS.md
echo "" >> skills/mlops/AGENTS.md
echo "> 入口：[skills/AGENTS.md](../AGENTS.md)" >> skills/mlops/AGENTS.md
```

#### 1.2 根 AGENTS.md 模板

```markdown
# Hermes Agent Skills — 智能体地图

> 本文件由 `generate-agents.sh` 自动生成。
> **核心原则：仓库即记录系统。**

---

## 快速路由

| 任务 | 入口 |
|------|------|
| GitHub/PR/Issues | [github/AGENTS.md](github/AGENTS.md) |
| MLOps 全流程 | [mlops/AGENTS.md](mlops/AGENTS.md) |
| 创意媒体 | [creative/AGENTS.md](creative/AGENTS.md) |
| 研究工具 | [research/AGENTS.md](research/AGENTS.md) |

---

## 子地图索引

| 目录 | 内容 |
|------|------|
| `github/` | PR/Issues/CodeReview (6 skills) |
| `mlops/` | 训练/推理/评估 (22 skills) |

> 加载 Skill：`skill_view(name="category/skill-name")`

*最后生成：YYYY-MM-DD*
```

### 关键陷阱

⚠️ **不要在根 AGENTS.md 中写不存在的 skill 引用**
- 如果 `skills/` 下没有 `apple-notes` 这个目录，就不要写 `` `apple-notes` ``
- 路由表只引用**实际存在的子目录**
- 引用格式用 `[name](path/)` 而不是 backtick 裸名

---

## Step 2：写 Linter（C2）

### 目标
创建 `validate-skills.sh`，用脚本验证 skills 目录的健康度。

### 原则
- **lint 规则 = 修复指令**：每个 FAIL 都附带清晰的修复方向
- **必须可执行**：bash 脚本，macOS/Linux 兼容
- **不含 bash 特性**：bash 3.2 兼容（macOS 默认）

### 脚本模板

```bash
#!/bin/bash
# =====================================================================
# validate-skills.sh — Skills 目录 Linter
#
# 检查项：
#   R1. 每个技能目录必须有 SKILL.md
#   R2. 子 AGENTS.md 与父 AGENTS.md 交叉链接
#   R3. AGENTS.md 表格引用必须指向真实目录
#   R4. 根 AGENTS.md ≤60行
#   R5. SKILL.md 无硬编码个人路径
# =====================================================================

SKILLS_DIR="$HOME/.hermes/hermes-agent/skills"
cd "$SKILLS_DIR"

ERRORS=0; PASSES=0; WARNINGS=0; TOTAL_CHECKS=0

RED='\033[0;31m'; GREEN='\033[0;32m'; YELLOW='\033[1;33m'
BOLD='\033[1m'; RESET='\033[0m'

pass()    { echo -e "${GREEN}[PASS]${RESET}  $1"; PASSES=$((PASSES+1)); TOTAL_CHECKS=$((TOTAL_CHECKS+1)); }
fail()    { echo -e "${RED}[FAIL]${RESET}  $1";  ERRORS=$((ERRORS+1));  TOTAL_CHECKS=$((TOTAL_CHECKS+1)); }
header()  { echo -e "\n${BOLD}━━━ $1 ━━━${RESET}"; }

# ── R1: 每个技能目录必须有 SKILL.md ─────────────────────────
header "R1: SKILL.md 存在性"

skill_dirs=$(find . -mindepth 2 -maxdepth 3 -type d ! -name 'scripts' 2>/dev/null | sort)
total=0; missing=0

for dir in $skill_dirs; do
  case "$dir" in
    */templates|*/references|*/assets) continue ;;
  esac
  total=$((total+1))

  # 分组目录（内有子 skill）不要求自身有 SKILL.md
  has_subs=$(find "$dir" -mindepth 1 -maxdepth 1 -type d 2>/dev/null \
    | grep -v '/templates$' | grep -v '/references$')
  if [[ -n "$has_subs" && ! -f "$dir/SKILL.md" ]]; then
    total=$((total-1))
    continue
  fi

  if [[ -f "$dir/SKILL.md" ]]; then
    pass "SKILL.md 存在: $dir"
  else
    fail "缺少 SKILL.md: $dir"
    missing=$((missing+1))
  fi
done

[[ $total -gt 0 ]] && [[ $missing -eq 0 ]] && pass "所有 ${total} 个技能目录都有 SKILL.md"

# ── R2: 子 AGENTS.md 交叉链接父 ─────────────────────────────
header "R2: AGENTS.md 交叉链接"

for agents_file in $(find . -name "AGENTS.md" ! -path "./AGENTS.md" 2>/dev/null | sort); do
  if grep -q "AGENTS.md" "$agents_file" 2>/dev/null; then
    pass "$agents_file → 父级链接"
  else
    fail "$agents_file 未引用父级"
  fi
done

# ── R3: AGENTS.md 表格引用有效性 ────────────────────────────
# ⚠️ 关键：只检查表格中的引用（| ... | `ref` |），跳过 markdown 标题
header "R3: Skill 引用有效性"

for agents_file in $(find . -name "AGENTS.md" 2>/dev/null | sort); do
  agents_dir=$(dirname "$agents_file")

  # 只从表格行提取引用
  refs=$(grep '^[[:space:]]*|' "$agents_file" 2>/dev/null \
         | grep -oE '`[a-z0-9-]+(/[a-z0-9-]+)*' \
         | tr -d '`' | sort -u)

  for ref in $refs; do
    case "$ref" in
      http*|https*|skill_view|~\/|../) continue ;;
    esac
    # 引用可能相对于 AGENTS.md 所在目录
    target1="$SKILLS_DIR/$agents_dir/$ref"
    target2="$SKILLS_DIR/$ref"
    if [[ -d "$target1" || -f "$target1/SKILL.md" || -d "$target2" || -f "$target2/SKILL.md" ]]; then
      pass "引用有效: $ref"
    else
      fail "引用无效: $ref (在 $agents_file)"
    fi
  done
done

# ── R4: 根 AGENTS.md ≤60行 ──────────────────────────────────
header "R4: Agent Readability（≤60行）"

root_lines=$(wc -l < "AGENTS.md" 2>/dev/null || echo 999)
if [[ $root_lines -le 60 ]]; then
  pass "根 AGENTS.md: ${root_lines}行（≤60 ✓）"
else
  fail "根 AGENTS.md: ${root_lines}行（>60行限制）"
fi

# ── R5: SKILL.md 无硬编码个人路径 ───────────────────────────
header "R5: 无硬编码路径"

hardcoded=$(find . -mindepth 2 -maxdepth 3 -name "SKILL.md" \
  -exec grep -l '~/Projects/\|/Users/[^/]*/\|C:\\Users\|D:\\' {} \; 2>/dev/null)
if [[ -n "$hardcoded" ]]; then
  for f in $hardcoded; do
    fail "硬编码路径: $f"
  done
else
  pass "无硬编码个人路径"
fi

# ── 背压强度评分 ──────────────────────────────────────────────
header "背压强度评分"

pass_rate=0
if [[ $TOTAL_CHECKS -gt 0 ]]; then
  pass_rate=$(( (PASSES * 100) / TOTAL_CHECKS ))
fi

echo ""
echo "  检查项  : $TOTAL_CHECKS"
echo "  通过    : $PASSES"
echo "  失败    : $ERRORS"
echo ""
echo -e "  ${BOLD}背压强度:${RESET}  ${PASSES}/${TOTAL_CHECKS} = ${pass_rate}%"
echo -e "  ${BOLD}熵积累率:${RESET}  $((100 - pass_rate))%"

echo ""
if [[ $pass_rate -ge 95 ]]; then
  echo -e "${GREEN}${BOLD}✓ 优秀 — 零熵或极低熵${RESET}"
elif [[ $pass_rate -ge 80 ]]; then
  echo -e "${YELLOW}良好 — 有少量漂移${RESET}"
else
  echo -e "${RED}警告 — 熵积累严重，运行 generate-agents.sh${RESET}"
fi
```

### 写入文件

```bash
mkdir -p skills/scripts
cat > skills/scripts/validate-skills.sh << 'LINTER_EOF'
#（粘贴上面脚本内容）
LINTER_EOF
chmod +x skills/scripts/validate-skills.sh
```

### 关键陷阱

⚠️ **不使用 `set -e` + `((var++))` 组合**
- macOS bash 3.2 中 `((0))` 返回退出码 1
- `set -e` 会在此时终止脚本
- **修复**：用 `var=$((var+1))` 或 `((var++)) || true`

⚠️ **R3 必须只检查表格引用**
- `#### \`skill-name\`` 这种 markdown 标题也会被 backtick 匹配
- **修复**：grep 时限定 `^[[:space:]]*|`，只在表格行中查找引用

⚠️ **R1 分组目录判断**
- `mlops/training/`、`mlops/inference/` 这种是**分组目录**（内有子 skill）
- 不要求自身有 SKILL.md，但子 skill 必须有
- **修复**：检测 `has_subs`，如果存在子目录且自身无 SKILL.md 则跳过

---

## Step 3：写生成器（C3）

### 目标
创建 `generate-agents.sh`，从实际目录结构自动生成 AGENTS.md。

### 原则
- **机器生成，人工不维护**：生成后的内容由脚本决定，禁止手工编辑
- **从目录结构投影**：skill 列表来自 `find` 命令，而非手写清单
- **消除手工漂移**：目录变了 → 运行生成器 → AGENTS.md 自动同步

### 脚本模板

```bash
#!/bin/bash
# =====================================================================
# generate-agents.sh — 从实际目录结构自动生成 AGENTS.md
#
# 原理：AGENTS.md 是目录结构 + SKILL.md metadata 的投影
#       由机器生成，保证与实际状态永远一致
# =====================================================================
set -euo pipefail

SKILLS_DIR="$HOME/.hermes/hermes-agent/skills"
cd "$SKILLS_DIR"

RED='\033[0;31m'; GREEN='\033[0;32m'; CYAN='\033[0;36m'
BOLD='\033[1m'; RESET='\033[0m'
pass() { echo -e "${GREEN}[GEN]${RESET}  $1"; }
warn() { echo -e "${RED}[WARN]${RESET}  $1"; }

# ── 从 SKILL.md 提取描述 ─────────────────────────────────────
get_description() {
  local skill_md="$1"
  if [[ ! -f "$skill_md" ]]; then echo ""; return; fi

  # YAML frontmatter 中的 description 字段
  if grep -q '^---$' "$skill_md" 2>/dev/null; then
    local desc=$(awk '/^---$/,/^---$/ {next} /^description:/ {sub(/^description:[[:space:]]*/, ""); print; exit}' "$skill_md")
    [[ -n "$desc" ]] && echo "$desc" && return
  fi

  # 回退：第一个非标题行
  sed -n '1,/^[#>]/p' "$skill_md" 2>/dev/null \
    | grep -v '^#' | grep -v '^>' | grep -v '^$' | grep -v '^---$' \
    | head -1 | sed 's/^[[:space:]]*//; s/[[:space:]]*$//'
}

# ── 分类标签 ─────────────────────────────────────────────────
get_category() {
  case "$1" in
    apple)          echo "Apple 系统" ;;
    creative)       echo "创意工具" ;;
    github)          echo "GitHub" ;;
    mlops)          echo "MLOps" ;;
    research)       echo "研究工具" ;;
    productivity)   echo "生产力" ;;
    software-dev*)  echo "软件开发" ;;
    autonomous*)    echo "AI Agent 调度" ;;
    *)              echo "工具" ;;
  esac
}

# ── 生成根 AGENTS.md（≤60行）───────────────────────────────
generate_root_agents() {
  local out="AGENTS.md"

  {
    echo "# Hermes Agent Skills — 智能体地图"
    echo ""
    echo "> 本文件由 \`generate-agents.sh\` 自动生成。"
    echo "> **核心原则：仓库即记录系统。**"
    echo ""
    echo "---"
    echo ""
    echo "## 快速路由"
    echo ""
    echo "| 任务 | 入口 |"
    echo "|------|------|"
    echo "| GitHub/PR/Issues | [github/AGENTS.md](github/AGENTS.md) |"
    echo "| MLOps | [mlops/AGENTS.md](mlops/AGENTS.md) |"
    echo "| 创意媒体 | [creative/AGENTS.md](creative/AGENTS.md) |"
    echo "| 研究工具 | [research/AGENTS.md](research/AGENTS.md) |"
    echo "| macOS 系统 | [apple/](apple/) |"
    echo "| 软件开发 | [software-development/](software-development/) |"
    echo "| AI Agent 调度 | [autonomous-ai-agents/](autonomous-ai-agents/) |"
    echo "| 效率工具 | [productivity/](productivity/) |"
    echo ""
    echo "---"
    echo ""
    echo "## 子地图索引"
    echo ""
    echo "| 目录 | 内容 |"
    echo "|------|------|"
    echo "| \`github/\` | PR/Issues/CodeReview (N skills) |"
    echo "| \`mlops/\` | 训练/推理/评估/云GPU (N skills) |"
    echo "| \`creative/\` | 媒体生成/可视化 (N skills) |"
    echo "| \`research/\` | 论文/博客/预测市场 (N skills) |"
    echo ""
    echo "> 加载 Skill：\`skill_view(name=\"category/skill-name\")\`"
    echo ""
    echo "*最后生成：$(date '+%Y-%m-%d %H:%M:%S')*"

  } > "$out"

  local lines=$(wc -l < "$out")
  if [[ $lines -gt 60 ]]; then
    warn "$out 为 ${lines}行（>60行限制）"
  else
    pass "生成 $out (${lines}行 ≤60行)"
  fi
}

# ── 生成子目录 AGENTS.md ────────────────────────────────────
generate_sub_agents() {
  local sub_dirs="github mlops research creative"

  for sub in $sub_dirs; do
    [[ -d "$sub" ]] || continue
    local out="$sub/AGENTS.md"
    local count=0

    {
      echo "# $(echo "$sub" | tr 'a-z' 'A-Z') — $(get_category "$sub")"
      echo ""
      echo "> 入口：[skills/AGENTS.md](../AGENTS.md)"
      echo "> 本文件由 \`generate-agents.sh\` 自动生成"
      echo ""
      echo "---"
      echo ""
      echo "## 快速路由"
      echo ""
      echo "| 任务 | Skill |"
      echo "|------|-------|"

      if [[ "$sub" == "mlops" ]]; then
        # mlops 有多级子目录（training/models/inference/...）
        for sub2_dir in "$sub"/*/; do
          [[ -d "$sub2_dir" ]] || continue
          sub2_name=$(basename "$sub2_dir")
          if [[ -f "${sub2_dir}SKILL.md" ]]; then
            desc=$(get_description "${sub2_dir}SKILL.md")
            echo "| $(echo "$desc" | cut -c1-50)… | \`${sub2_name}\` |"
            count=$((count+1))
          else
            for skill_dir in "${sub2_dir}"*/; do
              [[ -d "$skill_dir" && -f "${skill_dir}SKILL.md" ]] || continue
              skill_name=$(basename "$skill_dir")
              desc=$(get_description "${skill_dir}SKILL.md")
              # ⚠️ 统一用完整相对路径（带 sub2 前缀），便于 linter R3 验证
              echo "| $(echo "$desc" | cut -c1-40)… | \`${sub2_name}/${skill_name}\` |"
              count=$((count+1))
            done
          fi
        done
      else
        for skill_dir in "$sub"/*/; do
          [[ -d "$skill_dir" && -f "${skill_dir}SKILL.md" ]] || continue
          skill_name=$(basename "$skill_dir")
          desc=$(get_description "${skill_dir}SKILL.md")
          echo "| $(echo "$desc" | cut -c1-50)… | \`${skill_name}\` |"
          count=$((count+1))
        done
      fi

      echo ""
      echo "---"
      echo ""
      echo "## Skill 详情"
      echo ""

      if [[ "$sub" == "mlops" ]]; then
        for sub2_dir in "$sub"/*/; do
          [[ -d "$sub2_dir" ]] || continue
          sub2_name=$(basename "$sub2_dir")
          if [[ -f "${sub2_dir}SKILL.md" ]]; then
            desc=$(get_description "${sub2_dir}SKILL.md")
            echo "### \`${sub2_name}\`"
            echo "- **描述：** ${desc:-—}"
            echo ""
          else
            echo "### 📁 ${sub2_name}/"
            echo ""
            for skill_dir in "${sub2_dir}"*/; do
              [[ -d "$skill_dir" && -f "${skill_dir}SKILL.md" ]] || continue
              skill_name=$(basename "$skill_dir")
              desc=$(get_description "${skill_dir}SKILL.md")
              echo "#### \`${skill_name}\`"
              echo "- **描述：** ${desc:-—}"
              echo ""
            done
          fi
        done
      else
        for skill_dir in "$sub"/*/; do
          [[ -d "$skill_dir" && -f "${skill_dir}SKILL.md" ]] || continue
          skill_name=$(basename "$skill_dir")
          desc=$(get_description "${skill_dir}SKILL.md")
          echo "### \`${skill_name}\`"
          echo "- **描述：** ${desc:-—}"
          echo ""
        done
      fi

      echo "*最后生成：$(date '+%Y-%m-%d %H:%M:%S')*"

    } > "$out"

    pass "生成 $out ($count 个 skill)"
  done
}

# ── 主流程 ──────────────────────────────────────────────────
echo -e "${BOLD}━━━ AGENTS.md 生成器 ━━━${RESET}"
generate_root_agents
generate_sub_agents
echo ""
pass "完成 — 所有 AGENTS.md 已同步"
```

### 关键陷阱

⚠️ **mlops 多级子目录路径一致性**
- 路由表和详情中的 skill 路径必须一致
- 路由表用 `` `training/axolotl` ``，详情用 `#### \`axolotl\``（裸名）
- **结果**：linter R3 抓取表格的 `training/axolotl` 验证通过，但抓取详情的 `axolotl` 失败
- **修复**：统一生成器——详情也加前缀，或者路由表不加前缀
- **推荐**：路由表统一用全路径（如 `training/axolotl`），避免 linter 误报

⚠️ **YAML frontmatter 解析**
- SKILL.md 有 `---` 分隔的 YAML frontmatter 时，`description:` 字段在 frontmatter 内
- **修复**：用 awk 提取 `description:` 行，而非简单 grep

⚠️ **根 AGENTS.md 不要用 backtick 引用不存在的 skill**
- `[name](path/)` 链接形式不触发 linter R3 检查
- `` `skill-name` `` 会触发 R3 验证目录是否存在
- **推荐**：根 AGENTS.md 路由表全部用链接形式 `[text](path/)`，避免 R3 误报

---

## Step 4：Agent 可读性（C4）

### 目标
根 AGENTS.md 控制在 60 行以内。

### 原则
- **HumanLayer 规则**：文档超过 60 行，Agent 读取效果下降
- **约束越严，自主性越强**：缩小解空间，减少犯错概率
- **渐进式披露**：入口简洁，细节在子地图

### 操作

1. 运行 `generate-agents.sh`
2. 检查根 AGENTS.md 行数：`wc -l AGENTS.md`
3. 如果超过 60 行：精简为路由表格式，把详情移到子地图

### 60 行以内的模板

```markdown
# Hermes Agent Skills — 智能体地图

> 本文件由 `generate-agents.sh` 自动生成。

---

## 快速路由

| 任务 | 入口 |
|------|------|
| GitHub | [github/AGENTS.md](github/AGENTS.md) |
| MLOps | [mlops/AGENTS.md](mlops/AGENTS.md) |
| 创意 | [creative/AGENTS.md](creative/AGENTS.md) |
| 研究 | [research/AGENTS.md](research/AGENTS.md) |

---

## 子地图索引

| 目录 | 内容 |
|------|------|
| `github/` | 6 skills |
| `mlops/` | 22 skills |
| `creative/` | 9 skills |
| `research/` | 5 skills |

> 加载：`skill_view(name="category/skill-name")`

*最后生成：YYYY-MM-DD*
```

---

## Step 5：背压评分（C5）

### 目标
通过 linter 运行，确认背压强度达到 95% 以上。

### 操作

```bash
bash skills/scripts/validate-skills.sh
```

### 评分标准

| 分数 | 状态 | 行动 |
|------|------|------|
| 100% | 优秀 | ✅ 零熵，可以 push |
| 95-99% | 良好 | ⚠️ 有少量漂移，修复后再 push |
| <80% | 警告 | 🔴 熵积累严重，运行 generate-agents.sh 后再验证 |

### 修复低分的常见操作

```bash
# 1. 运行生成器同步 AGENTS.md
bash skills/scripts/generate-agents.sh

# 2. 创建缺失的 SKILL.md
mkdir -p skills/category/skill-name
echo "# skill-name" > skills/category/skill-name/SKILL.md

# 3. 修复引用路径（根 AGENTS.md 中不存在的 skill 引用）
# 用 [text](path/) 格式替代 `backtick` 引用
```

---

## Step 6：状态文档（C6）

### 目标
创建 `HARNESS.md`，记录 harness 组件状态。

### 模板

```markdown
# Hermes Agent Harness — 状态文档

---

## 2×2 Guides × Sensors 矩阵

| | 计算性（确定性） | 推理性（语义） |
|--|---------|---------|
| **引导器** | ✅ bootstrap脚本 | ✅ AGENTS.md ✅ Skills |
| **传感器** | ✅ validate-skills.sh | ⚠️ 待实现 |

---

## 引导器 — 已实现

- **AGENTS.md**：42行路由表
- **Skills**：82个渐进式知识包

---

## 传感器 — 已实现

- **validate-skills.sh**：5条规则，背压评分
- **generate-agents.sh**：自动同步 AGENTS.md

---

## 传感器 — 缺失（待实现）

- **AI Code Review**：提交后自动 LLM 审查
- **LLM-as-Judge**：评估 Agent 输出质量

---

## 背压强度

| 日期 | 分数 |
|------|------|
| YYYY-MM-DD | 100% |

---

## 快速命令

\`\`\`bash
# 验证
bash skills/scripts/validate-skills.sh

# 同步
bash skills/scripts/generate-agents.sh
\`\`\`
```

---

## 常见问题与修复

### Q1：linter R3 报告大量"引用无效"，但目录确实存在

**原因**：引用路径解析错误
- 根 AGENTS.md 引用 `apple-notes`，实际路径是 `apple/apple-notes`
- linter 在 `skills/` 下找 `apple-notes`，找不到

**修复**：
- 根 AGENTS.md 用链接格式 `[text](path/)` 而非 backtick
- 或者确保所有引用都相对于 skills/ 根目录

---

### Q2：linter R1 报告分组目录缺少 SKILL.md

**原因**：`mlops/training/`、`mlops/inference/` 这种是**分组目录**，内有子 skill
- 分组目录本身不需要 SKILL.md
- 但 linter 没有正确识别

**修复**：在 linter R1 逻辑中加入分组目录检测：
```bash
has_subs=$(find "$dir" -mindepth 1 -maxdepth 1 -type d 2>/dev/null \
  | grep -v '/templates$' | grep -v '/references$')
if [[ -n "$has_subs" && ! -f "$dir/SKILL.md" ]]; then
  total=$((total-1))
  continue  # 是分组目录，跳过检查
fi
```

---

### Q3：R3 把 markdown 标题中的 backtick 也当 skill 引用

**原因**：`grep -oE '`[a-z0-9-]+(/[a-z0-9-]+)*`'`` 会匹配所有 backtick 内容，包括 `#### \`skill-name\``

**修复**：只检查表格行中的引用：
```bash
refs=$(grep '^[[:space:]]*|' "$agents_file" 2>/dev/null \
       | grep -oE '`[a-z0-9-]+(/[a-z0-9-]+)*' \
       | tr -d '`' | sort -u)
```

---

### Q4：脚本在 macOS 上报 `timeout: command not found`

**原因**：macOS 没有 `timeout` 命令

**修复**：
```bash
# 不用 timeout，直接运行
bash scripts/validate-skills.sh

# 或用 gtimeout（需要 brew install coreutils）
brew install coreutils
gtimeout 30 bash scripts/validate-skills.sh
```

---

### Q5：`((var++))` 导致脚本提前退出

**原因**：`set -e` + bash 3.2 中 `((0))` 返回退出码 1，触发 `set -e` 终止

**修复**：
```bash
# ❌ 错误
((PASSES++))  # PASSES=0 时返回1，set -e 会终止

# ✅ 正确
PASSES=$((PASSES+1))
# 或
((PASSES++)) || true
```

---

### Q6：generate-agents.sh 把 YAML frontmatter 内容当描述

**原因**：SKILL.md 格式不统一，有的第一行是 `---` frontmatter
```yaml
---
description: This is the actual description
---
# SKILL.md Title
```

**修复**：用 awk 提取 frontmatter 中的 `description:` 字段：
```bash
desc=$(awk '/^---$/,/^---$/ {next} /^description:/ {
  sub(/^description:[[:space:]]*/, ""); print; exit}' "$skill_md")
```

---

### Q7：根 AGENTS.md 行数超过 60 行

**原因**：把太多 skill 详情塞进了入口文件

**修复**：
- 改为路由表格式，只列出子地图入口
- 详情全部移到子地图
- 删除空行和多余分隔线

---

### Q8：GitHub push 被拒绝

**原因**：SSH key 未认证或分支不存在

**修复**：
```bash
# 检查 SSH key
ssh -T git@github.com

# 如果失败，添加 SSH key
# GitHub → Settings → SSH Keys → 添加

# 确认分支存在
git branch -a

# 如果 feat-optional-skills 不存在，创建并推送
git checkout -b feat-optional-skills
git push -u origin feat-optional-skills
```

---

## 最终文件清单

```
skills/
├── AGENTS.md                    # 入口地图（≤60行）
├── HARNESS.md                   # Harness 状态文档
├── scripts/
│   ├── validate-skills.sh       # Linter（5规则，背压评分）
│   └── generate-agents.sh      # AGENTS.md 自动生成器
├── github/
│   └── AGENTS.md               # GitHub 子地图
├── mlops/
│   ├── AGENTS.md               # MLOps 子地图
│   ├── training/               # 训练相关 skills
│   ├── inference/              # 推理相关 skills
│   └── ...
├── creative/
│   └── AGENTS.md               # 创意工具子地图
├── research/
│   └── AGENTS.md               # 研究工具子地图
└── [category]/
    └── [skill-name]/
        └── SKILL.md            # 每个 skill 的定义文件
```

---

## 快速命令速查

```bash
# 1. 首次部署：运行 linter 看看现状
bash skills/scripts/validate-skills.sh

# 2. 同步 AGENTS.md（目录变更后必须运行）
bash skills/scripts/generate-agents.sh

# 3. 再次验证
bash skills/scripts/validate-skills.sh | tail -10

# 4. 提交所有变更
git add skills/
git commit -m "描述"
git push origin feat-optional-skills

# 5. 查看背压强度
bash skills/scripts/validate-skills.sh | grep 背压
```

---

## 部署检查清单

- [ ] `skills/scripts/validate-skills.sh` 存在且可执行
- [ ] `skills/scripts/generate-agents.sh` 存在且可执行
- [ ] 根 `skills/AGENTS.md` 存在且 ≤60 行
- [ ] 子地图 AGENTS.md 存在于 github/mlops/creative/research/
- [ ] 每个 skill 目录有 SKILL.md
- [ ] 背压强度 ≥95%
- [ ] `HARNESS.md` 状态文档已创建
- [ ] 所有变更已 git commit + push

---

*文档版本：v1.0 — 2026-04-16*
*来源：Hermes Agent Harness Engineering 实战*
