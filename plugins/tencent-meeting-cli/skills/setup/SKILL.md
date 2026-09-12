---
name: setup
description: Check or install Tencent Meeting CLI (tencentmeeting), guide OAuth2 authorization, and verify local CLI readiness without exposing credentials.
---

# 设置腾讯会议 CLI

这个 Skill 只负责 CLI 安装、OAuth2 授权准备和本地可用性检查，不代替用户执行会议写操作。

## 工作流

1. 检查 CLI 是否已安装：

   ```bash
   command -v tencentmeeting
   tencentmeeting --help
   ```

2. 缺失时，引导用户按官方仓库说明安装最新版。优先查看仓库的 Installation/README；不要猜测发行包名或静默执行远程安装脚本。安装完成后重新运行 `tencentmeeting --help`。

3. 按实时帮助确认 OAuth2 登录命令和回调方式：

   ```bash
   tencentmeeting auth --help
   tencentmeeting config --help
   ```

   用户应在本机浏览器完成腾讯会议开放平台 OAuth2 授权。不要让用户把 client secret、access token、refresh token 或授权码粘贴到聊天，也不要把密钥放进命令参数、shell 历史或日志。

4. 重新检查认证状态并用只读命令验证：

   ```bash
   tencentmeeting auth status
   tencentmeeting --help
   ```

   若 CLI 版本的实际命令不同，以当前 `--help` 输出为准，并在报告中说明实际命令。

## 成功条件

只有 CLI 可执行且 OAuth2 状态检查成功，才能报告“基础就绪”。认证被拒、回调失败或需要开放平台管理员配置时，说明阻塞点并停止后续远程操作。

## 参考

- [腾讯会议 CLI 官方仓库](https://github.com/TencentCloud/tencentmeeting-cli)
- [腾讯会议开放平台](https://meeting.tencent.com/)
