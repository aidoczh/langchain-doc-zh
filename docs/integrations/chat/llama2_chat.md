# Llama2Chat

本文档展示了如何使用 `Llama2Chat` 包装器来增强 Llama-2 中的 `LLM`，以支持 [Llama-2 聊天提示格式](https://huggingface.co/blog/llama2#how-to-prompt-llama-2)。LangChain 中的几个 `LLM` 实现可以作为 Llama-2 聊天模型的接口。这些包括 [ChatHuggingFace](/docs/integrations/chat/huggingface), [LlamaCpp](/docs/tutorials/local_rag), [GPT4All](/docs/integrations/llms/gpt4all) 等等。

`Llama2Chat` 是一个通用的包装器，实现了 `BaseChatModel`，因此可以作为 [聊天模型](/docs/how_to#chat-models) 在应用程序中使用。`Llama2Chat` 将消息列表转换为 [所需的聊天提示格式](https://huggingface.co/blog/llama2#how-to-prompt-llama-2)，并将格式化的提示作为 `str` 转发给包装的 `LLM`。

```python
from langchain.chains import LLMChain
from langchain.memory import ConversationBufferMemory
from langchain_experimental.chat_models import Llama2Chat
```

对于下面的聊天应用示例，我们将使用以下聊天 `prompt_template`：

```python
from langchain_core.messages import SystemMessage
from langchain_core.prompts.chat import (
    ChatPromptTemplate,
    HumanMessagePromptTemplate,
    MessagesPlaceholder,
)
template_messages = [
    SystemMessage(content="You are a helpful assistant."),
    MessagesPlaceholder(variable_name="chat_history"),
    HumanMessagePromptTemplate.from_template("{text}"),
]
prompt_template = ChatPromptTemplate.from_messages(template_messages)
```

## 通过 `HuggingFaceTextGenInference` LLM 与 Llama-2 进行聊天

HuggingFaceTextGenInference LLM 封装了对 [text-generation-inference](https://github.com/huggingface/text-generation-inference) 服务器的访问。在下面的示例中，推理服务器提供了一个 [meta-llama/Llama-2-13b-chat-hf](https://huggingface.co/meta-llama/Llama-2-13b-chat-hf) 模型。可以在本地启动：

```bash
docker run \
  --rm \
  --gpus all \
  --ipc=host \
  -p 8080:80 \
  -v ~/.cache/huggingface/hub:/data \
  -e HF_API_TOKEN=${HF_API_TOKEN} \
  ghcr.io/huggingface/text-generation-inference:0.9 \
  --hostname 0.0.0.0 \
  --model-id meta-llama/Llama-2-13b-chat-hf \
  --quantize bitsandbytes \
  --num-shard 4
```

例如，这可以在一台配备 4 张 RTX 3080ti 显卡的机器上运行。根据可用的 GPU 数量调整 `--num_shard` 值。`HF_API_TOKEN` 环境变量保存着 Hugging Face API 令牌。

```python
# !pip3 install text-generation
```

创建一个连接到本地推理服务器的 `HuggingFaceTextGenInference` 实例，并将其包装成 `Llama2Chat`。

```python
from langchain_community.llms import HuggingFaceTextGenInference
llm = HuggingFaceTextGenInference(
    inference_server_url="http://127.0.0.1:8080/",
    max_new_tokens=512,
    top_k=50,
    temperature=0.1,
    repetition_penalty=1.03,
)
model = Llama2Chat(llm=llm)
```

然后，您可以在 `LLMChain` 中使用聊天 `model`、`prompt_template` 和对话 `memory`。

```python
memory = ConversationBufferMemory(memory_key="chat_history", return_messages=True)
chain = LLMChain(llm=model, prompt=prompt_template, memory=memory)
```

```python
print(
    chain.run(
        text="What can I see in Vienna? Propose a few locations. Names only, no details."
    )
)
```

```output
 Sure, I'd be happy to help! Here are a few popular locations to consider visiting in Vienna:
1. Schönbrunn Palace
2. St. Stephen's Cathedral
3. Hofburg Palace
4. Belvedere Palace
5. Prater Park
6. Vienna State Opera
7. Albertina Museum
8. Museum of Natural History
9. Kunsthistorisches Museum
10. Ringstrasse
```

```python
print(chain.run(text="Tell me more about #2."))
```

```output
 Certainly! St. Stephen's Cathedral (Stephansdom) is one of the most recognizable landmarks in Vienna and a must-see attraction for visitors. This stunning Gothic cathedral is located in the heart of the city and is known for its intricate stone carvings, colorful stained glass windows, and impressive dome.
The cathedral was built in the 12th century and has been the site of many important events throughout history, including the coronation of Holy Roman emperors and the funeral of Mozart. Today, it is still an active place of worship and offers guided tours, concerts, and special events. Visitors can climb up the south tower for panoramic views of the city or attend a service to experience the beautiful music and chanting.
```

## 通过 `LlamaCPP` LLM 与 Llama-2 进行聊天

使用 [LlamaCPP](/docs/integrations/llms/llamacpp) 的 `LMM` 与 Llama-2 聊天模型配合使用时，需要安装 `llama-cpp-python` 库，安装说明请参考[这些安装说明](/docs/integrations/llms/llamacpp#installation)。以下示例使用了一个量化的 [llama-2-7b-chat.Q4_0.gguf](https://huggingface.co/TheBloke/Llama-2-7b-Chat-GGUF/resolve/main/llama-2-7b-chat.Q4_0.gguf) 模型，该模型存储在本地路径 `~/Models/llama-2-7b-chat.Q4_0.gguf`。

创建 `LlamaCpp` 实例后，`llm` 再次被包装成 `Llama2Chat`。

```python
from os.path import expanduser
from langchain_community.llms import LlamaCpp
model_path = expanduser("~/Models/llama-2-7b-chat.Q4_0.gguf")
llm = LlamaCpp(
    model_path=model_path,
    streaming=False,
)
model = Llama2Chat(llm=llm)
```

然后可以像之前的示例一样使用。

```python
memory = ConversationBufferMemory(memory_key="chat_history", return_messages=True)
chain = LLMChain(llm=model, prompt=prompt_template, memory=memory)
```

```python
print(
    chain.run(
        text="What can I see in Vienna? Propose a few locations. Names only, no details."
    )
)
```

```output
  当然！维也纳是一座拥有丰富历史和文化的美丽城市。以下是一些您可能想考虑参观的热门旅游景点：
1. 谢恩布鲁恩宫
2. 圣史蒂芬大教堂
3. 哈夫堡宫
4. 贝尔维德宫
5. 普拉特公园
6. 博物馆区
7. 环城大道
8. 维也纳国家歌剧院
9. 维也纳艺术史博物馆
10. 帝国宫殿
这些只是维也纳众多令人惊叹的地方之一。每个地方都有其独特的历史和魅力，希望您喜欢探索这座美丽的城市！
```

```output
llama_print_timings:        load time =     250.46 ms
llama_print_timings:      sample time =      56.40 ms /   144 runs   (    0.39 ms per token,  2553.37 tokens per second)
llama_print_timings: prompt eval time =    1444.25 ms /    47 tokens (   30.73 ms per token,    32.54 tokens per second)
llama_print_timings:        eval time =    8832.02 ms /   143 runs   (   61.76 ms per token,    16.19 tokens per second)
llama_print_timings:       total time =   10645.94 ms
```

```python
print(chain.run(text="Tell me more about #2."))
```

```output
Llama.generate: prefix-match hit
```

```output
  当然！圣史蒂芬大教堂（也称斯蒂芬大教堂）是一座位于奥地利维也纳市中心的哥特式大教堂，是该市最具代表性的地标之一。
以下是关于圣史蒂芬大教堂的一些有趣事实：
1. 历史：圣史蒂芬大教堂的建造始于12世纪，在一座前罗曼式教堂的遗址上，历时600多年才完工。大教堂在其历史上进行过多次翻修和扩建，其中最重要的一次翻修发生在19世纪。
2. 建筑风格：圣史蒂芬大教堂采用哥特式风格，以其高耸的尖塔、尖拱和精美的石雕而闻名。大教堂融合了罗曼式、哥特式和巴洛克式元素，形成了独特的风格。
3. 设计：大教堂的设计基于十字架平面，有一条长长的中殿和两条较短的侧殿。主祭坛是
```

```output
llama_print_timings:        load time =     250.46 ms
llama_print_timings:      sample time =     100.60 ms /   256 runs   (    0.39 ms per token,  2544.73 tokens per second)
llama_print_timings: prompt eval time =    5128.71 ms /   160 tokens (   32.05 ms per token,    31.20 tokens per second)
llama_print_timings:        eval time =   16193.02 ms /   255 runs   (   63.50 ms per token,    15.75 tokens per second)
llama_print_timings:       total time =   21988.57 ms
```