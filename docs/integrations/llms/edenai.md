# Eden AI

Eden AI正在通过整合最佳的人工智能提供商，赋予用户解锁无限可能性的能力，发挥人工智能的真正潜力，从而彻底改变人工智能领域。通过一体化全面且无忧的平台，用户可以快速将人工智能功能部署到生产环境中，通过单个API轻松访问全部人工智能功能的广泛能力。 (网站: [https://edenai.co/](https://edenai.co/))

以下示例演示了如何使用LangChain与Eden AI模型进行交互

-----------------------------------------------------------------------------------

要访问EDENAI的API，需要一个API密钥，您可以通过创建一个账户 [https://app.edenai.run/user/register](https://app.edenai.run/user/register) 并前往 [https://app.edenai.run/admin/account/settings](https://app.edenai.run/admin/account/settings) 获取。

获取密钥后，我们需要将其设置为环境变量，运行以下命令：

```bash
export EDENAI_API_KEY="..."
```

如果您不想设置环境变量，可以直接通过edenai_api_key命名参数传递密钥，在初始化EdenAI LLM类时：

```python
from langchain_community.llms import EdenAI
```

```python
llm = EdenAI(edenai_api_key="...", provider="openai", temperature=0.2, max_tokens=250)
```

## 调用模型

EdenAI API汇集了各种提供商，每个提供商都提供多个模型。

要访问特定模型，只需在实例化期间添加'model'。

例如，让我们探索OpenAI提供的模型，如GPT3.5

### 文本生成

```python
from langchain.chains import LLMChain
from langchain_core.prompts import PromptTemplate
llm = EdenAI(
    feature="text",
    provider="openai",
    model="gpt-3.5-turbo-instruct",
    temperature=0.2,
    max_tokens=250,
)
prompt = """
User: Answer the following yes/no question by reasoning step by step. Can a dog drive a car?
Assistant:
"""
llm(prompt)
```

### 图像生成

```python
import base64
from io import BytesIO
from PIL import Image
def print_base64_image(base64_string):
    # 将base64字符串解码为二进制数据
    decoded_data = base64.b64decode(base64_string)
    # 创建内存流以读取二进制数据
    image_stream = BytesIO(decoded_data)
    # 使用PIL打开图像
    image = Image.open(image_stream)
    # 显示图像
    image.show()
```

```python
text2image = EdenAI(feature="image", provider="openai", resolution="512x512")
```

```python
image_output = text2image("A cat riding a motorcycle by Picasso")
```

```python
print_base64_image(image_output)
```

### 带回调的文本生成

```python
from langchain_community.llms import EdenAI
from langchain_core.callbacks import StreamingStdOutCallbackHandler
llm = EdenAI(
    callbacks=[StreamingStdOutCallbackHandler()],
    feature="text",
    provider="openai",
    temperature=0.2,
    max_tokens=250,
)
prompt = """
User: Answer the following yes/no question by reasoning step by step. Can a dog drive a car?
Assistant:
"""
print(llm.invoke(prompt))
```

## 链式调用

```python
from langchain.chains import LLMChain, SimpleSequentialChain
from langchain_core.prompts import PromptTemplate
```

```python
llm = EdenAI(feature="text", provider="openai", temperature=0.2, max_tokens=250)
text2image = EdenAI(feature="image", provider="openai", resolution="512x512")
```

```python
prompt = PromptTemplate(
    input_variables=["product"],
    template="What is a good name for a company that makes {product}?",
)
chain = LLMChain(llm=llm, prompt=prompt)
```

```python
second_prompt = PromptTemplate(
    input_variables=["company_name"],
    template="Write a description of a logo for this company: {company_name}, the logo should not contain text at all ",
)
chain_two = LLMChain(llm=llm, prompt=second_prompt)
```

```python
third_prompt = PromptTemplate(
    input_variables=["company_logo_description"],
    template="{company_logo_description}",
)
chain_three = LLMChain(llm=text2image, prompt=third_prompt)
```

```python
# 运行链，仅指定第一个链的输入变量。
overall_chain = SimpleSequentialChain(
    chains=[chain, chain_two, chain_three], verbose=True
)
output = overall_chain.run("hats")
```

```python
# 打印图像
print_base64_image(output)
```