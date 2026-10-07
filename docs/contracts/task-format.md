# 课程任务 Markdown 格式契约

状态：A v1.0，2026-10-07。此文件约束样例任务和后续解析器/云端映射；正式量规和教师政策仍需评审。

## 必须字段

```yaml
school_id: SCHOOL-DEMO-01
course_id: COURSE-DEMO-01
course_offering_id: OFFERING-DEMO-2026-FALL
assignment_id: TASK-003
task_version: 1
rubric_version: RUBRIC-DEMO-1
title: 任务标题
open_at: 2026-10-07 09:00
close_at: 2026-10-14 18:00
duration_seconds: 1200
audience: SCHOOL-DEMO-01/CLASS-DEMO-01
teacher: TEACHER-DEMO-01
required_videos: 0
optional_videos: 0
help_policy: 待评审
exception_policy: 待评审
retest_policy: 待评审
```

字段含义：

- `school_id`、`course_id`、`course_offering_id`、`assignment_id` 是归属链；不能用标题或学生姓名替代。
- `task_version` 和 `rubric_version` 必须随规则/量规改变递增。学生开始会话后锁定这两个版本。
- `open_at`、`close_at` 是任务发布窗口；`duration_seconds` 首轮固定为 `1200`，除非教师批准其他规则。
- `deadline` 是当前 Personal 解析器使用的兼容别名，值应与 `close_at` 一致；云端契约以 `close_at` 为准。
- `audience`、`teacher` 只表达分配和责任范围，服务端仍需执行对象级授权。
- `required_videos`/`optional_videos` 只表达数量；视频 URL、观看限制、覆盖区间和完成证据由视频模块保存，不能用数量代替完成事件。
- `help_policy`、`exception_policy`、`retest_policy` 未确认时必须写“待评审”或使用明确的课程配置，不能静默套用建议政策。

## 章节

任务至少包含 `## 必做任务`，并有一项可作答内容。推荐同时包含：

1. `## 学习目标`
2. `## 学习材料`
3. `## 必做任务`，使用 `Q1：...`、`Q2：...` 等稳定题号
4. `## 交付清单`
5. `## 评分标准`

任务文件使用 UTF-8（允许 BOM）或 GB18030，扩展名为 `.md` 或 `.markdown`。合法文件名不要求以 `StudyCheck_` 开头。

## 与当前 Personal 解析器的边界

当前离线 Personal 解析器会保留 `course_id`、`assignment_id`、`task_version`、`title`、`deadline` 和 `estimated_minutes`，并解析章节与题号；其余扩展字段目前作为前置元数据读取但尚未映射到 `Task` 对象。`samples/tasks/` 的扩展字段是 B/C/D/E 联调目标，不能据此声称 v0.1.5 已支持服务端身份、视频完成或 20 分钟计时。

## 样例索引

- `samples/tasks/data-cleaning.md`：有题目和交付清单的常规任务，保留 `deadline` 兼容别名。
- `samples/tasks/no-video-task.md`：0 个视频、可直接作答的任务。
- `samples/tasks/invalid-task.md`：缺少“必做任务”章节，应给出可理解错误。
- `samples/answers/no-video-task-boundary.md`：合格、边界、不合格和技术故障的脱敏预期。
