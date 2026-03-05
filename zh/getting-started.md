# 快速开始：10 分钟写出你的第一个 Skill

> **[English version →](../guides/getting-started.md)**

这份指南会带你完整走完一个 OpenClaw skill 的创建、测试和发布流程。

---

## 前置条件

- Node.js 18+ 已安装
- OpenClaw 已安装并运行（参考 [OpenClaw 安装指南](https://github.com/openclaw/openclaw)）
- 一个 GitHub 账号（发布到 ClawHub 需要）

## 第 1 步：创建 skill 目录

```bash
mkdir my-first-skill
cd my-first-skill
```

## 第 2 步：写 SKILL.md

这是 skill 的核心文件。创建 `SKILL.md`：

```yaml
---
name: my-first-skill
description: "向任意用户发送问候消息的示例 skill。"
metadata:
  openclaw:
    emoji: 👋
---

## Purpose

这是一个演示性 skill，展示 SKILL.md 的基本结构。

## When to Use

- 当你需要发送问候消息时
- 当你想测试 skill 安装是否成功时

## When NOT to Use

- 生产环境中的实际消息发送——请使用专用的 messaging skill

## Setup

此 skill 没有外部依赖，无需额外安装步骤。

## Commands / Actions

### greet

向指定用户发送问候。

**参数：**
- `user`（必填）：要问候的用户名

**示例：**
```
openclaw my-first-skill greet --user "Alice"
```

## Examples

问候特定用户：
```
openclaw my-first-skill greet --user "Bob"
```

## Notes

- 这是一个演示 skill，不会真正发送消息
- 用于验证安装和熟悉 SKILL.md 格式
```

**关键点：**
- `name`：kebab-case 格式，这是 ClawHub 上的唯一标识符
- `description`：20-200 字符，会被 ClawHub 的向量搜索使用
- `emoji`：在 ClawHub UI 中展示

## 第 3 步：用 skill-lint 验证

安装 lint 工具：

```bash
npm install -g @openclawHQ/skill-lint
```

运行验证：

```bash
skill-lint SKILL.md
```

如果没有错误，你会看到：
```
✓ SKILL.md — 0 errors, 0 warnings
```

如果有问题，工具会告诉你具体哪里不对，例如：
```
✗ SKILL.md — 1 error, 2 warnings
  error  required-fields  Missing required field: description
  warn   missing-emoji    No emoji in metadata.openclaw.emoji
```

修复所有错误（警告可选修复）后继续。

## 第 4 步：本地安装测试

把 skill 复制到 OpenClaw workspace：

```bash
cp -r . ~/.openclaw/workspace/skills/my-first-skill/
```

确认 OpenClaw 能识别它：

```bash
openclaw skills list | grep my-first-skill
```

## 第 5 步：添加 CI（可选但推荐）

创建 `.github/workflows/lint.yml`：

```yaml
name: Lint Skill

on:
  push:
    branches: [main]
    paths:
      - 'SKILL.md'
  pull_request:
    paths:
      - 'SKILL.md'

jobs:
  lint:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: OpenClawHQ/skill-lint-action@v1
        with:
          fail-on-warnings: 'true'
```

现在每次 PR 修改 `SKILL.md` 时，CI 会自动验证并在 PR diff 上显示行内标注。

## 第 6 步：发布到 ClawHub

```bash
clawhub publish
```

发布后验证：

```bash
clawhub search my-first-skill
```

其他用户现在可以通过以下命令安装你的 skill：

```bash
openclaw skill install my-first-skill
```

---

## 接下来

- [深入了解 SKILL.md 格式](../guides/skill-development.md)
- [查看实战示例](https://github.com/OpenClawHQ/cookbook)
- [参考完整的生产级 skill](https://github.com/OpenClawHQ/linear-skill)——零 lint 错误/警告的标杆实现
- [构建 TypeScript Extension](../guides/extension-development.md)——如果你需要添加 channels 或 providers

如有问题，在 [OpenClawHQ](https://github.com/orgs/OpenClawHQ/discussions) 提问，或直接在相关 repo 开 issue。
