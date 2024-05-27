# LangSmith LLM 运行

本笔记本演示了如何直接从 LangSmith 的 LLM 运行中加载数据，并在该数据上微调模型。

该过程简单明了，包括 3 个步骤。

1. 选择要训练的 LLM 运行。

2. 使用 LangSmithRunChatLoader 加载运行为聊天会话。

3. 对模型进行微调。

然后，您可以在 LangChain 应用程序中使用经过微调的模型。

在深入研究之前，让我们安装必要的先决条件。

## 先决条件

确保您已安装 langchain >= 0.0.311，并已使用您的 LangSmith API 密钥配置了您的环境。

```python
%pip install --upgrade --quiet  langchain langchain-openai
```

```python
import os
import uuid
uid = uuid.uuid4().hex[:6]
project_name = f"Run Fine-tuning Walkthrough {uid}"
os.environ["LANGCHAIN_TRACING_V2"] = "true"
os.environ["LANGCHAIN_API_KEY"] = "YOUR API KEY"
os.environ["LANGCHAIN_PROJECT"] = project_name
```

## 1. 选择运行

第一步是选择要在其上进行微调的运行。一个常见情况是选择在接收到积极用户反馈的跟踪中的 LLM 运行。您可以在[LangSmith Cookbook](https://github.com/langchain-ai/langsmith-cookbook/blob/main/exploratory-data-analysis/exporting-llm-runs-and-feedback/llm_run_etl.ipynb)和[文档](https://docs.smith.langchain.com/tracing/use-cases/export-runs/local)中找到这方面的示例。

为了本教程的目的，我们将生成一些运行供您在此处使用。让我们尝试对一个简单的函数调用链进行微调。

```python
from enum import Enum
from langchain_core.pydantic_v1 import BaseModel, Field
class Operation(Enum):
    add = "+"
    subtract = "-"
    multiply = "*"
    divide = "/"
class Calculator(BaseModel):
    """一个计算器函数"""
    num1: float
    num2: float
    operation: Operation = Field(..., description="+,-,*,/")
    def calculate(self):
        if self.operation == Operation.add:
            return self.num1 + self.num2
        elif self.operation == Operation.subtract:
            return self.num1 - self.num2
        elif self.operation == Operation.multiply:
            return self.num1 * self.num2
        elif self.operation == Operation.divide:
            if self.num2 != 0:
                return self.num1 / self.num2
            else:
                return "不能除以零"
```

```python
from pprint import pprint
from langchain_core.pydantic_v1 import BaseModel
from langchain_core.utils.function_calling import convert_pydantic_to_openai_function
openai_function_def = convert_pydantic_to_openai_function(Calculator)
pprint(openai_function_def)
```

```output
{'description': '一个计算器函数',
 'name': 'Calculator',
 'parameters': {'description': '一个计算器函数',
                'properties': {'num1': {'title': 'Num1', 'type': 'number'},
                               'num2': {'title': 'Num2', 'type': 'number'},
                               'operation': {'allOf': [{'description': 'An '
                                                                       'enumeration.',
                                                        'enum': ['+',
                                                                 '-',
                                                                 '*',
                                                                 '/'],
                                                        'title': 'Operation'}],
                                             'description': '+,-,*,/'}},
                'required': ['num1', 'num2', 'operation'],
                'title': 'Calculator',
                'type': 'object'}}
```

```python
from langchain_core.output_parsers.openai_functions import PydanticOutputFunctionsParser
from langchain_core.prompts import ChatPromptTemplate
from langchain_openai import ChatOpenAI
prompt = ChatPromptTemplate.from_messages(
    [
        ("system", "You are an accounting assistant."),
        ("user", "{input}"),
    ]
)
chain = (
    prompt
    | ChatOpenAI().bind(functions=[openai_function_def])
    | PydanticOutputFunctionsParser(pydantic_schema=Calculator)
    | (lambda x: x.calculate())
)
```

```python
math_questions = [
    "45 除以 9 等于多少？",
    "81 除以 9 等于多少？",
    "72 除以 8 等于多少？",
    "56 除以 7 等于多少？",
    "36 除以 6 等于多少？",
    "64 除以 8 等于多少？",
    "12 乘以 6 等于多少？",
    "8 乘以 8 等于多少？",
    "10 乘以 10 等于多少？",
    "11 乘以 11 等于多少？",
    "13 乘以 13 等于多少？",
    "45 加上 30 等于多少？",
    "72 加上 28 等于多少？",
    "56 加上 44 等于多少？",
    "63 加上 37 等于多少？",
    "70 减去 35 等于多少？",
    "60 减去 30 等于多少？",
    "50 减去 25 等于多少？",
    "40 减去 20 等于多少？",
    "30 减去 15 等于多少？",
]
results = chain.batch([{"input": q} for q in math_questions], return_exceptions=True)
```

#### 加载未出错的运行

现在我们可以选择要在其上进行微调的成功运行。

```python
from langsmith.client import Client
client = Client()
```

```python
successful_traces = {
    run.trace_id
    for run in client.list_runs(
        project_name=project_name,
        execution_order=1,
        error=False,
    )
}
llm_runs = [
    run
    for run in client.list_runs(
        project_name=project_name,
        run_type="llm",
    )
    if run.trace_id in successful_traces
]
```

## 2. 准备数据

现在我们可以创建一个 LangSmithRunChatLoader 的实例，并使用它的 lazy_load() 方法加载聊天会话。

```python
from langchain_community.chat_loaders.langsmith import LangSmithRunChatLoader
loader = LangSmithRunChatLoader(runs=llm_runs)
chat_sessions = loader.lazy_load()
```

在加载了聊天会话之后，将它们转换成适合进行微调的格式。

```python
from langchain_community.adapters.openai import convert_messages_for_finetuning
training_data = convert_messages_for_finetuning(chat_sessions)
```

## 3. 微调模型

现在，使用 OpenAI 库来启动微调过程。

```python
import json
import time
from io import BytesIO
import openai
my_file = BytesIO()
for dialog in training_data:
    my_file.write((json.dumps({"messages": dialog}) + "\n").encode("utf-8"))
my_file.seek(0)
training_file = openai.files.create(file=my_file, purpose="fine-tune")
job = openai.fine_tuning.jobs.create(
    training_file=training_file.id,
    model="gpt-3.5-turbo",
)
# 等待微调完成（可能需要一些时间）
status = openai.fine_tuning.jobs.retrieve(job.id).status
start_time = time.time()
while status != "succeeded":
    print(f"Status=[{status}]... {time.time() - start_time:.2f}s", end="\r", flush=True)
    time.sleep(5)
    status = openai.fine_tuning.jobs.retrieve(job.id).status
# 现在你的模型已经完成微调！
```

```output
Status=[running]... 349.84s. 17.72s
```

## 4. 在 LangChain 中使用

微调完成后，将得到的模型 ID 与 LangChain 应用中的 ChatOpenAI 模型类一起使用。

```python
# 获取微调后的模型 ID
job = openai.fine_tuning.jobs.retrieve(job.id)
model_id = job.fine_tuned_model
# 在 LangChain 中使用微调后的模型
from langchain_openai import ChatOpenAI
model = ChatOpenAI(
    model=model_id,
    temperature=1,
)
```

```python
(prompt | model).invoke({"input": "What's 56/7?"})
```

```output
AIMessage(content='Let me calculate that for you.')
```

现在，你已成功使用 LangSmith LLM 运行的数据对模型进行了微调！