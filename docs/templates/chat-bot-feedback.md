# 聊天机器人反馈模板

这个模板展示了如何在没有明确用户反馈的情况下评估您的聊天机器人。它在 [chain.py](https://github.com/langchain-ai/langchain/blob/master/templates/chat-bot-feedback/chat_bot_feedback/chain.py) 中定义了一个简单的聊天机器人和自定义评估器，根据随后用户的回复对机器人的响应效果进行评分。您可以通过在提供服务之前在聊天机器人上调用 `with_config` 来应用此运行评估器。您也可以直接使用此模板部署您的聊天应用程序。

[聊天机器人](https://python.langchain.com/docs/use_cases/chatbots) 是部署 LLMs 最常见的接口之一。聊天机器人的质量各不相同，因此持续开发非常重要。但是，用户往往不会通过类似点赞或踩的按钮等机制留下明确的反馈。此外，传统的分析方法，如“会话长度”或“对话长度”，通常缺乏清晰度。然而，与聊天机器人进行多轮对话可以提供大量信息，我们可以将其转化为微调、评估和产品分析的指标。

以 [Chat Langchain](https://chat.langchain.com/) 为案例研究，大约只有 0.04% 的所有查询收到明确的反馈。然而，大约 70% 的查询是对先前问题的后续。这些后续查询中的大部分继续提供有用信息，我们可以用来推断先前 AI 响应的质量。

这个模板有助于解决这个“反馈稀缺性”问题。以下是对此聊天机器人的示例调用：

![](https://smith.langchain.com/public/3378daea-133c-4fe8-b4da-0a3044c5dbe8/r?runtab=1)

当用户对此做出回应时（[链接](https://smith.langchain.com/public/a7e2df54-4194-455d-9978-cecd8be0df1e/r)），将调用响应评估器，导致以下评估运行：

![](https://smith.langchain.com/public/534184ee-db8f-4831-a386-3f578145114c/r)

如图所示，评估器发现用户越来越沮丧，表明先前的响应并不有效。

## LangSmith 反馈

[LangSmith](https://smith.langchain.com/) 是一个用于构建生产级 LLM 应用程序的平台。除了其调试和离线评估功能外，LangSmith 还帮助您捕获用户和模型辅助反馈，以完善您的 LLM 应用程序。此模板使用 LLM 生成反馈，您可以使用这些反馈持续改进您的服务。要了解如何使用 LangSmith 收集反馈的更多示例，请参阅[文档](https://docs.smith.langchain.com/cookbook/feedback-examples)。

## 评估器实现

用户反馈是通过自定义的 `RunEvaluator` 推断的。通过使用 `EvaluatorCallbackHandler` 调用此评估器，可以在单独的线程中运行它，以避免干扰聊天机器人的运行时。您可以通过在 LangChain 对象上调用以下函数，在任何兼容的聊天机器人上使用此自定义评估器：

```python
my_chain.with_config(
    callbacks=[
        EvaluatorCallbackHandler(
            evaluators=[
                ResponseEffectivenessEvaluator(evaluate_response_effectiveness)
            ]
        )
    ],
)
```

该评估器指示一个 LLM，具体是 `gpt-3.5-turbo`，根据用户的后续回复评估 AI 的最新聊天消息。它生成一个分数和相应的推理，将其转换为 LangSmith 中的反馈，并应用于提供的 `last_run_id` 值。

在 LLM 中使用的提示[可在 hub 上找到](https://smith.langchain.com/hub/wfh/response-effectiveness)。请随意根据需要自定义它，例如添加额外的应用程序上下文（例如应用程序的目标或应该响应的问题类型）或您希望 LLM 关注的“症状”。此评估器还利用 OpenAI 的函数调用 API，以确保更一致、结构化的输出用于评分。

## 环境变量

确保设置 `OPENAI_API_KEY` 以使用 OpenAI 模型。还要通过设置 `LANGSMITH_API_KEY` 配置 LangSmith。

```bash
export OPENAI_API_KEY=sk-...
export LANGSMITH_API_KEY=...
export LANGCHAIN_TRACING_V2=true
export LANGCHAIN_PROJECT=my-project # 设置要保存到的项目
```

## 用法

如果通过 `LangServe` 部署，我们建议配置服务器以返回回调事件。这将确保后端跟踪包括您使用 `RemoteRunnable` 生成的任何跟踪。

```python
from chat_bot_feedback.chain import chain
add_routes(app, chain, path="/chat-bot-feedback", include_callback_events=True)
```

在服务器运行时，您可以使用以下代码片段来为两轮对话流式传输聊天机器人响应：

```python
from functools import partial
from typing import Dict, Optional, Callable, List
from langserve import RemoteRunnable
from langchain.callbacks.manager import tracing_v2_enabled
from langchain_core.messages import BaseMessage, AIMessage, HumanMessage
# 使用您的 LangServe 服务器提供的 URL 进行更新
chain = RemoteRunnable("http://127.0.0.1:8031/chat-bot-feedback")
def stream_content(
    text: str,
    chat_history: Optional[List[BaseMessage]] = None,
    last_run_id: Optional[str] = None,
    on_chunk: Callable = None,
):
    results = []
    with tracing_v2_enabled() as cb:
        for chunk in chain.stream(
            {"text": text, "chat_history": chat_history, "last_run_id": last_run_id},
        ):
            on_chunk(chunk)
            results.append(chunk)
        last_run_id = cb.latest_run.id if cb.latest_run else None
    return last_run_id, "".join(results)
chat_history = []
text = "我的钥匙在哪里？"
last_run_id, response_message = stream_content(text, on_chunk=partial(print, end=""))
print()
chat_history.extend([HumanMessage(content=text), AIMessage(content=response_message)])
text = "我到处都找不到它们"  # 由于用户的沮丧情绪似乎在升级，先前的回复可能会得到较低的分数。
last_run_id, response_message = stream_content(
    text,
    chat_history=chat_history,
    last_run_id=str(last_run_id),
    on_chunk=partial(print, end=""),
)
print()
chat_history.extend([HumanMessage(content=text), AIMessage(content=response_message)])
```

```
这里使用 `tracing_v2_enabled` 回调管理器来获取调用的运行 ID，在后续的对话中我们会提供这个 ID，这样评估器就可以将反馈分配给适当的追踪。
## 结论
这个模板提供了一个简单的聊天机器人定义，您可以直接使用 LangServe 进行部署。它定义了一个自定义评估器，用于记录机器人的评估反馈，而无需任何明确的用户评分。这是一种有效的方式，可以增强您的分析能力，并更好地选择数据点进行微调和评估。```