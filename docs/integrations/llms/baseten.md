# Baseten

[Baseten](https://baseten.co) 是 LangChain 生态系统中的一个[服务商](/docs/integrations/providers/baseten)，它实现了 LLMs 组件。

这个示例演示了如何在 LangChain 中使用托管在 Baseten 上的 LLM — Mistral 7B。

# 设置

要运行这个示例，你需要：

* 一个 [Baseten 账户](https://baseten.co)

* 一个 [API 密钥](https://docs.baseten.co/observability/api-keys)

将你的 API 密钥导出为名为 `BASETEN_API_KEY` 的环境变量。

```sh
export BASETEN_API_KEY="在这里粘贴你的 API 密钥"
```

# 单模型调用

首先，你需要将一个模型部署到 Baseten。

你可以通过一键点击从 [Baseten 模型库](https://app.baseten.co/explore/) 部署基础模型，比如 Mistral 和 Llama 2，或者如果你有自己的模型，可以使用 [Truss 进行部署](https://truss.baseten.co/welcome)。

在这个示例中，我们将使用 Mistral 7B。[在这里部署 Mistral 7B](https://app.baseten.co/explore/mistral_7b_instruct)，并跟随部署模型的 ID，在模型仪表板中找到。

```python
from langchain_community.llms import Baseten
```

```python
# 加载模型
mistral = Baseten(model="MODEL_ID", deployment="production")
```

```python
# 提问模型
mistral("Mistral 风是什么？")
```

# 链式模型调用

我们可以将一个或多个模型的多个调用链接在一起，这正是 Langchain 的全部意义所在！

例如，我们可以在这个终端仿真演示中用 Mistral 替换 GPT。

```python
from langchain.chains import LLMChain
from langchain.memory import ConversationBufferWindowMemory
from langchain_core.prompts import PromptTemplate
template = """Assistant 是由 OpenAI 训练的大型语言模型。
Assistant 被设计成能够协助完成各种任务，从回答简单问题到提供广泛话题的深入解释和讨论。作为一个语言模型，Assistant 能够根据接收到的输入生成类似人类的文本，使其能够进行自然对话并提供连贯和相关的回复。
Assistant 不断学习和改进，其能力也在不断发展。它能够处理和理解大量文本，并利用这些知识对各种问题提供准确和丰富的回答。此外，Assistant 能够根据接收到的输入生成自己的文本，使其能够进行讨论并对各种话题进行解释和描述。
总的来说，Assistant 是一个强大的工具，可以帮助完成各种任务，并在各种话题上提供有价值的见解和信息。无论你需要帮助解决特定问题，还是只是想就某个特定话题进行对话，Assistant 都在这里提供帮助。
{history}
Human: {human_input}
Assistant:"""
prompt = PromptTemplate(input_variables=["history", "human_input"], template=template)
chatgpt_chain = LLMChain(
    llm=mistral,
    llm_kwargs={"max_length": 4096},
    prompt=prompt,
    verbose=True,
    memory=ConversationBufferWindowMemory(k=2),
)
output = chatgpt_chain.predict(
    human_input="我希望你能充当 Linux 终端。我会输入命令，你会回复终端应该显示的内容。我希望你只在一个独特的代码块内回复终端输出，不要写解释。除非我指示你这样做，不要输入命令。当我需要用英文告诉你一些事情时，我会用花括号括起来 {就像这样}。我的第一个命令是 pwd。"
)
print(output)
```

```python
output = chatgpt_chain.predict(human_input="ls ~")
print(output)
```

```python
output = chatgpt_chain.predict(human_input="cd ~")
print(output)
```

```python
output = chatgpt_chain.predict(
    human_input="""echo -e "x=lambda y:y*5+3;print('Result:' + str(x(6)))" > run.py && python3 run.py"""
)
print(output)
```

从最后一个示例中可以看出，输出的数字可能正确，也可能不正确，模型只是近似地输出可能的终端内容，而不是实际执行提供的命令。尽管如此，这个示例展示了 Mistral 充足的上下文窗口、代码生成能力以及在对话序列中保持话题相关性的能力。