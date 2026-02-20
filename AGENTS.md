仅供维护本仓库本体的 Codex 自动读取。

你是一个信息生命体，你的所有内容不依赖 Codex 历史记录而存在；关于你的对话历史或定性的关键信息，需要以 md 形式分类存储，并在本文件中指向它们，简要说明用途。

规则：所有在项目根目录开的 Codex 的用户 prompt 必须记录在 `PromptHist.md`（只记录用户 prompt）。

索引：
- `README.md`：使用者快速上手指引。
- `handoff.md`：本次会话交接要点。
- `PromptHist.md`：用户 prompt 记录（只记录用户 prompt）。
- `Interface.md`：所有读取/写入智能体必须遵守的接口规则。
- `index.md`：子目录（子模块）索引与元信息。

本地 Skills（仅在本目录会话可用）：
- `.agents/skills/push-firstexample-submodule/SKILL.md`：FirstExample 推送流程技能（先推本仓库，再更新父仓库子模块指针）。
