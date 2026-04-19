# Win 场景可复现 Prompt 模板（cmd/powershell + 新手小白 + SSH 推送）

## 0. 提炼范围

本模板由以下历史文本共性提炼：

- `S:\阿里云盘\SurfaceBook2\S\Data\temp\RbSv\Bi\20260403_080706_974.txt`
- `S:\阿里云盘\SurfaceBook2\S\Data\temp\RbSv\20260331_034435_160.txt`
- `S:\阿里云盘\SurfaceBook2\S\Data\temp\RbSv\20260409_234609_687.txt`
- `S:\阿里云盘\SurfaceBook2\S\Data\temp\RbSv\20260327_220422_307.txt`
- `S:\阿里云盘\SurfaceBook2\S\Data\temp\RbSv\20260419_223956_102.txt`

共性关键词：`新手小白可复现`、`cmd.exe/powershell.exe`、`写入新md`、`git@github...`、`ssh推送`。

---

## 1. 模板 A（完整版，直接可投喂 Codex）

```text
请用 cmd.exe/powershell.exe 在 Windows 环境执行，不要用 Linux 命令替代关键安装/编译步骤。

任务主题：【{{TASK_THEME}}】

请访问目录：{{SOURCE_DIR}}
搜寻并提炼与“新手小白可复现”相关的 prompt 模板，重点覆盖：
1) 任务目标定义
2) 执行边界（必须 cmd.exe/powershell.exe）
3) 路径与目录约束（优先放 {{WIN_WORK_DIR}}）
4) 交付物结构（结论->路线->原理）
5) Git 写入与 SSH 推送流程

目标仓库：{{TARGET_REPO_SSH}}
目标产物：写入一个以 `win` 开头的新 md（如 `win_{{ID}}_{{TOPIC}}.md`）

写作要求（必须全部满足）：
1) 主次分明：先给战报结论，再给执行路线，再讲原理。
2) 全流程覆盖：从 `{{START_STATE}}` 到 `{{END_STATE}}`。
3) 内容完整：上下文、绝对路径、目录结构、脚本/命令、参数说明、失败回滚点。
4) 新手友好：每一步可直接复制执行，并注明预期输出与常见报错。
5) 架构可理解：附 GitHub/Typora 可渲染 Mermaid 图。
6) 解释深度：既讲“怎么做”，也讲“为什么这样做”。

Git 执行要求：
1) clone/pull {{TARGET_REPO_SSH}}
2) 写入 `win` 开头新 md
3) git add/commit
4) git push（SSH）
5) 返回提交哈希与文件路径
```

---

## 2. 模板 B（精简版，适合快速派单）

```text
用 cmd.exe/powershell.exe 访问 {{SOURCE_DIR}}，提炼“新手小白可复现”的 prompt 模板，写到 {{TARGET_REPO_SSH}} 的 `win` 开头新 md，并通过 SSH 完成 git push。

要求：
1) 文档结构：结论 -> 路线 -> 原理
2) 必含：绝对路径、命令、参数、预期输出、报错排查
3) 必含：从 {{START_STATE}} 到 {{END_STATE}} 的完整复现链路
4) 返回：新 md 文件名 + commit hash
```

---

## 3. 模板 C（偏教学讲解版）

```text
请站在“教师指挥官”角度，统筹全局安排分析，输出一份新手小白可一步步复现的 Win 实战文档。

执行环境限制：
- 所有关键执行统一走 cmd.exe/powershell.exe
- 文件编辑可在 WSL，但安装/编译/运行在 Windows
- 可放 {{WIN_WORK_DIR}} 的都放该目录，实在不能放再落 C 盘

交付：
- 产出到 {{TARGET_REPO_SSH}} 的 `win_*.md`
- 完成 SSH 推送
- 给出“可复用 prompt 模板 + 一次真实命令演示”
```

---

## 4. 变量清单（填写即用）

- `{{SOURCE_DIR}}`：待检索原始对话/文本目录（例：`S:\阿里云盘\SurfaceBook2\S\Data\temp\RbSv`）
- `{{TARGET_REPO_SSH}}`：目标仓库 SSH 地址（例：`git@github.com:ShengNW/SnwHist_FirstExample.git`）
- `{{WIN_WORK_DIR}}`：Windows 侧工作目录（例：`D:\exe\SNW_GitLab`）
- `{{START_STATE}}`：起点状态
- `{{END_STATE}}`：终点状态
- `{{ID}}/{{TOPIC}}`：文档编号与主题

---

## 5. 推荐文件命名

- `win_004_cmd_ps_reproduce_prompt_templates.md`
- `win_005_<你的主题>.md`

