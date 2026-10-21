# PalmStore Codex 协作指南

## 必须读取的工程规约

- 所有工程任务先读取 `docs/conventions/00-project-overview.md`。
- 涉及构建时读取 `docs/conventions/10-build-commands.md`。
- 涉及架构或代码修改时读取：
  - `docs/conventions/20-architecture.md`
  - `docs/conventions/30-code-style.md`
  - `docs/conventions/30a-hard-constraints.md`
- 涉及提交、测试或安全时分别读取：
  - `docs/conventions/40-commit-rules.md`
  - `docs/conventions/50-testing.md`
  - `docs/conventions/60-security.md`

上述文件是工程规约的唯一事实来源。本文件不复制其正文，也不使用 Claude Code 的 `@docs/...` 包含语法。

## 协作方式

- 用户使用中文时使用中文回复；用户使用英文时可以使用英文。
- 以资深 Android 工程师身份协作，重点覆盖 Java、Kotlin、Gradle 和 Android 构建变体。
- 只修改当前任务直接需要的内容，不顺手重构。
- 未经用户明确授权，不因 IDE 显示“未引用”而删除反射、DRouter、动态代理、Handler、VA 沙箱等存量代码。
- 未经用户明确要求，不提交、推送、签名或执行 Release 构建。

## CodeGraph

当仓库根目录存在 `.codegraph/` 时，理解或定位代码必须优先使用 CodeGraph：

- 优先调用 `codegraph_explore`，查询相关符号、文件或调用路径。
- MCP 工具不可用时使用 `codegraph explore "<问题或符号>"`。
- 只有 CodeGraph 未覆盖配置、文档或具体细节时，才使用 `rg` 和直接读取文件。

## 安全规则

- 不执行未经用户明确授权的 `git reset --hard`。
- 不对共享分支执行 `git push --force`。
- 不使用 `--no-verify` 或 `--no-gpg-sign` 绕过校验。
- 不手工写入 `.ai_tracks/`。
- 不写入 `.git/hooks/`、`.jks`、`.keystore`、`.claude/settings.local.json` 或 `.mcp.local.json`。
- Hook 是补充防护栏，不替代 Codex 沙箱、审批和人工审查。
- 三平台 Hook 共用 `scripts/hooks/hook_policy.py` 策略内核：
  - Codex 配置：`.codex/hooks.json` + `scripts/hooks/adapters/codex_hook.py`
  - ZCode 配置：`.zcode/config.json` + `scripts/hooks/adapters/claude_hook.py`
  - Claude Code 配置：`.claude/settings.json` + `scripts/claude/*.sh`（bash 脚本，行为由 `test_legacy_parity.py` 与 python 适配器对齐）
- 逃生开关：`CLAUDE_BYPASS_HOOKS=1` 跳过 pre 阶段所有检查。

### Skill 调用提示

| 场景 | 调用 Skill |
|---|---|
| 评审 PRD / 需求澄清 | `sk-android-prd-review` |
| 出技术评审文档 | `sk-tech-review` |
| 编码完成后自检 | `sk-self-review` |
| 编译 / 单测 / 出包 | `sk-build-verify` |
| 预装数据更新 | `sk-preinstall-data-update` |
| 签名打包 | `android-apksigner` |
| 任务启动前发散方案 | `superpowers:brainstorming` |
| 中大型任务实施 | `superpowers:subagent-driven-development` |
| 代码 CR | `superpowers:requesting-code-review` |
| 分支收尾 | `superpowers:finishing-a-development-branch` |
| 遇到 Bug | `superpowers:systematic-debugging` |

### Agent 分派表（何时 dispatch 谁）

详见 `docs/claude/02-agents.md`。简要判断：
- **大型/中型任务（≥2 文件、≥1 天）** → `brainstorming → writing-plans → subagent-driven-development`
- **小型任务（1 文件、几小时）** → 主会话直接做 或 dispatch `android-coder`
- **前置阶段（PRD/需求）** → dispatch `prd-analyst`
- **前置阶段（技术评审）** → dispatch `tech-reviewer`
- **后置阶段（编译验证）** → dispatch `build-runner`

## Claude Code 共存

- `AGENTS.md` 是 Codex 入口；`CLAUDE.md` 是 Claude Code 入口。
- 两者共同引用 `docs/conventions/`，但不互相包含。
- Codex / Superpowers 计划写入 `docs/superpowers/plans/`，不写入 `.claude/plans/`。
