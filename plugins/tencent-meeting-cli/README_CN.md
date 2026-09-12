# 腾讯会议 CLI 插件

这是腾讯官方 [tencentmeeting-cli](https://github.com/TencentCloud/tencentmeeting-cli) 的轻量 Skill 封装，不捆绑二进制、MCP 服务或腾讯会议凭证。

## 使用

- `/tencent-meeting-cli:setup`：检查或安装 CLI，引导 OAuth2 授权并验证状态。
- `/tencent-meeting-cli:cli`：通过实时帮助发现会议、录制和参会报告命令。

插件不会索取或输出 Client Secret、Token。创建、修改、取消会议等远程写操作会先展示变更并请求确认。
