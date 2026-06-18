# Calicat Agent Skills

[English Version](./README.en.md)

这是一个兼容 Vercel Skills 生态的 Calicat Agent Skills 仓库，用来分发与 Calicat 相关的 AI Skill。

用户可以通过 `npx skills add` 直接从本地目录或 GitHub 安装这些 Skill。

## 安装

查看仓库中有哪些 Skill：

```bash
npx skills add ./calicat-agent-skills --list
```

本地开发时从本地路径安装：

```bash
npx skills add ./calicat-agent-skills --skill calicat-cli-operator
```

仓库发布到 GitHub 后，从 GitHub 安装：

```bash
npx skills add calicatcn/calicat-agent-skills --skill calicat-cli-operator
```

给 Codex 全局安装：

```bash
npx skills add calicatcn/calicat-agent-skills --skill calicat-cli-operator -g -a codex -y
```

给 Claude Code 全局安装：

```bash
npx skills add calicatcn/calicat-agent-skills --skill calicat-cli-operator -g -a claude-code -y
```

## 当前可用 Skill

### `calicat-cli-operator`

这个 Skill 用来指导 AI 通过本地 Calicat CLI 完成以下任务：

- 正确解析 Calicat 设计链接
- 识别 `file_id`、`canvas_id`、`selected_layer_id`
- 渐进式获取画布、页面、图层结构
- 获取 PRD 数据和交互设计数据
- 避免一次性加载过大的设计 JSON
- 在需要时检查 CLI 版本、更新 CLI、或指导用户安装 CLI

## 仓库结构

```text
skills/
  calicat-cli-operator/
    SKILL.md
    references/
      cli-lifecycle.md
```

## 说明

- 这个仓库遵循 `npx skills` 使用的开放 Skill 仓库结构。
- Skill 本身是被动上下文，不是独立程序；AI 会在任务匹配时加载它。
- CLI 的安装、更新、版本检查等低频内容被放在 `references/cli-lifecycle.md` 中，以便按需渐进式加载。
