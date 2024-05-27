# Argilla

[Argilla](https://argilla.io/) 是一个用于 LLM 的开源数据整理平台。

使用 Argilla，每个人都可以通过更快的数据整理，结合人工和机器反馈，构建强大的语言模型。我们为 MLOps 周期中的每个步骤提供支持，从数据标记到模型监控。

在本指南中，我们将演示如何跟踪您的 LLM 的输入和响应，以生成 Argilla 中的数据集，使用 `ArgillaCallbackHandler`。

跟踪您的 LLM 的输入和输出以生成未来微调的数据集非常有用。当您使用 LLM 为特定任务生成数据时，比如问答、摘要或翻译，这一点尤其有用。

## 安装和设置

```python
%pip install --upgrade --quiet langchain langchain-openai argilla
```

### 获取 API 凭据

要获取 Argilla 的 API 凭据，请按照以下步骤操作：

1. 进入您的 Argilla 用户界面。

2. 点击您的个人资料图片，进入“我的设置”。

3. 然后复制 API 密钥。

在 Argilla 中，API URL 将与您的 Argilla 用户界面的 URL 相同。

要获取 OpenAI 的 API 凭据，请访问 [https://platform.openai.com/account/api-keys](https://platform.openai.com/account/api-keys)。

```python
import os
os.environ["ARGILLA_API_URL"] = "..."
os.environ["ARGILLA_API_KEY"] = "..."
os.environ["OPENAI_API_KEY"] = "..."
```

### 设置 Argilla

要使用 `ArgillaCallbackHandler`，我们需要在 Argilla 中创建一个新的 `FeedbackDataset`，以跟踪您的 LLM 实验。请使用以下代码：

```python
import argilla as rg
```

```python
from packaging.version import parse as parse_version
if parse_version(rg.__version__) < parse_version("1.8.0"):
    raise RuntimeError(
        "`FeedbackDataset` 仅在 Argilla v1.8.0 或更高版本中可用，请升级 `argilla`，命令为 `pip install argilla --upgrade`。"
    )
```

```python
dataset = rg.FeedbackDataset(
    fields=[
        rg.TextField(name="prompt"),
        rg.TextField(name="response"),
    ],
    questions=[
        rg.RatingQuestion(
            name="response-rating",
            description="您如何评价响应的质量？",
            values=[1, 2, 3, 4, 5],
            required=True,
        ),
        rg.TextQuestion(
            name="response-feedback",
            description="对响应有何反馈？",
            required=False,
        ),
    ],
    guidelines="您被要求评价响应的质量并提供反馈。",
)
rg.init(
    api_url=os.environ["ARGILLA_API_URL"],
    api_key=os.environ["ARGILLA_API_KEY"],
)
dataset.push_to_argilla("langchain-dataset")
```

> 📌 注意：目前仅支持提示-响应对作为 `FeedbackDataset.fields`，因此 `ArgillaCallbackHandler` 将仅跟踪提示，即 LLM 输入，以及响应，即 LLM 输出。

## 跟踪

要使用 `ArgillaCallbackHandler`，您可以使用以下代码，或者只需复制以下部分示例中呈现的示例之一。

```python
from langchain_community.callbacks.argilla_callback import ArgillaCallbackHandler
argilla_callback = ArgillaCallbackHandler(
    dataset_name="langchain-dataset",
    api_url=os.environ["ARGILLA_API_URL"],
    api_key=os.environ["ARGILLA_API_KEY"],
)
```

### 场景 1：跟踪 LLM

首先，让我们运行几次单个 LLM，并在 Argilla 中捕获生成的提示-响应对。

```python
from langchain_core.callbacks.stdout import StdOutCallbackHandler
from langchain_openai import OpenAI
argilla_callback = ArgillaCallbackHandler(
    dataset_name="langchain-dataset",
    api_url=os.environ["ARGILLA_API_URL"],
    api_key=os.environ["ARGILLA_API_KEY"],
)
callbacks = [StdOutCallbackHandler(), argilla_callback]
llm = OpenAI(temperature=0.9, callbacks=callbacks)
llm.generate(["Tell me a joke", "Tell me a poem"] * 3)
```

```output
LLMResult(generations=[[Generation(text='\n\nQ: What did the fish say when he hit the wall? \nA: Dam.', generation_info={'finish_reason': 'stop', 'logprobs': None})], [Generation(text='\n\nThe Moon \n\nThe moon is high in the midnight sky,\nSparkling like a star above.\nThe night so peaceful, so serene,\nFilling up the air with love.\n\nEver changing and renewing,\nA never-ending light of grace.\nThe moon remains a constant view,\nA reminder of life’s gentle pace.\n\nThrough time and space it guides us on,\nA never-fading beacon of hope.\nThe moon shines down on us all,\nAs it continues to rise and elope.', generation_info={'finish_reason': 'stop', 'logprobs': None})], [Generation(text='\n\nQ. What did one magnet say to the other magnet?\nA. "I find you very attractive!"', generation_info={'finish_reason': 'stop', 'logprobs': None})], [Generation(text="\n\nThe world is charged with the grandeur of God.\nIt will flame out, like shining from shook foil;\nIt gathers to a greatness, like the ooze of oil\nCrushed. Why do men then now not reck his rod?\n\nGenerations have trod, have trod, have trod;\nAnd all is seared with trade; bleared, smeared with toil;\nAnd wears man's smudge and shares man's smell: the soil\nIs bare now, nor can foot feel, being shod.\n\nAnd for all this, nature is never spent;\nThere lives the dearest freshness deep down things;\nAnd though the last lights off the black West went\nOh, morning, at the brown brink eastward, springs —\n\nBecause the Holy Ghost over the bent\nWorld broods with warm breast and with ah! bright wings.\n\n~Gerard Manley Hopkins", generation_info={'finish_reason': 'stop', 'logprobs': None})], [Generation(text='\n\nQ: What did one ocean say to the other ocean?\nA: Nothing, they just waved.', generation_info={'finish_reason': 'stop', 'logprobs': None})], [Generation(text="\n\nA poem for you\n\nOn a field of green\n\nThe sky so blue\n\nA gentle breeze, the sun above\n\nA beautiful world, for us to love\n\nLife is a journey, full of surprise\n\nFull of joy and full of surprise\n\nBe brave and take small steps\n\nThe future will be revealed with depth\n\nIn the morning, when dawn arrives\n\nA fresh start, no reason to hide\n\nSomewhere down the road, there's a heart that beats\n\nBelieve in yourself, you'll always succeed.", generation_info={'finish_reason': 'stop', 'logprobs': None})]], llm_output={'token_usage': {'completion_tokens': 504, 'total_tokens': 528, 'prompt_tokens': 24}, 'model_name': 'text-davinci-003'})
```

![Argilla UI with LangChain LLM input-response](https://docs.argilla.io/en/latest/_images/llm.png)

### 场景2：在链中跟踪LLM

然后，我们可以使用提示模板创建一个链，然后在Argilla中跟踪初始提示和最终响应。

```python
from langchain.chains import LLMChain
from langchain_core.callbacks.stdout import StdOutCallbackHandler
from langchain_core.prompts import PromptTemplate
from langchain_openai import OpenAI
argilla_callback = ArgillaCallbackHandler(
    dataset_name="langchain-dataset",
    api_url=os.environ["ARGILLA_API_URL"],
    api_key=os.environ["ARGILLA_API_KEY"],
)
callbacks = [StdOutCallbackHandler(), argilla_callback]
llm = OpenAI(temperature=0.9, callbacks=callbacks)
template = """You are a playwright. Given the title of play, it is your job to write a synopsis for that title.
Title: {title}
Playwright: This is a synopsis for the above play:"""
prompt_template = PromptTemplate(input_variables=["title"], template=template)
synopsis_chain = LLMChain(llm=llm, prompt=prompt_template, callbacks=callbacks)
test_prompts = [{"title": "Documentary about Bigfoot in Paris"}]
synopsis_chain.apply(test_prompts)
```

```output
> 进入新的LLMChain链...
格式化后的提示：
你是一位剧作家。给定剧名，你的工作是为该剧写一个简介。
标题：Documentary about Bigfoot in Paris
剧作家：这是上述剧的简介：
> 链结束。
```

```output
[{'text': "\n\nDocumentary about Bigfoot in Paris focuses on the story of a documentary filmmaker and their search for evidence of the legendary Bigfoot creature in the city of Paris. The play follows the filmmaker as they explore the city, meeting people from all walks of life who have had encounters with the mysterious creature. Through their conversations, the filmmaker unravels the story of Bigfoot and finds out the truth about the creature's presence in Paris. As the story progresses, the filmmaker learns more and more about the mysterious creature, as well as the different perspectives of the people living in the city, and what they think of the creature. In the end, the filmmaker's findings lead them to some surprising and heartwarming conclusions about the creature's existence and the importance it holds in the lives of the people in Paris."}]
```

![Argilla UI with LangChain Chain input-response](https://docs.argilla.io/en/latest/_images/chain.png)

### 场景3：使用带有工具的代理

最后，作为更高级的工作流程，您可以创建一个使用一些工具的代理。因此，`ArgillaCallbackHandler`将跟踪输入和输出，但不会跟踪中间步骤/思考，因此给定一个提示，我们记录原始提示和给定提示的最终响应。

> 请注意，对于此场景，我们将使用Google搜索API（Serp API），因此您需要安装`google-search-results`，即`pip install google-search-results`，并将Serp API密钥设置为`os.environ["SERPAPI_API_KEY"] = "..."`（您可以在https://serpapi.com/dashboard找到它），否则下面的示例将无法工作。

```python
from langchain.agents import AgentType, initialize_agent, load_tools
from langchain_core.callbacks.stdout import StdOutCallbackHandler
from langchain_openai import OpenAI
argilla_callback = ArgillaCallbackHandler(
    dataset_name="langchain-dataset",
    api_url=os.environ["ARGILLA_API_URL"],
    api_key=os.environ["ARGILLA_API_KEY"],
)
callbacks = [StdOutCallbackHandler(), argilla_callback]
llm = OpenAI(temperature=0.9, callbacks=callbacks)
tools = load_tools(["serpapi"], llm=llm, callbacks=callbacks)
agent = initialize_agent(
    tools,
    llm,
    agent=AgentType.ZERO_SHOT_REACT_DESCRIPTION,
    callbacks=callbacks,
)
agent.run("Who was the first president of the United States of America?")
```

```output
> 进入新的AgentExecutor链...
我需要回答一个历史问题
动作：搜索
动作输入："who was the first president of the United States of America" 
观察：乔治·华盛顿
思考：乔治·华盛顿是第一任总统
最终答案：乔治·华盛顿是美利坚合众国的第一任总统。
> 链结束。
```

```output
'乔治·华盛顿是美利坚合众国的第一任总统。'
```

![Argilla UI with LangChain Agent input-response](https://docs.argilla.io/en/latest/_images/agent.png)