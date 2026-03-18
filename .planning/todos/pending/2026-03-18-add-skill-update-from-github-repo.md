---
created: 2026-03-18T15:13:01.896Z
title: Add skill update from GitHub repo
area: tooling
files:
  - .claude/skills/
---

## Problem

当前本地 skills 没有自动从远程仓库同步更新的机制。用户希望有一个新技能（skill），调用时能从 GitHub 仓库 https://github.com/Lylll9436/Paper-Polish-Workflow-skill 下载最新的 skills 文件并更新本地 `.claude/skills/` 目录中的对应文件。

## Solution

创建一个新的 skill（如 `update-skills-skill`），其 workflow 为：
1. 从 GitHub 仓库 `Lylll9436/Paper-Polish-Workflow-skill` 拉取最新的 skills 文件（可使用 `gh` CLI 或 `git clone/pull`）
2. 对比本地 `.claude/skills/` 与远程仓库中的 skill 文件
3. 更新本地的 SKILL.md 和相关文件到最新版本
4. 报告哪些 skill 被更新/新增
