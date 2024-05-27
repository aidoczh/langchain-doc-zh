# LlamaEdge

[LlamaEdge](https://github.com/second-state/LlamaEdge) 允许您在本地和通过聊天服务与 [GGUF](https://github.com/ggerganov/llama.cpp/blob/master/gguf-py/README.md) 格式的 LLM 进行聊天。

- `LlamaEdgeChatService` 为开发人员提供了一个与 LLM 通过 HTTP 请求进行聊天的 OpenAI API 兼容的服务。

- `LlamaEdgeChatLocal` 使开发人员能够在本地与 LLM 进行聊天（即将推出）。

`LlamaEdgeChatService` 和 `LlamaEdgeChatLocal` 都在由 [WasmEdge Runtime](https://wasmedge.org/) 驱动的基础架构上运行，该基础架构为 LLM 推理任务提供了一个轻量级和便携的 WebAssembly 容器环境。

## 通过 API 服务进行聊天

`LlamaEdgeChatService` 在 `llama-api-server` 上运行。按照 [llama-api-server 快速入门](https://github.com/second-state/llama-utils/tree/main/api-server#readme) 中的步骤，您可以托管自己的 API 服务，这样您就可以在任何设备上通过任何可用的互联网与您喜欢的任何模型进行聊天。

```python
from langchain_community.chat_models.llama_edge import LlamaEdgeChatService
from langchain_core.messages import HumanMessage, SystemMessage
```

### 以非流式模式与 LLM 进行聊天

```python
# 服务 URL
service_url = "https://b008-54-186-154-209.ngrok-free.app"
# 创建 wasm-chat 服务实例
chat = LlamaEdgeChatService(service_url=service_url)
# 创建消息序列
system_message = SystemMessage(content="You are an AI assistant")
user_message = HumanMessage(content="What is the capital of France?")
messages = [system_message, user_message]
# 与 wasm-chat 服务进行聊天
response = chat.invoke(messages)
print(f"[Bot] {response.content}")
```

```output
[Bot] 你好！法国的首都是巴黎。
```

### 以流式模式与 LLM 进行聊天

```python
# 服务 URL
service_url = "https://b008-54-186-154-209.ngrok-free.app"
# 创建 wasm-chat 服务实例
chat = LlamaEdgeChatService(service_url=service_url, streaming=True)
# 创建消息序列
system_message = SystemMessage(content="You are an AI assistant")
user_message = HumanMessage(content="What is the capital of Norway?")
messages = [
    system_message,
    user_message,
]
output = ""
for chunk in chat.stream(messages):
    # print(chunk.content, end="", flush=True)
    output += chunk.content
print(f"[Bot] {output}")
```

```output
[Bot] 你好！我很高兴帮助您解答问题。挪威的首都是奥斯陆。
```