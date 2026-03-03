# OpenClaw 飞书集成部署 Skill

## 前置准备清单

在开始部署前，请准备好以下信息：

### 1. 飞书应用信息
- **App ID**: `cli_xxxxx`（飞书开放平台创建应用后获取）
- **App Secret**: `xxxxx`（飞书开放平台创建应用后获取）
- 获取地址：https://open.feishu.cn/app

### 2. AI 模型 API Key（三选一）
**选项 A：OpenRouter（推荐，支持免费模型）**
- **API Key**: `sk-or-v1-xxxxx`
- 获取地址：https://openrouter.ai/settings/keys
- 免费模型：`openrouter/qwen/qwen3-coder:free`

**选项 B：通义千问 DashScope**
- **API Key**: `sk-xxxxx`
- 获取地址：https://dashscope.console.aliyun.com/
- 注意：需通过 OpenRouter 访问

**选项 C：MiniMax**
- **API Key**: `xxxxx`
- 获取地址：https://www.minimaxi.com/


### 3. 系统要求
- macOS 系统
- 已安装 Homebrew
- 已安装 Node.js 22+（或使用 nvm）

---

## 部署步骤

### 步骤 1：安装 OpenClaw

```bash
# 下载并执行安装脚本
curl -fsSL https://openclaw.ai/install.sh | bash
```

**验证安装：**
```bash
openclaw --version
# 应输出：2026.3.1 或更高版本
```

---

### 步骤 2：配置飞书插件

```bash
# 安装飞书插件
openclaw plugins install @openclaw/feishu
```

**手动配置飞书渠道：**

编辑配置文件：
```bash
vim ~/.openclaw/openclaw.json
```

在 `channels` 部分添加飞书配置：
```json
{
  "channels": {
    "feishu": {
      "appId": "cli_xxxxx",  // 替换为你的 App ID
      "appSecret": "xxxxx",   // 替换为你的 App Secret
      "enabled": true
    }
  }
}
```

---

### 步骤 3：配置 Gateway 模式

确保配置文件中有 `gateway.mode` 设置：

```bash
openclaw config set gateway.mode "local"
```

或手动编辑 `~/.openclaw/openclaw.json`：
```json
{
  "gateway": {
    "mode": "local",
    "auth": {
      "mode": "token",
      "token": "自动生成的token"
    }
  }
}
```

---




### 步骤 4：启动并验证 Gateway

```bash
# 重启 Gateway 服务
launchctl stop ai.openclaw.gateway
sleep 3
launchctl start ai.openclaw.gateway
sleep 5

# 检查服务状态
openclaw gateway status
```

**预期输出：**
```
Runtime: running (pid xxxxx)
RPC probe: ok
Listening: 127.0.0.1:18789
```

---

### 步骤 6：配置飞书用户配对

**1. 在飞书中给机器人发送任意消息**

你会收到类似这样的回复：
```
OpenClaw: access not configured.
Your Feishu user id: ou_xxxxx
Pairing code: ABC12345
Ask the bot owner to approve with:
openclaw pairing approve feishu ABC12345
```

**2. 批准配对请求：**
```bash
openclaw pairing approve feishu ABC12345
```

**3. 验证配对成功：**
```bash
openclaw channels status
```

---

### 步骤 7：测试完整流程

**在飞书中 @机器人 发送消息：**
```
你好，测试一下
```

机器人应该会正常回复。

---

## 故障排查

### 问题 1：Gateway 启动失败

**错误信息：**
```
Gateway start blocked: set gateway.mode=local
```

**解决方案：**
```bash
openclaw config set gateway.mode "local"
launchctl restart ai.openclaw.gateway
```

---

### 问题 2：模型认证失败

**错误信息：**
```
No API key found for provider "openrouter"
```

**解决方案：**
1. 检查认证文件是否存在：
```bash
cat ~/.openclaw/agents/main/agent/auth-profiles.json
```

2. 确认格式正确（参考步骤 4）

3. 重启 Gateway：
```bash
launchctl unload ~/Library/LaunchAgents/ai.openclaw.gateway.plist
launchctl load ~/Library/LaunchAgents/ai.openclaw.gateway.plist
```

---

### 问题 3：飞书消息无响应

**检查清单：**
1. Gateway 是否正常运行：
```bash
openclaw gateway status
```

2. 飞书渠道是否启用：
```bash
openclaw channels status
```

3. 用户是否已配对：
```bash
openclaw pairing list
```

4. 查看日志：
```bash
openclaw logs --follow
```

---

## 配置文件位置

- **主配置文件**: `~/.openclaw/openclaw.json`
- **认证配置**: `~/.openclaw/agents/main/agent/auth-profiles.json`
- **日志文件**: `/tmp/openclaw/openclaw-YYYY-MM-DD.log`
- **服务配置**: `~/Library/LaunchAgents/ai.openclaw.gateway.plist`

---

## 常用命令

```bash
# 查看配置
openclaw config file
openclaw config get <path>

# 模型管理
openclaw models list
openclaw models status
openclaw models set <model-id>

# 渠道管理
openclaw channels list
openclaw channels status

# 服务管理
openclaw gateway status
launchctl stop ai.openclaw.gateway
launchctl start ai.openclaw.gateway

# 查看日志
openclaw logs --follow

# 健康检查
openclaw doctor
```

---

## 安全建议

1. **定期轮换 API Key**
2. **不要将 API Key 提交到 Git 仓库**
3. **使用环境变量存储敏感信息**（可选）
4. **定期备份配置文件**

---

## 更新 OpenClaw

```bash
# 检查更新
openclaw update check

# 执行更新
openclaw update
```

---

## 卸载

```bash
# 卸载 OpenClaw（保留 CLI）
openclaw uninstall

# 完全卸载（包括 CLI）
npm uninstall -g openclaw
rm -rf ~/.openclaw
```

---

## 参考资源

- OpenClaw 官网：https://openclaw.ai/
- 官方文档：https://docs.openclaw.ai/
- GitHub Issues：https://github.com/openclaw/openclaw/issues
- 飞书开放平台：https://open.feishu.cn/
- OpenRouter：https://openrouter.ai/

---

## 版本信息

- **Skill 版本**: 1.0.0
- **OpenClaw 版本**: 2026.3.1
- **最后更新**: 2026-03-03
- **测试环境**: macOS (Darwin 22.6.0)
