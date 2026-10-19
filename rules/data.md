# 数据与保存

每书独立：blueprint.yaml 保存 capabilities 列表（id、description、priority、knowledge、tasks）；每个 task 有 id、description、variations、criteria。learner.yaml 保存 capabilities 映射，各能力可有 independent / with_support / transfer / delayed_retention（status、evidence），gaps（description、evidence）。无证据保持 untested / insufficient_evidence。二者 format_version=1。

knowledge/ID.md 使用 YAML frontmatter：id、origin（source_based / inference / observation）、sources（source_id、locator、fingerprint）、related。正文按实际需要保存解释、方法、适用条件和例子。

source/index/ID.yaml 是单元：id、title、source_id、fingerprint、locator、text。text 只保存该语义单元，locator 指向原书。原书与可还原原文默认不提交 Git；用户明确要求跨端访问且目标仓库允许时，可显式跟踪准备后的原文。manifest 的外部路径不由上下文工具自动打开。机械准备工具生成的片段带 segmentation=mechanical_fragment，不能把材料已准备等同于已阅读或掌握。

practice/ID.yaml：id、capability、task、purpose（training / assessment）、status（in_progress / completed / cancelled）、created_at（ISO 时间）、task_snapshot（prompt、criteria、blueprint_revision）、source_units（单元 ID 列表）、turns（按发生顺序，每项 kind、text、assistance、at）、assessment（可选；observations、hypotheses、result、evaluator）。题目与标准快照发出后不改，turns 只追加。取消记录带 cancellation（reason、at），不代表错误或掌握不足；完成或取消记录不覆盖；更正用新记录并通过 corrects 引用原记录。每次回答/提示立即保存，不能等练习结束才持久化。

state.yaml 保存 book_id、format_version、revision、当前 capability/unit/practice、phase、pending、read_ranges、integrated_units、next_action。pending 为 null 或 {practice: ID}，完整题目保存在 practice。可选 activity（旧仓库缺失等同 null）保存当前非练习活动：mode（reading / explanation / demonstration）、status（provided / reported_read）、source_units、text、at；reported_read 必须带用户原话 user_report，且只用于 reading。activity 与 current_practice/pending 互斥。read_complete 在 read_ranges 追加 {source_units, user_report, at}，不改变 integrated_units 或 learner。读过的范围与已整合范围分开。不得凭空生成学习证据。

通过 `scripts/context.py TARGET save --book BOOK --input PATCH.json` 保存；格式为：

```json
{"expected_revision": 0, "updates": [{"kind": "state", "data": {"next_action": "establish_goal"}}]}
```

支持 capability / profile / practice / unit（id 与 data 映射），knowledge（id 与 content 字符串），state（data 为仅需改变的字段）。capability/profile 按 ID 在磁盘侧合并进 blueprint/learner，禁止让模型整份替换档案；inspect capability 返回该能力及对应 profile。state 的 revision 由工具递增，不手写。一次只提交需要改变的对象。先 inspect 获取相关当前值；同一书级 revision 防止旧读取覆盖新写入。工具检查引用、练习原始回合保留、文件路径和结构；语义评价仍由 agent 负责。

保存采用单写者锁、可恢复的未完成写入标记与修改前备份。多文件不是断电原子提交。工具发现中断时拒绝继续读写，使用 `recover --book BOOK` 校验磁盘内容后回滚未完成批次，再重试；遇到第三方变更停止，不强行覆盖。完成后只返回 revision 和变更路径，避免把整个档案重新输出。

这些文件是事实来源。查询索引只能是可重建缓存；当前使用磁盘扫描与分页，无向量检索、自动最优调度或全历史事件重放。输出有界不代表会话历史自动清除。

日常问答优先使用 interaction.md 的明确操作接口，自动生成时间、帮助条件和幂等回执；context save 保留给知识、能力目标、profile 等细粒度维护。.receipts/ 为操作去重记录，不是全历史事件源。
