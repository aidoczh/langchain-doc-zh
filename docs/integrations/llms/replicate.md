# 复制

[Replicate](https://replicate.com/blog/machine-learning-needs-better-tools) 在云端运行机器学习模型。我们拥有一系列开源模型，您可以用几行代码来运行。如果您正在构建自己的机器学习模型，Replicate 可以帮助您轻松地进行规模部署。

这个示例演示了如何使用 LangChain 与 `Replicate` [模型](https://replicate.com/explore) 进行交互。

## 设置

```python
# 在您编辑此笔记本时，使用魔术命令自动重新加载外部模块
%load_ext autoreload
%autoreload 2
```

要运行这个笔记本，您需要创建一个 [replicate](https://replicate.com) 账户，并安装 [replicate python 客户端](https://github.com/replicate/replicate-python)。

```python
!poetry run pip install replicate
```

```output
Collecting replicate
  Using cached replicate-0.25.1-py3-none-any.whl.metadata (24 kB)
Requirement already satisfied: httpx<1,>=0.21.0 in /Users/charlieholtz/miniconda3/envs/langchain/lib/python3.9/site-packages (from replicate) (0.24.1)
Requirement already satisfied: packaging in /Users/charlieholtz/miniconda3/envs/langchain/lib/python3.9/site-packages (from replicate) (23.2)
Requirement already satisfied: pydantic>1.10.7 in /Users/charlieholtz/miniconda3/envs/langchain/lib/python3.9/site-packages (from replicate) (1.10.14)
Requirement already satisfied: typing-extensions>=4.5.0 in /Users/charlieholtz/miniconda3/envs/langchain/lib/python3.9/site-packages (from replicate) (4.10.0)
Requirement already satisfied: certifi in /Users/charlieholtz/miniconda3/envs/langchain/lib/python3.9/site-packages (from httpx<1,>=0.21.0->replicate) (2024.2.2)
Requirement already satisfied: httpcore<0.18.0,>=0.15.0 in /Users/charlieholtz/miniconda3/envs/langchain/lib/python3.9/site-packages (from httpx<1,>=0.21.0->replicate) (0.17.3)
Requirement already satisfied: idna in /Users/charlieholtz/miniconda3/envs/langchain/lib/python3.9/site-packages (from httpx<1,>=0.21.0->replicate) (3.6)
Requirement already satisfied: sniffio in /Users/charlieholtz/miniconda3/envs/langchain/lib/python3.9/site-packages (from httpx<1,>=0.21.0->replicate) (1.3.1)
Requirement already satisfied: h11<0.15,>=0.13 in /Users/charlieholtz/miniconda3/envs/langchain/lib/python3.9/site-packages (from httpcore<0.18.0,>=0.15.0->httpx<1,>=0.21.0->replicate) (0.14.0)
Requirement already satisfied: anyio<5.0,>=3.0 in /Users/charlieholtz/miniconda3/envs/langchain/lib/python3.9/site-packages (from httpcore<0.18.0,>=0.15.0->httpx<1,>=0.21.0->replicate) (3.7.1)
Requirement already satisfied: exceptiongroup in /Users/charlieholtz/miniconda3/envs/langchain/lib/python3.9/site-packages (from anyio<5.0,>=3.0->httpcore<0.18.0,>=0.15.0->httpx<1,>=0.21.0->replicate) (1.2.0)
Using cached replicate-0.25.1-py3-none-any.whl (39 kB)
Installing collected packages: replicate
Successfully installed replicate-0.25.1
```

```python
# 获取一个令牌：https://replicate.com/account
from getpass import getpass
REPLICATE_API_TOKEN = getpass()
```

```python
import os
os.environ["REPLICATE_API_TOKEN"] = REPLICATE_API_TOKEN
```

```python
from langchain.chains import LLMChain
from langchain_community.llms import Replicate
from langchain_core.prompts import PromptTemplate
```

## 调用模型

在 [replicate 探索页面](https://replicate.com/explore) 找到一个模型，然后按照以下格式粘贴模型名称和版本：model_name/version。

例如，这里是 [`Meta Llama 3`](https://replicate.com/meta/meta-llama-3-8b-instruct)。

```python
llm = Replicate(
    model="meta/meta-llama-3-8b-instruct",
    model_kwargs={"temperature": 0.75, "max_length": 500, "top_p": 1},
)
prompt = """
User: Answer the following yes/no question by reasoning step by step. Can a dog drive a car?
Assistant:
"""
llm(prompt)
```

```output
"让我们逐步分解：\n\n1. 狗是一种生物，具体来说是哺乳动物。\n2. 狗没有操作车辆（如汽车）所需的认知能力或物理特征。\n3. 操作汽车需要复杂的心理和物理能力，包括：\n\t* 理解交通法规和规则\n\t* 能够阅读和理解道路标志\n\t* 能够快速准确地做出决策\n\t* 能够操作车辆的控制装置（例如方向盘、踏板）\n4. 狗没有这些能力。它们无法阅读或理解书面语言，更不用说复杂的交通法规了。\n5. 狗也缺乏操作车辆控制装置的物理灵活性和协调性。它们的爪子和爪子不适合抓握或操作方向盘或踏板等小型精细物体。\n6. 因此，狗无法驾驶汽车。\n\n答案：不可以。"
```

作为另一个例子，对于这个[dolly模型](https://replicate.com/replicate/dolly-v2-12b)，点击API选项卡。模型名称/版本为：`replicate/dolly-v2-12b:ef0e1aefc61f8e096ebe4db6b2bacc297daf2ef6899f0f7e001ec445893500e5`

只有`model`参数是必需的，但在初始化时我们可以添加其他模型参数。

例如，如果我们正在运行稳定扩散并希望更改图像尺寸：

```
Replicate(model="stability-ai/stable-diffusion:db21e45d3f7023abc2a46ee38a23973f6dce16bb082a930b0c49861f96d1e5bf", input={'image_dimensions': '512x512'})
```

*请注意，模型的第一个输出将被返回。*

```python
llm = Replicate(
    model="replicate/dolly-v2-12b:ef0e1aefc61f8e096ebe4db6b2bacc297daf2ef6899f0f7e001ec445893500e5"
)
```

```python
prompt = """
通过逐步推理回答以下是/否问题。
一只狗能开车吗？
"""
llm(prompt)
```

```output
'不行，狗缺乏操作机动车辆所需的某些脑功能。它们无法集中注意力并及时做出加速或刹车的反应。此外，它们没有足够的肌肉控制来正确操作方向盘。\n\n'
```

我们可以使用这种语法调用任何复制模型。例如，我们可以调用稳定扩散。

```python
text2image = Replicate(
    model="stability-ai/stable-diffusion:db21e45d3f7023abc2a46ee38a23973f6dce16bb082a930b0c49861f96d1e5bf",
    model_kwargs={"image_dimensions": "512x512"},
)
```

```python
image_output = text2image("毕加索画的一只骑摩托车的猫")
image_output
```

```output
'https://pbxt.replicate.delivery/bqQq4KtzwrrYL9Bub9e7NvMTDeEMm5E9VZueTXkLE7kWumIjA/out-0.png'
```

该模型会输出一个URL。让我们将其渲染。

```python
!poetry run pip install Pillow
```

```output
Requirement already satisfied: Pillow in /Users/bagatur/langchain/.venv/lib/python3.9/site-packages (9.5.0)
[notice] A new release of pip is available: 23.2 -> 23.2.1
[notice] To update, run: pip install --upgrade pip
```

```python
from io import BytesIO
import requests
from PIL import Image
response = requests.get(image_output)
img = Image.open(BytesIO(response.content))
img
```

## 流式响应

您可以选择在生成过程中流式传输响应，这有助于向用户展示耗时生成的交互性。有关更多信息，请参阅[流式传输](/docs/how_to/streaming_llm)的详细文档。

```python
from langchain_core.callbacks import StreamingStdOutCallbackHandler
llm = Replicate(
    streaming=True,
    callbacks=[StreamingStdOutCallbackHandler()],
    model="a16z-infra/llama13b-v2-chat:df7690f1994d94e96ad9d568eac121aecf50684a0b0963b25a41cc40061269e5",
    model_kwargs={"temperature": 0.75, "max_length": 500, "top_p": 1},
)
prompt = """
用户：通过逐步推理回答以下是/否问题。一只狗能开车吗？
助手：
"""
_ = llm.invoke(prompt)
```

```output
1. 狗没有操作车辆的身体能力。
```

# 停止序列

您还可以指定停止序列。如果您对将要生成的内容有明确的停止序列，并且无论您是否处于流式模式，只要达到一个或多个停止序列，最好（更便宜、更快！）取消生成，而不是让模型继续到指定的`max_length`。停止序列无论您是否处于流式模式，都可以工作，而且复制只会为达到停止序列之前的生成收费。

```python
import time
llm = Replicate(
    model="a16z-infra/llama13b-v2-chat:df7690f1994d94e96ad9d568eac121aecf50684a0b0963b25a41cc40061269e5",
    model_kwargs={"temperature": 0.01, "max_length": 500, "top_p": 1},
)
prompt = """
用户：学习Python的最佳方法是什么？
助手：
"""
start_time = time.perf_counter()
raw_output = llm.invoke(prompt)  # 原始输出，无停止
end_time = time.perf_counter()
print(f"原始输出：\n {raw_output}")
print(f"原始输出运行时间：{end_time - start_time} 秒")
start_time = time.perf_counter()
stopped_output = llm.invoke(prompt, stop=["\n\n"])  # 在双换行符上停止
end_time = time.perf_counter()
print(f"停止输出：\n {stopped_output}")
print(f"停止输出运行时间：{end_time - start_time} 秒")
```

```output
原始输出：
有几种学习Python的方法，对于您来说最佳的方法将取决于您的学习风格和目标。以下是一些建议：
1. 在线教程和课程：网站如Codecademy、Coursera和edX提供互动编程课程，可以帮助您开始学习Python。这些课程通常面向初学者，涵盖Python编程的基础知识。
2. 书籍：有许多可用的书籍可以教您Python，从入门级文本到更高级的手册。一些流行的选择包括Eric Matthes的《Python Crash Course》，Al Sweigart的《用Python自动化繁琐工作》和Wes McKinney的《Python数据分析》。
3. 视频：YouTube和其他视频平台上有大量关于Python编程的教程和讲座。许多这些视频是由经验丰富的程序员创建的，可以提供Python概念的详细解释和示例。
4. 练习：学习Python的最佳方法之一是练习编写代码。从简单的程序开始，逐渐转向更复杂的项目。随着经验的积累，您将更加熟悉这种语言，并对其功能有更好的理解。
5. 加入社区：有许多致力于Python编程的在线社区和论坛，如Reddit的r/learnpython社区。这些社区可以在您学习过程中提供支持、资源和反馈。
6. 参加在线课程：许多大学和组织提供关于Python编程的在线课程。这些课程可以提供结构化的学习体验，通常包括练习和作业，帮助您练习您的技能。
7. 使用Python IDE：集成开发环境（IDE）是一种提供编写、调试和测试代码界面的软件应用程序。流行的Python IDE包括PyCharm、Visual Studio Code和Spyder。这些工具可以帮助您编写更高效的代码，并提供诸如代码完成、调试和项目管理等功能。
您认为上述哪种选项是学习Python的最佳方法？
原始输出运行时间：25.27470933299992 秒
停止输出：
有几种学习Python的方法，对于您来说最佳的方法将取决于您的学习风格和目标。以下是一些建议：
停止输出运行时间：25.77039254200008 秒
```

## 调用链

langchain 的整个目的就是要进行链式调用！以下是如何实现的示例。

```python
from langchain.chains import SimpleSequentialChain
```

首先，让我们将该模型的 LLM 定义为 flan-5，并将 text2image 定义为一个稳定的扩散模型。

```python
dolly_llm = Replicate(
    model="replicate/dolly-v2-12b:ef0e1aefc61f8e096ebe4db6b2bacc297daf2ef6899f0f7e001ec445893500e5"
)
text2image = Replicate(
    model="stability-ai/stable-diffusion:db21e45d3f7023abc2a46ee38a23973f6dce16bb082a930b0c49861f96d1e5bf"
)
```

链中的第一个提示

```python
prompt = PromptTemplate(
    input_variables=["product"],
    template="一个制造 {product} 的公司应该取什么好名字？",
)
chain = LLMChain(llm=dolly_llm, prompt=prompt)
```

第二个提示，用于获取公司描述的标志

```python
second_prompt = PromptTemplate(
    input_variables=["company_name"],
    template="为这家公司的标志写一个描述：{company_name}",
)
chain_two = LLMChain(llm=dolly_llm, prompt=second_prompt)
```

第三个提示，让我们根据从第二个提示输出的描述创建图像

```python
third_prompt = PromptTemplate(
    input_variables=["company_logo_description"],
    template="{company_logo_description}",
)
chain_three = LLMChain(llm=text2image, prompt=third_prompt)
```

现在让我们运行它！

```python
# 运行链，仅指定第一个链的输入变量。
overall_chain = SimpleSequentialChain(
    chains=[chain, chain_two, chain_three], verbose=True
)
catchphrase = overall_chain.run("colorful socks")
print(catchphrase)
```

```output
> 进入新的 SimpleSequentialChain 链...
Colorful socks 可以取自披头士乐队的一首歌或一种颜色（黄色、蓝色、粉色）。一个好的字母和数字组合是 6399。苹果也拥有域名 6399.com，因此这可能被保留给该公司。
一只色彩缤纷的袜子，分别以黄色、蓝色和粉色屏印数字 3、9 和 99。
https://pbxt.replicate.delivery/P8Oy3pZ7DyaAC1nbJTxNw95D1A3gCPfi2arqlPGlfG9WYTkRA/out-0.png
> 链结束。
https://pbxt.replicate.delivery/P8Oy3pZ7DyaAC1nbJTxNw95D1A3gCPfi2arqlPGlfG9WYTkRA/out-0.png
```

```python
response = requests.get(
    "https://replicate.delivery/pbxt/682XgeUlFela7kmZgPOf39dDdGDDkwjsCIJ0aQ0AO5bTbbkiA/out-0.png"
)
img = Image.open(BytesIO(response.content))
img
```