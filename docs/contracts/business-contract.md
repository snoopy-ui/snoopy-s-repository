# StudyCheck 首轮业务契约

状态：A v1.1，2026-10-07。共享结构变更由 A 发起说明，B/C/D/E 评审后再合并。需求 PDF v1.0 是评审草案，未确认的政策必须保持可配置。

## 标识与版本

| 字段 | 要求 |
|---|---|
| `school_id` / `course_id` / `course_offering_id` | 稳定归属标识；标题改变不改 ID |
| `assignment_id` / `task_version` / `rubric_version` | 任务、任务规则和量规版本；发布后变更递增 |
| `student_id` / `member_id` | 服务端身份标识；姓名不能做主键 |
| `audience` | 学校、专业、年级、班级或成员范围；服务端校验 |
| `attempt_id` | 一次可信视频完成后创建的限时会话；同一任务至多一个正式会话 |
| `revision_id` | 会话内草稿/修改版本；每轮自评、反馈和修改说明均绑定 |
| `submission_id` | 一次收件请求的幂等键；网络重试沿用同一值 |
| `submission_version` | 修改重交版本；新版本不能延长旧会话 |
| `started_at` / `deadline_at` | 服务端记录的会话起点和截止；`deadline_at = started_at + 1200s` |
| `content_sha256` | 冻结正文、附件和清单的内容指纹；收件时校验 |

任务发布还必须声明 `open_at`、`close_at`、必看/可选视频、必交材料、视频-作业关系、教师、帮助/例外/重测政策和公布范围。学生正式开始后锁定这些规则。

## 会话计时不变量

1. 必看视频产生可信完成事件后，服务端原子创建 `attempt_id` 并写入 `started_at`；无视频任务在进入作答时执行同一原子创建。
2. 有效写入区间是 `[started_at, deadline_at)`；服务端拒绝截止时刻及之后的新修改。
3. 回看、重启、换设备、改系统时间、自查重试和 AI 等待不重置或无限暂停计时。
4. 到时冻结最后一个合法 `revision_id`；冻结 worker 延迟、重复事件和晚到稿件都不能改写快照。
5. 旧 worker 不能写入新 `task_version` 或新 `revision_id`；新任务规则不能覆盖已开始会话。

## 状态语义

学习和作业状态：`assigned` → `learning` → `ready_to_start` → `in_progress` → `ai_waiting` → `frozen` → `pending_upload` → `received` → `pending_review` → `returned`/`resubmitted` → `published`。

异常状态：`download_temp_failed`、`download_permanent_failed`、`check_pending_manual`、`upload_failed`、`evaluation_pending`、`appeal_hold`。异常状态必须携带原因、影响范围和下一步操作。

`draft` 表示可继续编辑的草稿；`frozen` 表示只读快照；`check_passed` 只表示当前版本通过检查；`received` 只表示云端收件并有回执；`published` 才表示教师已发布结果。任何一个状态都不能替代另一个状态。

## 提交与重试

客户端发送冻结快照、身份/任务/量规版本、`attempt_id`、`revision_id`、`submission_id`、`submission_version`、观看摘要、自评、AI 反馈、修改说明和文件清单。服务端以 `submission_id` 幂等：同 ID 同内容返回原回执，同 ID 不同内容返回冲突并拒绝写入。超时先查询原 ID，不创建新收件记录。修改重交递增 `submission_version`，旧正文、回执和教师意见只读保留。

## 人工复核与评价边界

自查服务不可用、结果晚到、旧稿反馈或原件/哈希异常时，返回技术状态并进入 `check_pending_manual` 或 `evaluation_pending`，不能生成合格结论。人工复核必须记录复核人、时间、结论、依据和覆盖理由；AI 不能修改身份、期限或正式成绩。云端评价只能读取冻结原件和可信量规，无依据时标记待复核。

## 下载边界

视频数量为 0 时状态为 `无需下载`，作答入口可用。单个平台或单项视频失败必须只影响该项，其他资源、笔记和进度保留；失败需区分临时/永久并提供单项重试或教师批准的替代材料。不能把错误页面保存为视频，也不能用下载状态阻塞无视频任务。

## 验收责任

- A：确认课程规则、量规版本、状态语义、发布范围、待评审决策和 AC 追踪。
- 甲（B）：验证学生界面、无视频/下载失败、草稿恢复、重交反馈和状态展示。
- 乙（C）：验证身份授权、提交幂等、回执、原件哈希和历史版本。
- 丙（D）：验证确定性检查、AI 结果 Schema、技术故障分类和人工复核语义。
- 丁（E）：按契约建立可复现测试；`PASS`、`FAIL`、`BLOCKED`、`SKIP`、`CLEANUP` 分开记录，未执行不得标为通过。
