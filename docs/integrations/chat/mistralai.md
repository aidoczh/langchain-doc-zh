---

sidebar_label: MistralAI

---

# MistralAI

本文介绍如何通过[MistralAI的API](https://docs.mistral.ai/api/)开始使用MistralAI聊天模型。

与API通信需要一个有效的[API密钥](https://console.mistral.ai/users/api-keys/)。

请查看[API参考文档](https://api.python.langchain.com/en/latest/chat_models/langchain_mistralai.chat_models.ChatMistralAI.html)以获取所有属性和方法的详细文档。

## 设置

您需要安装`langchain-core`和`langchain-mistralai`包才能使用API。您可以使用以下命令安装：

```bash
pip install -U langchain-core langchain-mistralai
我们还需要获取[Mistral API密钥](https://console.mistral.ai/users/api-keys/)
```python
import getpass
api_key = getpass.getpass()
```
## 使用
```python
from langchain_core.messages import HumanMessage
from langchain_mistralai.chat_models import ChatMistralAI
```
```python
# 如果未传递api_key，则默认行为是使用`MISTRAL_API_KEY`环境变量。
chat = ChatMistralAI(api_key=api_key)
```
```python
messages = [HumanMessage(content="knock knock")]
chat.invoke(messages)
```
```output

AIMessage(content="Who's there? I was just about to ask the same thing! How can I assist you today?")

```
### 异步
```python
await chat.ainvoke(messages)
```
```output

AIMessage(content='Who\'s there?\n\n(接着你可以继续"敲门"笑话，说出应该回应的人或角色的名字。例如，如果我说"香蕉"，你可以回答"香蕉谁？"，然后我会说"香蕉一串！懂了吗？因为一群香蕉被称为"一串"！"，然后我们会一起笑得很开心。但实际上，你可以在我说"香蕉"的地方放任何你想要的东西，它仍然在技术上是一个"敲门"笑话。可能性是无限的！)')

```
### 流式处理
```python
for chunk in chat.stream(messages):
    print(chunk.content, end="")
```
```output

Who's there?

(之后，对话可以继续进行"敲门"式的问答笑话。以下是一个示例：

你说：橙子。

我说：橙子谁？

你说：橙子你高兴我没说香蕉吗！)

但由于你特别要求了一个"敲门"笑话，这里有一个给你：

敲门。

我：谁在那里？

你：生菜。

我：生菜谁？

你：生菜进来吧，外面太冷了！

希望这能让你微笑！你有喜欢的"敲门"笑话想分享吗？我很想听听。

```
### 批处理
```python
chat.batch([messages])
```
```output

[AIMessage(content="Who's there? I was just about to ask the same thing! Go ahead and tell me who's there. I love a good knock-knock joke.")]

```
## 链接
您还可以轻松地与提示模板结合，以便轻松构建用户输入的结构。我们可以使用[LCEL](/docs/concepts#langchain-expression-language)来实现这一点。
```python
from langchain_core.prompts import ChatPromptTemplate
prompt = ChatPromptTemplate.from_template("Tell me a joke about {topic}")
chain = prompt | chat
```
```python
chain.invoke({"topic": "bears"})
```
```output

AIMessage(content='Why do bears hate shoes so much? They like to run around in their bear feet.')

```