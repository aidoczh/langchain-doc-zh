# 梯度

`Gradient` 允许使用简单的 Web API 对 LLM 进行微调和获取完成。

本笔记介绍如何使用 Langchain 与 [Gradient](https://gradient.ai/)。

## 导入

```python
from langchain.chains import LLMChain
from langchain_community.llms import GradientLLM
from langchain_core.prompts import PromptTemplate
```

## 设置环境 API 密钥

确保从 Gradient AI 获取您的 API 密钥。您将获得价值 10 美元的免费信用额度，用于测试和微调不同的模型。

```python
import os
from getpass import getpass
if not os.environ.get("GRADIENT_ACCESS_TOKEN", None):
    # 在 https://auth.gradient.ai/select-workspace 下获取访问令牌
    os.environ["GRADIENT_ACCESS_TOKEN"] = getpass("gradient.ai 访问令牌:")
if not os.environ.get("GRADIENT_WORKSPACE_ID", None):
    # 在 `$ gradient workspace list` 中列出的 `ID`
    # 也可以在 https://auth.gradient.ai/select-workspace 登录后显示
    os.environ["GRADIENT_WORKSPACE_ID"] = getpass("gradient.ai 工作空间 ID:")
```

可选：验证您的环境变量 `GRADIENT_ACCESS_TOKEN` 和 `GRADIENT_WORKSPACE_ID`，以获取当前部署的模型。使用 `gradientai` Python 包。

```python
%pip install --upgrade --quiet  gradientai
```

```output
Requirement already satisfied: gradientai in /home/michi/.venv/lib/python3.10/site-packages (1.0.0)
Requirement already satisfied: aenum>=3.1.11 in /home/michi/.venv/lib/python3.10/site-packages (from gradientai) (3.1.15)
Requirement already satisfied: pydantic<2.0.0,>=1.10.5 in /home/michi/.venv/lib/python3.10/site-packages (from gradientai) (1.10.12)
Requirement already satisfied: python-dateutil>=2.8.2 in /home/michi/.venv/lib/python3.10/site-packages (from gradientai) (2.8.2)
Requirement already satisfied: urllib3>=1.25.3 in /home/michi/.venv/lib/python3.10/site-packages (from gradientai) (1.26.16)
Requirement already satisfied: typing-extensions>=4.2.0 in /home/michi/.venv/lib/python3.10/site-packages (from pydantic<2.0.0,>=1.10.5->gradientai) (4.5.0)
Requirement already satisfied: six>=1.5 in /home/michi/.venv/lib/python3.10/site-packages (from python-dateutil>=2.8.2->gradientai) (1.16.0)
```

```python
import gradientai
client = gradientai.Gradient()
models = client.list_models(only_base=True)
for model in models:
    print(model.id)
```

```output
99148c6d-c2a0-4fbe-a4a7-e7c05bdb8a09_base_ml_model
f0b97d96-51a8-4040-8b22-7940ee1fa24e_base_ml_model
cc2dafce-9e6e-4a23-a918-cad6ba89e42e_base_ml_model
```

```python
new_model = models[-1].create_model_adapter(name="my_model_adapter")
new_model.id, new_model.name
```

```output
('674119b5-f19e-4856-add2-767ae7f7d7ef_model_adapter', 'my_model_adapter')
```

## 创建 Gradient 实例

您可以指定不同的参数，如模型、生成的最大标记数、温度等。

由于我们稍后要对模型进行微调，我们选择具有 ID `674119b5-f19e-4856-add2-767ae7f7d7ef_model_adapter` 的模型适配器，但您可以使用任何基础或可微调模型。

```python
llm = GradientLLM(
    # 在 `$ gradient model list` 中列出的 `ID`
    model="674119b5-f19e-4856-add2-767ae7f7d7ef_model_adapter",
    # # 可选: 设置新凭据，它们默认为环境变量
    # gradient_workspace_id=os.environ["GRADIENT_WORKSPACE_ID"],
    # gradient_access_token=os.environ["GRADIENT_ACCESS_TOKEN"],
    model_kwargs=dict(max_generated_token_count=128),
)
```

## 创建提示模板

我们将为问题和答案创建一个提示模板。

```python
template = """Question: {question}
Answer: """
prompt = PromptTemplate.from_template(template)
```

## 初始化 LLMChain

```python
llm_chain = LLMChain(prompt=prompt, llm=llm)
```

## 运行 LLMChain

提供一个问题并运行 LLMChain。

```python
question = "1994 年超级碗的冠军是哪支 NFL 球队？"
llm_chain.run(question=question)
```

```output
'\n1994 年的超级碗冠军是圣弗朗西斯科 49 人队。'
```

# 通过微调改进结果（可选）

嗯 - 那是错误的 - 圣弗朗西斯科 49 人队并没有获胜。

对于这个问题的正确答案应该是 `达拉斯牛仔队!`。

让我们通过在正确答案上微调来增加正确答案的几率，使用 PromptTemplate。

```python
dataset = [
    {
        "inputs": template.format(question="1994 年超级碗的冠军是哪支 NFL 球队？")
        + " 达拉斯牛仔队!"
    }
]
dataset
```

```output
[{'inputs': 'Question: 1994 年超级碗的冠军是哪支 NFL 球队？\n\nAnswer:  达拉斯牛仔队!'}]
```

```python
new_model.fine_tune(samples=dataset)
```

```output
FineTuneResponse(number_of_trainable_tokens=27, sum_loss=78.17996)
```

```python
# 我们可以保留 llm_chain，因为注册的模型刚刚在 gradient.ai 服务器上刷新。
llm_chain.run(question=question)
```

```output
'达拉斯牛仔队'
```