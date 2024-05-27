# Gradio

在 `Hugging Face Spaces` 上有成千上万个 `Gradio` 应用。这个库让它们触手可及 🦾

具体来说，`gradio-tools` 是一个 Python 库，用于将 `Gradio` 应用转换为可以被大型语言模型 (LLM) 基于代理使用的工具，以完成其任务。例如，LLM 可以使用 `Gradio` 工具来转录它在网上找到的语音录音，然后为您进行总结。或者它可以使用另一个 `Gradio` 工具来对您的 Google Drive 上的文档应用 OCR，然后回答相关问题。

如果您想使用一个不是预先构建的工具空间，创建您自己的工具非常容易。请参阅 `gradio-tools` 文档的这一部分，了解如何操作。欢迎所有贡献！

```python
%pip install --upgrade --quiet  gradio_tools
```

## 使用工具

```python
from gradio_tools.tools import StableDiffusionTool
```

```python
local_file_path = StableDiffusionTool().langchain.run(
    "请创建一张狗骑滑板的照片"
)
local_file_path
```

```output
已加载为 API: https://gradio-client-demos-stable-diffusion.hf.space ✔
任务状态: Status.STARTING 预计时间: None
```

```output
'/Users/harrisonchase/workplace/langchain/docs/modules/agents/tools/integrations/b61c1dd9-47e2-46f1-a47c-20d27640993d/tmp4ap48vnm.jpg'
```

```python
from PIL import Image
```

```python
im = Image.open(local_file_path)
```

```python
from IPython.display import display
display(im)
```

## 在代理中使用

```python
from gradio_tools.tools import (
    ImageCaptioningTool,
    StableDiffusionPromptGeneratorTool,
    StableDiffusionTool,
    TextToVideoTool,
)
from langchain.agents import initialize_agent
from langchain.memory import ConversationBufferMemory
from langchain_openai import OpenAI
llm = OpenAI(temperature=0)
memory = ConversationBufferMemory(memory_key="chat_history")
tools = [
    StableDiffusionTool().langchain,
    ImageCaptioningTool().langchain,
    StableDiffusionPromptGeneratorTool().langchain,
    TextToVideoTool().langchain,
]
agent = initialize_agent(
    tools, llm, memory=memory, agent="conversational-react-description", verbose=True
)
output = agent.run(
    input=(
        "请创建一张狗骑滑板的照片，但在使用图像生成器之前改进我的提示。"
        "请为生成的图像加上标题，并使用改进的提示为其创建视频。"
    )
)
```

```output
已加载为 API: https://gradio-client-demos-stable-diffusion.hf.space ✔
已加载为 API: https://taesiri-blip-2.hf.space ✔
已加载为 API: https://microsoft-promptist.hf.space ✔
已加载为 API: https://damo-vilab-modelscope-text-to-video-synthesis.hf.space ✔
> 进入新的 AgentExecutor 链...
思考: 我需要使用一个工具吗？是
行动: StableDiffusionPromptGenerator
行动输入: 一只狗骑滑板
任务状态: Status.STARTING 预计时间: None
观察: 一只狗骑滑板，数字绘画，artstation，概念艺术，光滑，清晰焦点，插图，artgerm 和 greg rutkowski 和 alphonse mucha 的艺术
思考: 我需要使用一个工具吗？是
行动: StableDiffusion
行动输入: 一只狗骑滑板，数字绘画，artstation，概念艺术，光滑，清晰焦点，插图，artgerm 和 greg rutkowski 和 alphonse mucha 的艺术
任务状态: Status.STARTING 预计时间: None
任务状态: Status.PROCESSING 预计时间: None
观察: /Users/harrisonchase/workplace/langchain/docs/modules/agents/tools/integrations/2e280ce4-4974-4420-8680-450825c31601/tmpfmiz2g1c.jpg
思考: 我需要使用一个工具吗？是
行动: ImageCaptioner
行动输入: /Users/harrisonchase/workplace/langchain/docs/modules/agents/tools/integrations/2e280ce4-4974-4420-8680-450825c31601/tmpfmiz2g1c.jpg
任务状态: Status.STARTING 预计时间: None
观察: 一幅狗坐在滑板上的绘画
思考: 我需要使用一个工具吗？是
行动: TextToVideo
行动输入: 一幅狗坐在滑板上的绘画
任务状态: Status.STARTING 预计时间: None
由于应用程序流量过大，预测将需要大约 73 秒。为了更快地进行预测而不必等待排队，您可以使用以下方法复制空间：Client.duplicate(damo-vilab/modelscope-text-to-video-synthesis)
任务状态: Status.IN_QUEUE 预计时间: 73.89824726581574
由于应用程序流量过大，预测将需要大约 42 秒。为了更快地进行预测而不必等待排队，您可以使用以下方法复制空间：Client.duplicate(damo-vilab/modelscope-text-to-video-synthesis)
任务状态: Status.IN_QUEUE 预计时间: 42.49370198879602
任务状态: Status.IN_QUEUE 预计时间: 21.314297944849187
观察: /var/folders/bm/ylzhm36n075cslb9fvvbgq640000gn/T/tmp5snj_nmzf20_cb3m.mp4
思考: 我需要使用一个工具吗？不需要
AI: 这是一段狗坐在滑板上的绘画视频。
> 链结束。
```