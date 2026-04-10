# SearXNG 设为默认搜索提供商教程

## 1. 简介
SearXNG 是一个开源的元搜索引擎，可通过 JSON API 获取搜索结果。在 OpenClaw 中将其配置为默认 `web_search` 提供商，可让所有 `web_search` 调用自动使用 SearXNG，省去手动指定的麻烦。

## 2. 前置条件
- 已在系统中部署并运行 SearXNG 实例（示例地址：`http://searxng.zeabur.internal:8080`）。
- 确认 SearXNG 能正常响应 `format=json` 的查询，且具备 `json` 输出格式。

## 3. 步骤概述

### 3.1 打开配置文件
```bash
nano /home/node/.openclaw/openclaw.json
```

### 3.2 确保 `tools.web.search.provider` 为 `searxng`
文件中应已有类似配置：
```json
"tools": {
  "web": {
    "search": {
      "provider": "searxng",
      "maxResults": 5,
      "timeoutSeconds": 30,
      "cacheTtlMinutes": 15
    }
  }
}
```

### 3.3 为插件入口添加 JSON 格式支持
在插件配置的 `searxng.config` 中加入 `search.formats` 字段：

```json
"plugins": {
  "entries": {
    "searxng": {
      "config": {
        "webSearch": {
          "baseUrl": "http://searxng.zeabur.internal:8080",
          "categories": "general,news",
          "language": "zh-CN",
          "search.formats": ["json"]   // <-- 新增行
        }
      }
    }
  }
}
```

> **注意**：`search.formats` 必须是数组，且包含 `"json"`，否则 API 调用会返回 403 错误。

### 3.4 保存并重启网关
```bash
openclaw gateway restart
```
此命令会重启网关进程，使新配置生效。

### 3.5 验证配置
- 发送一个测试 `web_search` 请求（例如在聊天中输入 “web_search:cat pictures” 并观察返回结果）。
- 检查日志或直接查看返回的 JSON 是否包含预期的搜索结果。

## 4. 常见问题

| 症状 | 可能原因 | 解决办法 |
|------|----------|----------|
| `403 Forbidden` | `search.formats` 未包含 `json` 或未保存 | 确认 `search.formats: ["json"]` 已加入并保存配置 |
| 搜索结果为空 | `baseUrl` 或网络不可达 | 检查 SearXNG 实例是否可达，确认 URL 正确 |
| 超时 | `timeoutSeconds` 设置过低 | 适当增大 `timeoutSeconds` 直至足够 |

## 5. 小贴士
- **缓存**：默认缓存 15 分钟，可通过修改 `"cacheTtlMinutes"` 调整。
- **安全**：如在生产环境使用，建议启用 HTTPS 并配置身份验证。
- **多语言**：修改 `"language"` 参数可切换查询语言，如 `"language": "en"`。

## 6. 完整示例配置片段

```json
{
  "tools": {
    "web": {
      "search": {
        "provider": "searxng",
        "maxResults": 5,
        "timeoutSeconds": 30,
        "cacheTtlMinutes": 15
      }
    }
  },
  "plugins": {
    "entries": {
      "searxng": {
        "config": {
          "webSearch": {
            "baseUrl": "http://searxng.zeabur.internal:8080",
            "categories": "general,news",
            "language": "zh-CN",
            "search.formats": ["json"]
          }
        }
      }
    }
  }
}
```

完成以上步骤后，所有 `web_search` 调用将默认使用你部署的 SearXNG 实例，实现完全自动化的搜索体验。祝使用愉快！
