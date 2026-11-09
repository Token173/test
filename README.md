# 中转接口连通性测试仓库：判断 API 是否可用的最小验证

这个仓库只做一件事：用最少的请求确认 Token173（https://token173.net）的接口在你本机通不通、Key 有没有生效、模型带没带上。

## 为什么单独放一个测试仓

排查问题的时候最怕混在业务代码里。先把"网络通不通、Key 对不对、模型存不存在"这三件事分开验证，剩下的才好定位。

## 三个测试，逐个跑

**一、网络**

```bash
curl -I https://token173.net
```

返回 200 或 30x 说明通；超时就是本机 DNS 或网络的问题，和 Key 无关。

**二、Key 与文本接口**

```bash
curl https://token173.net/v1/chat/completions \
  -H "Authorization: Bearer sk-你的令牌" \
  -H "Content-Type: application/json" \
  -d '{"model":"gpt-6.1-sol","messages":[{"role":"user","content":"回复 OK"}]}'
```

- 200 且有内容：Key 和模型都正常
- 401：Key 复制错了或已删
- 404：路径写错，检查 `/v1`
- 400 且提示模型不存在：模型 ID 拼错或分组不含该模型

**三、改模型名验证分组**

把 `model` 换成 `claude-sonnet-5-5` 再发一次。能通说明 Anthropic 系列也在同一分组里；报错就说明分组是按模型隔离的，需要另建令牌。

## 常见返回码对照

| 状态码 | 含义 | 先查什么 |
|---|---|---|
| 200 | 正常 | — |
| 400 | 参数错误 | 模型名、JSON 结构 |
| 401 | 未授权 | Key 是否完整、是否被删 |
| 403 | 无权访问该模型 | 令牌分组、模型权限 |
| 404 | 路径不存在 | Base URL 是否多或少 `/v1` |
| 429 | 触发限流 | 并发、请求频率 |
| 500 / 502 | 服务端异常 | 稍后重试，持续出现找客服 |

## 这个仓库不做什么

不放业务代码，不放测试数据，不存任何 Key。测试用的令牌建议单独建一个，额度给到最小，验证完直接删。

## 相关

- 完整文档：https://docs.token173.net
- 接入配置总表：[doc-token173-api](https://github.com/Token173/doc-token173-api)
- 多模型聚合：[2026-Token173-Claude-ChatGPT-Grok-Gemini-AI-API](https://github.com/Token173/2026-Token173-Claude-ChatGPT-Grok-Gemini-AI-API)
