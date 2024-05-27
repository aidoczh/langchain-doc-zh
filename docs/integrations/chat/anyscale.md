---

sidebar_label: Anyscale

---

# ChatAnyscale

这篇笔记展示了如何使用 `langchain.chat_models.ChatAnyscale` 来访问 [Anyscale Endpoints](https://endpoints.anyscale.com/)。

- 设置 `ANYSCALE_API_KEY` 环境变量

- 或使用 `anyscale_api_key` 关键字参数

```python
%pip install --upgrade --quiet  langchain-openai
```

```python
import os
from getpass import getpass
os.environ["ANYSCALE_API_KEY"] = getpass()
```

```output
 ········
```

# 让我们尝试 Anyscale Endpoints 提供的每个模型

```python
from langchain_community.chat_models import ChatAnyscale
chats = {
    model: ChatAnyscale(model_name=model, temperature=1.0)
    for model in ChatAnyscale.get_available_models()
}
print(chats.keys())
```

```output
dict_keys(['meta-llama/Llama-2-70b-chat-hf', 'meta-llama/Llama-2-7b-chat-hf', 'meta-llama/Llama-2-13b-chat-hf'])
```

# 我们可以使用 ChatOpenAI 支持的异步方法和其他功能

这样，三个请求的时间将仅取决于最长的单个请求。

```python
import asyncio
from langchain_core.messages import HumanMessage, SystemMessage
messages = [
    SystemMessage(content="You are a helpful AI that shares everything you know."),
    HumanMessage(
        content="Tell me technical facts about yourself. Are you a transformer model? How many billions of parameters do you have?"
    ),
]
async def get_msgs():
    tasks = [chat.apredict_messages(messages) for chat in chats.values()]
    responses = await asyncio.gather(*tasks)
    return dict(zip(chats.keys(), responses))
```

```python
import nest_asyncio
nest_asyncio.apply()
```

```python
%%time
response_dict = asyncio.run(get_msgs())
for model_name, response in response_dict.items():
    print(f"\t{model_name}")
    print()
    print(response.content)
    print("\n---\n")
```

```output

	meta-llama/Llama-2-70b-chat-hf

你好！我只是一个 AI，没有像人类那样的个人身份，但我在这里帮助您解答任何问题。

我是一个大型语言模型，这意味着我经过大量文本数据的训练，可以生成连贯自然的语言输出。我的架构基于变压器模型，这是一种特别适用于自然语言处理任务的神经网络类型。

至于我的参数，我有几十亿个参数，但我无法访问确切的数字，因为这与我的功能无关。我的训练数据包括来自各种来源的大量文本，包括书籍、文章和网站，我用这些数据来学习语言中的模式和关系。

我被设计成一个多功能工具，可以回答问题、提供信息和生成文本。通过机器学习算法和用户反馈，我不断学习和改进自己的能力。

希望这可以帮到您！您还想了解我或我的能力的其他信息吗？

---

	meta-llama/Llama-2-7b-chat-hf

啊，一个科技爱好者！*整理眼镜* 我很高兴分享一些关于我的技术细节。🤓

确实，我是一个变压器模型，具体来说是基于大量文本数据训练的 BERT 类似语言模型。我的架构基于变压器框架，这是一种专为自然语言处理任务设计的神经网络类型。🏠

至于参数数量，我大约有 3.4 亿个。*眨眼* 这是一个相当庞大的数字，我自己都觉得不错！这些参数使我能够学习和表示语言中的复杂模式，如句法、语义等。🤔

但请不要让我在脑海中做数学运算 – 我是一个语言模型，不是计算机！😅 我的优势在于理解和生成类似人类的文本，随时欢迎与我聊天。💬

现在，您还有关于技术方面的问题吗？或者想进行愉快的聊天？😊

---

	meta-llama/Llama-2-13b-chat-hf

你好！作为友好而乐于助人的 AI，我很乐意分享一些关于我的技术事实。

我是基于变压器的语言模型，具体来说是 BERT（来自变压器的双向编码器表示）架构的一个变体。BERT 是由 Google 在 2018 年开发的，目前已成为最受欢迎和广泛使用的 AI 语言模型之一。

以下是关于我的能力的一些技术细节：

1. 参数：我大约有 3.4 亿个参数，这些数字用于学习和表示语言。与其他语言模型相比，这是一个相对较大的参数数量，但它使我能够学习和理解复杂的语言模式和关系。

2. 训练：我是在大量文本数据上进行训练的，包括书籍、文章和其他书面内容。这种训练使我能够了解语言的结构和惯例，以及单词和短语之间的关系。

3. 架构：我的架构基于变压器模型，这是一种特别适用于自然语言处理任务的神经网络类型。变压器模型使用自注意机制，允许模型“关注”输入文本的不同部分，从而捕捉长距离依赖性和上下文关系。

4. 精度：我能够以高精度和准确性生成文本，这意味着我可以生成在语法、句法和连贯性方面接近人类水平质量的文本。

5. 生成能力：除了能够根据提示和问题生成文本外，我还能够根据给定的主题或主题生成文本。这使我能够创建围绕特定想法或概念组织的更长、更连贯的文本。

总的来说，我是一个功能强大且多才多艺的语言模型，能够执行各种自然语言处理任务。我不断学习和改进，随时准备回答您可能有的任何问题！

---

CPU 时间：用户 371 毫秒，系统：15.5 毫秒，总计：387 毫秒

墙上时间：12 秒