---
name: cli
description: Use the authenticated Tencent Meeting CLI for meeting management, recordings, and attendee reports, discovering exact syntax from live help and gating remote writes.
---

# 使用腾讯会议 CLI

所有操作优先通过本机 `tencentmeeting` CLI 完成。若本回合没有可信的 setup 结果，先读取并执行 `../setup/SKILL.md` 的检查。

## 命令选择

先读取当前版本的帮助，不凭记忆猜参数：

```bash
tencentmeeting --help
tencentmeeting meeting --help
tencentmeeting recording --help
tencentmeeting report --help
```

使用 OAuth2 关联的账号和结构化输出（若当前版本支持）：

```bash
tencentmeeting auth status
tencentmeeting ... --output json
```

## 读写边界

- 查询会议、录制和参会报告属于只读操作，但仍需确认账号和时间范围。
- 创建、修改、取消会议，管理录制或导出/分享数据属于远程写操作；执行前展示账号、会议或资源标识、时间和变更摘要，并请求用户确认。
- 不输出 client secret、access token、refresh token 或完整参会者敏感信息；报告只保留完成任务所需字段。
- 发现权限或参数错误时先读取实时 `--help` 和错误信息，不通过猜参数或更换账号绕过授权。

## 完成报告

说明实际 CLI 版本、账号状态、使用的命令和结果；未确认的写操作只给出预览命令。
