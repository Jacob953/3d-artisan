# GitHub 连接器模式

没有可用 checkout/runtime 时，现有 GitHub 连接器仍可完成文本读取、原文定位和学习记录持久化；无需新增 MCP。明确说明当前使用连接器模式，不能声称执行了脚本或 hooks。

## 渐进读取

先取得目标分支头 SHA，固定本轮读取到该提交。读取 shapeshifter.yaml、AGENTS.md、入口规则、state/reading.yaml 和当前书 state.yaml。只加载当前动作需要的规则与记录。source/manifest.yaml 的 prepared.catalog 存在时，读取目录头及一页候选，再读取选择的 source/index/ID.yaml。核对 source_id、fingerprint。机械片段可能跨越一个完整思想，必要时沿 previous/next 补充，不能直接当成完整教学单元。未读原文不得出题后假称来自该书。

## 记录与保存

遵守 data.md 和 interaction.md 的格式、题目快照、帮助条件、只追加原始回答及书级 revision 约束。阅读和讲解不需要 capability/task；仅真正开始练习时建立有依据的目标，不杜撰掌握程度。连接器同样遵守 study/read_complete/end_activity 的状态与帮助条件，不得因不能运行脚本退回“先建题”。仅更新本次实际改变的对象，不改无关数据。

有可用 Python 和真实插件文件时优先运行校验和写入工具；否则 agent 按仓库契约构造记录，明确没有执行脚本校验。多文件学习变更必须一起创建 Git tree/commit，以观测到的分支头为 parent，再 non-force 更新引用。单文件检查点可用 Contents API 的 expected blob SHA。提交前重读分支头和书级 revision；并发改变则重新读取并协调，不能静默覆盖或强推。缺少必要的原子写入或冲突检测工具时停止学习写入，保留可审查的待提交内容并报告未同步。

提交后从返回 commit SHA 回读受影响文件，核对内容和远端分支包含该提交，再报告 synced、目标分支、提交 SHA。只保存本轮检查点所需内容，不每次交流都写 GitHub。首次提供阅读材料不要求先写能力和题目；需保存断点时，将实际活动合并为一个检查点。显式测试分支覆盖默认 sync.branch，但不得改写主分支配置。

恢复检查点使用 checkpoint.md；真实学习证据仍在 practice/ 等既定目录。来源准备仅建立可检索原文，不自动标记 read_ranges、integrated_units 或 mastery。
