# 信心满满

[DeepEval](https://confident-ai.com) 软件包用于对大型语言模型进行单元测试。

通过使用 Confident，每个人都可以通过更快的迭代过程构建健壮的语言模型，同时结合了单元测试和集成测试。我们为迭代的每个步骤提供支持，从合成数据创建到测试。

在本指南中，我们将演示如何测试和评估语言模型的性能。我们展示了如何使用我们的回调来衡量性能，以及如何定义自己的度量标准并将其记录在我们的仪表板中。

DeepEval 还提供了：

- 如何生成合成数据

- 如何衡量性能

- 一个仪表板，用于随时间监控和审阅结果

## 安装和设置

```python
%pip install --upgrade --quiet  langchain langchain-openai deepeval langchain-chroma
```

### 获取 API 凭据

要获取 DeepEval 的 API 凭据，请按照以下步骤操作：

1. 访问 https://app.confident-ai.com

2. 点击“组织”

3. 复制 API 密钥

登录时，还会要求您设置 `implementation` 名称。实现名称是必需的，用于描述实现的类型。（考虑一下您想如何命名您的项目。我们建议使其具有描述性。）

```python
!deepeval login
```

### 设置 DeepEval

您可以默认使用 `DeepEvalCallbackHandler` 来设置要跟踪的度量标准。但是，目前对度量标准的支持有限（很快会添加更多）。目前支持：

- [答案相关性](https://docs.confident-ai.com/docs/measuring_llm_performance/answer_relevancy)

- [偏见](https://docs.confident-ai.com/docs/measuring_llm_performance/debias)

- [有毒性](https://docs.confident-ai.com/docs/measuring_llm_performance/non_toxic)

```python
from deepeval.metrics.answer_relevancy import AnswerRelevancy
# 在这里，我们希望确保答案是最小相关的
answer_relevancy_metric = AnswerRelevancy(minimum_score=0.5)
```

## 入门

要使用 `DeepEvalCallbackHandler`，我们需要 `implementation_name`。

```python
from langchain_community.callbacks.confident_callback import DeepEvalCallbackHandler
deepeval_callback = DeepEvalCallbackHandler(
    implementation_name="langchainQuickstart", metrics=[answer_relevancy_metric]
)
```

### 场景 1：输入到 LLM

然后，您可以将其输入到使用 OpenAI 的 LLM 中。

```python
from langchain_openai import OpenAI
llm = OpenAI(
    temperature=0,
    callbacks=[deepeval_callback],
    verbose=True,
    openai_api_key="<YOUR_API_KEY>",
)
output = llm.generate(
    [
        "What is the best evaluation tool out there? (no bias at all)",
    ]
)
```

```output
LLMResult(generations=[[Generation(text='\n\nQ: What did the fish say when he hit the wall? \nA: Dam.', generation_info={'finish_reason': 'stop', 'logprobs': None})], [Generation(text='\n\nThe Moon \n\nThe moon is high in the midnight sky,\nSparkling like a star above.\nThe night so peaceful, so serene,\nFilling up the air with love.\n\nEver changing and renewing,\nA never-ending light of grace.\nThe moon remains a constant view,\nA reminder of life’s gentle pace.\n\nThrough time and space it guides us on,\nA never-fading beacon of hope.\nThe moon shines down on us all,\nAs it continues to rise and elope.', generation_info={'finish_reason': 'stop', 'logprobs': None})], [Generation(text='\n\nQ. What did one magnet say to the other magnet?\nA. "I find you very attractive!"', generation_info={'finish_reason': 'stop', 'logprobs': None})], [Generation(text="\n\nThe world is charged with the grandeur of God.\nIt will flame out, like shining from shook foil;\nIt gathers to a greatness, like the ooze of oil\nCrushed. Why do men then now not reck his rod?\n\nGenerations have trod, have trod, have trod;\nAnd all is seared with trade; bleared, smeared with toil;\nAnd wears man's smudge and shares man's smell: the soil\nIs bare now, nor can foot feel, being shod.\n\nAnd for all this, nature is never spent;\nThere lives the dearest freshness deep down things;\nAnd though the last lights off the black West went\nOh, morning, at the brown brink eastward, springs —\n\nBecause the Holy Ghost over the bent\nWorld broods with warm breast and with ah! bright wings.\n\n~Gerard Manley Hopkins", generation_info={'finish_reason': 'stop', 'logprobs': None})], [Generation(text='\n\nQ: What did one ocean say to the other ocean?\nA: Nothing, they just waved.', generation_info={'finish_reason': 'stop', 'logprobs': None})], [Generation(text="\n\nA poem for you\n\nOn a field of green\n\nThe sky so blue\n\nA gentle breeze, the sun above\n\nA beautiful world, for us to love\n\nLife is a journey, full of surprise\n\nFull of joy and full of surprise\n\nBe brave and take small steps\n\nThe future will be revealed with depth\n\nIn the morning, when dawn arrives\n\nA fresh start, no reason to hide\n\nSomewhere down the road, there's a heart that beats\n\nBelieve in yourself, you'll always succeed.", generation_info={'finish_reason': 'stop', 'logprobs': None})]], llm_output={'token_usage': {'completion_tokens': 504, 'total_tokens': 528, 'prompt_tokens': 24}, 'model_name': 'text-davinci-003'})
```

你可以通过调用 `is_successful()` 方法来检查度量标准是否成功。

```python
answer_relevancy_metric.is_successful()
# 返回 True/False
```

运行完上述代码后，你应该能够在下方看到我们的仪表板。

![Dashboard](https://docs.confident-ai.com/assets/images/dashboard-screenshot-b02db73008213a211b1158ff052d969e.png)

### 场景2：在没有回调的情况下跟踪链中的LLM

要在没有回调的情况下跟踪链中的LLM，你可以在链的末端插入它。

我们可以首先定义一个简单的链，如下所示。

```python
import requests
from langchain.chains import RetrievalQA
from langchain_chroma import Chroma
from langchain_community.document_loaders import TextLoader
from langchain_openai import OpenAI, OpenAIEmbeddings
from langchain_text_splitters import CharacterTextSplitter
text_file_url = "https://raw.githubusercontent.com/hwchase17/chat-your-data/master/state_of_the_union.txt"
openai_api_key = "sk-XXX"
with open("state_of_the_union.txt", "w") as f:
    response = requests.get(text_file_url)
    f.write(response.text)
loader = TextLoader("state_of_the_union.txt")
documents = loader.load()
text_splitter = CharacterTextSplitter(chunk_size=1000, chunk_overlap=0)
texts = text_splitter.split_documents(documents)
embeddings = OpenAIEmbeddings(openai_api_key=openai_api_key)
docsearch = Chroma.from_documents(texts, embeddings)
qa = RetrievalQA.from_chain_type(
    llm=OpenAI(openai_api_key=openai_api_key),
    chain_type="stuff",
    retriever=docsearch.as_retriever(),
)
# 提供一个新的问答管道
query = "Who is the president?"
result = qa.run(query)
```

在定义链之后，你可以手动检查答案的相似性。

```python
answer_relevancy_metric.measure(result, query)
answer_relevancy_metric.is_successful()
```

### 接下来做什么？

你可以在[这里](https://docs.confident-ai.com/docs/quickstart/custom-metrics)创建自己的自定义度量标准。

DeepEval 还提供其他功能，例如能够[自动创建单元测试](https://docs.confident-ai.com/docs/quickstart/synthetic-data-creation)，[测试幻觉](https://docs.confident-ai.com/docs/measuring_llm_performance/factual_consistency)等。

如果你感兴趣，请查看我们的 Github 仓库 [https://github.com/confident-ai/deepeval](https://github.com/confident-ai/deepeval)。我们欢迎任何改进LLM性能的PR和讨论。