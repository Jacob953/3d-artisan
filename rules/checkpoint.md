# 可跨端恢复的检查点

检查点是工作交接摘要，不替代书级 state、practice 原始回答或知识来源。普通讨论不创建；学习暂停、跨端交接或诊断确有进展时创建。格式固定为 `state/checkpoints/ID.json`。每个文件最多 16KB；详细证据引用文件，不复制整段历史。恢复时先看书级状态，需诊断或交接时再按文件名定位最近相关检查点，列表分页，不全量重放。

必填字段：

- format_version: 1；id：小写连字符 ID；kind：learning / synthetic_test / diagnostic。
- repository、branch、base_commit：这次读取的真实仓库、目标分支和完整 40 位分支头。created_at：实际当前时间，带时区。
- checks：非空列表；每项 name、status（pass / fail / not_run）、detail、evidence（路径或具体执行结果列表）。pass 必须有证据；fail 写明实际错误；not_run 写明前置阻塞。不要把“没有测”写成失败或成功。
- learning：user_answer_recorded、mastery_changed（布尔值）、evidence（持久化证据路径列表）。当 kind 为 synthetic_test 或 diagnostic 时，user_answer_recorded 与 mastery_changed 必须为 false，learning.evidence 为空；不得记录为真实掌握。
- next_action：足以执行的下一步，说明待获取的资源或待解决的阻塞，不写空泛的“继续”。

有执行环境时使用插件根目录 `scripts/checkpoint.py FILE.json` 校验；否则按这些同样的字段检查并明确没有运行校验器。保存后回读文件，并核对远端提交和分支头；提交 SHA 放在回复的保存回执中，不写进尚未产生该提交的文件。工具验证结构，agent 判断证据是否支持结论。
