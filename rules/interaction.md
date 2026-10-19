# 对话入口与可靠操作

用户只需要说开始学习、继续学习、查看进度，或直接提出问题。不要要求用户填写 JSON、记住文件路径或手动管理阶段。先按意图选择当前动作；阶段只是当前交互位置，不能作为必须完成的教学流水线。

## 会话接入

hook 提供 project、session_id、turn_id 和仓库规则入口。运行下面的交互工具时传入这些值：

```sh
python scripts/interact.py TARGET --book BOOK --session SESSION --turn TURN --input REQUEST.json
```

脚本由插件提供，位置以 inspect 返回的 runtime 为准。首次真正开始学习用 `{"action":"attach"}`；普通仓库编辑不要 attach。attach 返回断点，不自动生成目标或改变掌握判断。不同会话独立绑定书籍，切书前处理旧回合。没有 hooks 时可省略 session/turn，手动操作仍可运行，但不得声称已启用结束检查。

只看进度可使用 context resume/select。普通维护、纯查询不创建待保存义务。实际学习操作开始写入时才登记待处理轮次；失败后可重试，或 acknowledge deferred 明确保留未完成状态。不要把实际学习回答一律归为 read_only 来跳过记录。若上一回合中断未处理，status 查明 pending_turn，先保存已有证据或明确 deferred，再 attach 当前回合。不得把新问题当旧题答案。

## 学习操作

除 attach/status/acknowledge 外，请求均含 `operation_id`（稳定的唯一小写 ID）、`expected_revision`（最近读取的书级版本）。同一次操作重试沿用同一 ID 与完整请求，换内容必须换 ID。回执与学习变更一起纳入可恢复保存，重复操作不会重复追加；回执保存在书级 .receipts/，不加载进学习上下文。

| action | 额外输入 | 结果 |
|---|---|---|
| study | mode（reading / explanation / demonstration）、source_units、text（实际阅读指引或讲解） | 不依赖能力或题目，记录已提供材料；不记为已读 |
| read_complete | text（用户实际阅读完成报告） | 记录用户报告的阅读范围，不评价掌握 |
| end_activity | reason | 解除当前活动；未完成练习标为 cancelled，保留历史，不评价能力 |
| begin | capability、task、prompt；可选 practice、purpose、source_units、reference_allowed、prior_help | 仅用于练习，固定题目与评价标准快照，等待回答 |
| answer | text 为用户真实回答 | 追加回答，依据已给帮助标记条件，进入待反馈 |
| hint / explain | text 为实际提供的帮助；await_response 可选 | 记录提示/示范；解释默认不要求答题，提示默认允许再次尝试 |
| assess | assessment 含 observations、hypotheses、result、evaluator；complete 可选 | 保存评价，可结束练习，不自动提升能力状态 |
| pause | 无 | 暂停，保留当前任务及证据 |
| continue | 无 | 恢复当前尚未完成的交互 |

study 可以在没有 blueprint 能力和练习时使用；activity 只保留当前单元的指引或讲解及来源，不复制全书或累计聊天。首次给材料不要求先调用写入接口；在需要保存阅读断点时记录实际已提供的内容。read_complete 只能依据用户实际报告，不能因工具读取成功调用。study 前如有旧练习，用 end_activity 明确结束，不删除旧回答或用它推断不足。原文机械片段须先拼成合理阅读范围。

begin 不是“开始学习”的通用入口。仅在用户请求练习或已有足够准备时使用；先读 learn.md 判断。前一 study 的相关讲解会自动作为帮助加入新练习；同轮先讲答案再建题时必须传 prior_help，不能声称独立判断。begin 的 source_units 应包含真正相关的材料，不能省略它来规避帮助记录。

回答先保存，再选择帮助。不需要每个动作都出现；可直接解释、跳过、暂停或更换活动。独立能力的新检查应另起任务，不清除旧提示来制造独立证据。begin 前需已有对应 capability/task，使用 context save 按 ID 建立；这一步由 agent 根据用户目标与资料准备。知识和 profile 仍使用 context save，不能通过 hooks 自动推断。

## 帮助的选择与停止

读取 select.md 的候选原则，结合任务性质、已有知识、用户当前意图和表现选择解释、示范、回忆、对比或新任务。表达不清、问题歧义和知识不足是不同假设；必要时用一个有辨别力的问题核查。不设固定问题数、提示次数或复习间隔，不因一次错误建立全局规则。用户明确要求答案时直接解释并如实记录帮助，不强迫完成测试。

Stop 只检查已开始的写入操作是否有处理回执，而不是是否所有学习目标完成。等待用户回答是正常终点。缺失处理最多续跑一次，随后提示未保存风险；不得循环追问或自动编造用户回答。这个检查依赖 hooks 被宿主执行、会话 attach 和正确的用途判断，不是防绕过的强制边界。

## 持久化与跨端

有新证据或进度变化时及时保存工作目录；纯讨论不强制落盘。完成一个学习单元、暂停、结束本次学习或切换设备前，使用插件 sync 技能按仓库 sync 配置建立检查点并推送。用户要求立即保存到远端时立即同步。不要把本地保存回执称为已同步；只有远端分支验证包含提交后才报告 synced。失败保留本地变更并明确说明不能从另一端恢复最新内容。新设备开始前刷新远端；有本地变更或分叉时先解决冲突，禁止强推。
