# PipelineAI

[PipelineAI](https://pipeline.ai) 允许您在云中以规模运行您的机器学习模型。它还提供对[几个LLM模型](https://pipeline.ai)的API访问。

这份笔记将介绍如何在[PipelineAI](https://docs.pipeline.ai/docs)中使用Langchain。

## PipelineAI示例

[这个示例展示了PipelineAI如何与LangChain集成](https://docs.pipeline.ai/docs/langchain)，由PipelineAI创建。

## 设置

使用`PipelineAI` API，也称为`Pipeline Cloud`，需要安装`pipeline-ai`库。使用`pip install pipeline-ai`进行安装。

```python
# 安装包
%pip install --upgrade --quiet  pipeline-ai
```

## 示例

### 导入

```python
import os
from langchain.chains import LLMChain
from langchain_community.llms import PipelineAI
from langchain_core.prompts import PromptTemplate
```

### 设置环境API密钥

确保从PipelineAI获取您的API密钥。查看[云快速入门指南](https://docs.pipeline.ai/docs/cloud-quickstart)。您将获得为期30天的免费试用，可用于测试不同模型的无服务器GPU计算共10小时。

```python
os.environ["PIPELINE_API_KEY"] = "YOUR_API_KEY_HERE"
```

## 创建PipelineAI实例

在实例化PipelineAI时，您需要指定要使用的流水线的ID或标签，例如`pipeline_key = "public/gpt-j:base"`。然后，您可以选择传递其他特定于流水线的关键字参数：

```python
llm = PipelineAI(pipeline_key="YOUR_PIPELINE_KEY", pipeline_kwargs={...})
```

### 创建一个提示模板

我们将为问题和答案创建一个提示模板。

```python
template = """Question: {question}
Answer: 让我们逐步思考。"""
prompt = PromptTemplate.from_template(template)
```

### 初始化LLMChain

```python
llm_chain = LLMChain(prompt=prompt, llm=llm)
```

### 运行LLMChain

提供一个问题并运行LLMChain。

```python
question = "贾斯汀·比伯出生年份的超级碗冠军是哪支NFL球队？"
llm_chain.run(question)
```