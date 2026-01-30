moltbot/Clawdbot完整APIKEY配置指南
配置Claude Code API中转
1. 获取API资源
从Claude Code API中转服务获取：

API 基本 URL：https://api.vsvip.cn/api
API密钥：cr_xxxxxxxxxxxxx
推荐服务：

购买链接：https://api.vsvip.cn/register?aff=dqmJ
推荐中转API：https://api.vsvip.cn/register?aff=dqmJ
⚠️重要提示： Clawdbot不支持通过环境变量ANTHROPIC_BASE_URL来设置自定义API端点。必须通过配置文件的models.providers来配置。

步骤1：导入配置文件
cp ~/.clawdbot/clawdbot.json ~/.clawdbot/clawdbot.json.bak
步骤2：编辑配置文件
nano ~/.clawdbot/clawdbot.json
在配置文件中添加models部分：

{
  "models": {
    "providers": {
      "anthropic": {
        "baseUrl": "https://api.vsvip.cn/api",
        "apiKey": "cr_你的API密钥",
        "api": "anthropic-messages",
        "models": []
      }
    }
  }
}
关键配置说明：

字段	说明	简单
基本网址	自定义API端点	✅
apiKey	你的API键	✅
API	必须设置为anthropic-messages	✅
模型	必须包含此字段，可以为空备份[]	✅
完整配置示例：

{
  "meta": {
    "lastTouchedVersion": "2026.1.25",
    "lastTouchedAt": "2026-01-27T01:05:21.233Z"
  },
  "models": {
    "providers": {
      "anthropic": {
        "baseUrl": "https://api.vsvip.cn/api",
        "apiKey": "cr_你的API密钥",
        "api": "anthropic-messages",
        "models": []
      }
    }
  },
  "agents": {
    "defaults": {
      "model": {
        "primary": "anthropic/claude-sonnet-4-5"
      },
      "workspace": "/Users/你的用户名/clawd",
      "maxConcurrent": 4
    }
  },
  "gateway": {
    "port": 18789,
    "mode": "local",
    "bind": "loopback",
    "auth": {
      "mode": "token",
      "token": "你的gateway_token"
    }
  },
  "channels": {
    "telegram": {
      "enabled": false
    }
  }
}
步骤3：验证配置格式
# 使用jq验证JSON格式
cat ~/.clawdbot/clawdbot.json | jq '.models'

# 应该输出：
# {
#   "providers": {
#     "anthropic": {
#       "baseUrl": "https://api.vsvip.cn/api",
#       "apiKey": "cr_...",
#       "api": "anthropic-messages",
#       "models": []
#     }
#   }
# }
3.重启网关服务
clawdbot gateway restart
4.验证配置生效
# 检查Gateway状态
clawdbot channels status

# 应该显示：
# Gateway reachable.
验证和测试
1.检查网关状态
clawdbot channels status
正常输出：

Gateway reachable.
- Telegram default: disabled, configured, stopped
2.访问Web UI
浏览器访问：

http://127.0.0.1:18789/?token=你的token
网页界面功能：

💬 聊天：直接与 AI 对话
📊 概述：查看系统状态
🔌渠道：管理消息通道
⚙️配置：修改配置
3.发送测试消息
在Web UI的聊天界面：

输入消息：Hello, can you hear me?
点击发送按钮
等待AI回复
预期结果：

状态显示“健康状况良好”
收到AI的回复消息
右上角显示代币使用情况
4. 查看日志
如果遇到问题，检查日志：

# Gateway主日志
tail -f ~/.clawdbot/logs/gateway.log

# 错误日志
tail -f ~/.clawdbot/logs/gateway.err.log

# 详细调试日志
tail -f /tmp/clawdbot/clawdbot-$(date +%Y-%m-%d).log
常见踩坑点
❌踩坑1：环境变量配置无效
错误做法：

# 在LaunchAgent中设置环境变量（无效！）
<key>ANTHROPIC_BASE_URL</key>
<string>https://api.vsvip.cn/api</string>
问题原因： Clawdbot不支持通过ANTHROPIC_BASE_URL环境变量来设置自定义API端点。

✅ 正确做法：在~/.clawdbot/clawdbot.json配置文件中添加：

{
  "models": {
    "providers": {
      "anthropic": {
        "baseUrl": "https://api.vsvip.cn/api",
        "apiKey": "cr_你的密钥",
        "api": "anthropic-messages",
        "models": []
      }
    }
  }
}
❌踩坑2：缺货模型字段
错误配置：

{
  "models": {
    "providers": {
      "anthropic": {
        "baseUrl": "https://api.vsvip.cn/api",
        "apiKey": "cr_xxx",
        "api": "anthropic-messages"
        // 缺少models字段！
      }
    }
  }
}
错误信息：

Invalid config at ~/.clawdbot/clawdbot.json:
- models.providers.anthropic.models: Invalid input: expected array
✅ 正确做法：必须包含models字段，即使是空备份：

{
  "models": {
    "providers": {
      "anthropic": {
        "baseUrl": "https://code.claude-opus.top/api",
        "apiKey": "cr_xxx",
        "api": "anthropic-messages",
        "models": []  // 必须有这一行！
      }
    }
  }
}
❌踩坑3：Telegram连接失败导致网关不稳定
症状：

网关不断重启
日志显示TypeError: fetch failed
Web UI 无法连接
✅ 解决方案：临时取消Telegram：

clawdbot config set channels.telegram.enabled false
clawdbot gateway restart
❌踩坑4：Node.js版本过低
错误信息：

clawdbot requires Node >=22.0.0.
Detected: node 20.19.0
✅ 解决方案：

nvm install 22
nvm use 22
nvm alias default 22
node --version  # 应显示 v22.x.x
❌踩坑5：忘记重启网关
问题：修改配置后没有重启Gateway，配置不生效。

✅ 解决方案：

# 每次修改配置后都要重启
clawdbot gateway restart

# 验证配置生效
clawdbot channels status
