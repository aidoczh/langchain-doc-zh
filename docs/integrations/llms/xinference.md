# Xorbits 推理（Xinference）

[Xinference](https://github.com/xorbitsai/inference) 是一个功能强大且多才多艺的库，旨在为LLMs、语音识别模型和多模型提供服务，即使在您的笔记本电脑上也能运行。它支持与GGML兼容的各种模型，如chatglm、baichuan、whisper、vicuna、orca等等。本笔记演示了如何在LangChain中使用Xinference。

## 安装

通过PyPI安装 `Xinference`：

```python
%pip install --upgrade --quiet  "xinference[all]"
```

## 本地部署或在分布式集群中部署Xinference。

要进行本地部署，请运行 `xinference`。

要在集群中部署Xinference，首先使用 `xinference-supervisor` 启动Xinference监督员。您还可以使用选项 -p 指定端口和 -H 指定主机。默认端口为9997。

然后，在要运行Xinference的每台服务器上使用 `xinference-worker` 启动Xinference工作程序。

您可以查阅[Xinference](https://github.com/xorbitsai/inference)的README文件获取更多信息。

## 包装器

要在LangChain中使用Xinference，您需要首先启动一个模型。您可以使用命令行界面（CLI）来执行此操作：

```python
!xinference launch -n vicuna-v1.3 -f ggmlv3 -q q4_0
```

```output
Model uid: 7167b2b0-2a04-11ee-83f0-d29396a3f064
```

系统会返回一个模型UID供您使用。现在您可以在LangChain中使用Xinference：

```python
from langchain_community.llms import Xinference
llm = Xinference(
    server_url="http://0.0.0.0:9997", model_uid="7167b2b0-2a04-11ee-83f0-d29396a3f064"
)
llm(
    prompt="Q: where can we visit in the capital of France? A:",
    generate_config={"max_tokens": 1024, "stream": True},
)
```

```output
' 您可以参观法国首都巴黎的埃菲尔铁塔、巴黎圣母院、卢浮宫等许多历史遗迹。'
```

### 与LLMChain集成

```python
from langchain.chains import LLMChain
from langchain_core.prompts import PromptTemplate
template = "Where can we visit in the capital of {country}?"
prompt = PromptTemplate.from_template(template)
llm_chain = LLMChain(prompt=prompt, llm=llm)
generated = llm_chain.run(country="France")
print(generated)
```

```output
A: 您可以在巴黎参观许多地方，如埃菲尔铁塔、卢浮宫、巴黎圣母院、香榭丽舍大街、蒙马特、圣心大教堂和凡尔赛宫。
```

最后，在不需要使用模型时终止它：

```python
!xinference terminate --model-uid "7167b2b0-2a04-11ee-83f0-d29396a3f064"
```