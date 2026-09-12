# Tencent Meeting CLI Plugin

A lightweight Skill wrapper for Tencent's [tencentmeeting-cli](https://github.com/TencentCloud/tencentmeeting-cli). It does not bundle binaries, credentials, or an MCP server.

Use `/tencent-meeting-cli:setup` for installation and OAuth2 readiness, then `/tencent-meeting-cli:cli` for meeting, recording, and attendee-report workflows. Remote writes require confirmation and secrets are never requested in chat.
