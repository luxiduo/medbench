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
Authorization: Bearer 你的_API_KEY
```

## 调用示例（Python）

```python
import json
from openai import OpenAI

api_key = "你的_API_KEY"
base_url = "https://model.a-eye.cn/v1"
path = "Bianshi-27B"
question = "你好"

client = OpenAI(
    api_key=api_key,
    base_url=base_url,
)

completion = client.chat.completions.create(
    model=path,
    messages=[{'role': 'user', 'content': question}],
    max_tokens=32768,         # 【必填】必须设大，过小会截断长生成任务
    temperature=0.0,
    extra_body={'chat_template_kwargs': {'enable_thinking': True}},  # 【必填】必须开启深度思考
    timeout=300,           # 深度思考+长生成单题可达 60s+，超时必须设足（300~400s），过短会超时失败
)
response = json.loads(completion.model_dump_json())
print(response['choices'][0]['message']['content'])
```

## 调用示例（curl）

```bash
curl https://model.a-eye.cn/v1/chat/completions \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer 你的_API_KEY" \
  -d '{
    "model": "Bianshi-27B",
    "messages": [{"role": "user", "content": "你好"}],
    "max_tokens": 32768,
    "temperature": 0.0,
    "chat_template_kwargs": {"enable_thinking": true}
  }'
```

## 请求参数

| 参数 | 类型 | 说明 |
|------|------|------|
| `model` | string | 必填，模型 ID：`Bianshi-27B` |
| `messages` | array | 必填，对话消息列表（`[{role, content}]`） |
| `max_tokens` | integer | **必填**，最大生成长度，**必须设大**（部署上限 65536；评测含长文本生成+深度思考，官方调用必须设 ≥16384，建议 32768，过小会截断） |
| `chat_template_kwargs.enable_thinking` | boolean | **必填**，**必须为 `true`**（开启深度思考，官方调用必须开启） |
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
- **单请求耗时**：开启深度思考后，单题约 24~60 秒（复杂长生成题可达 60s+）
- **超时要求（重要）**：客户端**必须设足超时（建议 300~400 秒）**，过短会导致深度思考中的长生成任务超时失败。示例代码已设 `timeout=300`
- **并发**：支持高并发调用（多卡部署），预计并发量可设 16~32
