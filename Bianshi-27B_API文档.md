# Bianshi-27B API 调用文档

Bianshi-27B 是基于 Qwen3.6-27B 开源权重微调的医疗大语言模型，已部署为 OpenAI 兼容的 HTTP API 服务，供 MedBench 评测平台调用。

## 基本信息

| 项 | 值 |
|----|----|
| API Endpoint | `https://model.a-eye.cn/v1` |
| 模型 ID | `Bianshi-27B` |
| 参数量 | 27B |
| 上下文长度 | 32K |
| 请求方式 | POST `/chat/completions`（OpenAI 兼容） |
| 认证 | Bearer Token（请求头 `Authorization: Bearer <API_KEY>`） |

## 鉴权方式

在请求头中携带 API Key：

```
Authorization: Bearer sk-你的_API_Key
```

## 调用示例（Python）

```python
import json
from openai import OpenAI

api_key = "sk-你的_API_Key"
base_url = "https://model.a-eye.cn/v1"
path = "Bianshi-27B"
question = "你好"

client = OpenAI(
    api_key=api_key,
    base_url=base_url,
)

completion = client.chat.completions.create(model=path, messages=[{'role': 'user', 'content': question}])
response = json.loads(completion.model_dump_json())
print(response['choices'][0]['message']['content'])
```

## 调用示例（curl）

```bash
curl https://model.a-eye.cn/v1/chat/completions \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer sk-你的_API_Key" \
  -d '{
    "model": "Bianshi-27B",
    "messages": [{"role": "user", "content": "你好"}],
    "max_tokens": 2048,
    "temperature": 0.0
  }'
```

## 请求参数

| 参数 | 类型 | 说明 |
|------|------|------|
| `model` | string | 必填，模型 ID：`Bianshi-27B` |
| `messages` | array | 必填，对话消息列表（`[{role, content}]`） |
| `max_tokens` | integer | 选填，最大生成长度（默认 2048，最大 32768） |
| `temperature` | float | 选填，采样温度（默认 0.0，范围 [0, 2]） |
| `top_p` | float | 选填，核采样（默认 1.0） |

## 响应格式（OpenAI 兼容）

```json
{
  "id": "chatcmpl-xxx",
  "object": "chat.completion",
  "choices": [{
    "index": 0,
    "message": {"role": "assistant", "content": "回答内容"},
    "finish_reason": "stop"
  }],
  "usage": {"prompt_tokens": 10, "completion_tokens": 20, "total_tokens": 30}
}
```

> 注：`message.content` 为最终回答；模型可能附带思考链（`reasoning` 字段），评测读取 `content` 即可。

## 可用性

- 公网可访问，评测期间保持服务在线
- 支持并发调用
- 响应超时建议设为 300 秒以上（部分复杂推理题生成较长）
