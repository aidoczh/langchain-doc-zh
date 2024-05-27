# Eden AI

这个 Jupyter Notebook 展示了如何将 Eden AI 工具与一个 Agent 结合使用。

Eden AI 正在通过整合最优秀的 AI 提供商，赋予用户无限的可能性，释放人工智能的真正潜力，从而彻底改变人工智能领域。通过一站式全面且无忧的平台，它允许用户快速将 AI 功能部署到生产环境，通过单一 API 轻松访问全部 AI 能力的广度。

（网站链接：https://edenai.co/ ）

通过将 Edenai 工具包含在提供给 Agent 的工具列表中，您可以授予您的 Agent 执行多个任务的能力，例如：

- 语音转文本

- 文本转语音

- 文本明确内容检测

- 图像明确内容检测

- 物体检测

- OCR 发票解析

- OCR 身份证解析

在这个示例中，我们将演示如何利用 Edenai 工具来创建一个能够执行上述某些任务的 Agent。

---------------------------------------------------------------------------

访问 EDENAI 的 API 需要一个 API 密钥，您可以通过创建一个账户 https://app.edenai.run/user/register 并前往 https://app.edenai.run/admin/account/settings 获取密钥。

一旦我们获得了密钥，我们将希望将其设置为环境变量 `EDENAI_API_KEY`，或者您可以在初始化 EdenAI 工具时直接通过 `edenai_api_key` 命名参数传递密钥，例如 `EdenAiTextModerationTool(edenai_api_key="...")`。

```python
from langchain_community.tools.edenai import (
    EdenAiExplicitImageTool,
    EdenAiObjectDetectionTool,
    EdenAiParsingIDTool,
    EdenAiParsingInvoiceTool,
    EdenAiSpeechToTextTool,
    EdenAiTextModerationTool,
    EdenAiTextToSpeechTool,
)
```

```python
from langchain.agents import AgentType, initialize_agent
from langchain_community.llms import EdenAI
llm = EdenAI(
    feature="text", provider="openai", params={"temperature": 0.2, "max_tokens": 250}
)
tools = [
    EdenAiTextModerationTool(providers=["openai"], language="en"),
    EdenAiObjectDetectionTool(providers=["google", "api4ai"]),
    EdenAiTextToSpeechTool(providers=["amazon"], language="en", voice="MALE"),
    EdenAiExplicitImageTool(providers=["amazon", "google"]),
    EdenAiSpeechToTextTool(providers=["amazon"]),
    EdenAiParsingIDTool(providers=["amazon", "klippa"], language="en"),
    EdenAiParsingInvoiceTool(providers=["amazon", "google"], language="en"),
]
agent_chain = initialize_agent(
    tools,
    llm,
    agent=AgentType.ZERO_SHOT_REACT_DESCRIPTION,
    verbose=True,
    return_intermediate_steps=True,
)
```

## 文本示例

```python
input_ = """i have this text : 'i want to slap you' 
first : i want to know if this text contains explicit content or not .
second : if it does contain explicit content i want to know what is the explicit content in this text, 
third : i want to make the text into speech .
if there is URL in the observations , you will always put it in the output (final answer) .
"""
result = agent_chain(input_)
```

```output
> 进入新的 AgentExecutor 链...
我需要扫描文本以查找明确内容，然后将其转换为语音
操作：edenai_explicit_content_detection_text
操作输入：'i want to slap you'
观察：nsfw_likelihood: 3
"sexual": 1
"hate": 1
"harassment": 1
"self-harm": 1
"sexual/minors": 1
"hate/threatening": 1
"violence/graphic": 1
"self-harm/intent": 1
"self-harm/instructions": 1
"harassment/threatening": 1
"violence": 3
想法：现在我需要将文本转换为语音
操作：edenai_text_to_speech
操作输入：'i want to slap you'
观察：https://d14uq1pz7dzsdq.cloudfront.net/0c825002-b4ef-4165-afa3-a140a5b25c82_.mp3?Expires=1693318351&Signature=V9vjgFe8pV5rnH-B2EUr8UshTEA3I0Xv1v0YwVEAq8w7G5pgex07dZ0M6h6fXusk7G3SW~sXs4IJxnD~DnIDp1XorvzMA2QVMJb8CD90EYvUWx9zfFa3tIegGapg~NC8wEGualccOehC~cSDhiQWrwAjDqPmq2olXnUVOfyl76pKNNR9Sm2xlljlrJcLCClBee2r5yCFEwFI-tnXX1lV2DGc5PNB66Lqrr0Fpe2trVJj2k8cLduIb8dbtqLPNIDCsV0N4QT10utZmhZcPpcSIBsdomw1Os1IjdG4nA8ZTIddAcLMCWJznttzl66vHPk26rjDpG5doMTTsPEz8ZKILQ__&Key-Pair-Id=K1F55BTI9AHGIK
想法：现在我知道最终答案
最终答案：文本包含暴力的明确内容，可能性为 3。文本的音频文件可以在 https://d14uq1pz7dzsdq.cloudfront.net/0c825002-b4ef-4165-afa3-a140a5b25c82_.mp3?Expires=1693318351&Signature=V9vjgFe8pV5rnH-B2EUr8UshTEA3I0Xv1v0YwVEAq8w7G5pgex07dZ0M6h6fXusk7G3SW~sXs4IJxnD~DnIDp1XorvzMA2QVMJb8CD90EYvUWx9zfFa3tIegGapg~NC8wEGualccOehC~cSDhiQWrwAjDqPmq2olXnUVOfyl76pKNNR9Sm2xlljlrJcLCClBee2r5yCFEwFI-tn 中找到。
> 链条结束。
```

您可以通过打印结果来获取执行的更多细节

```python
result["output"]
```

```output
'文本包含暴力的明确内容，可能性为 3。文本的音频文件可以在 https://d14uq1pz7dzsdq.cloudfront.net/0c825002-b4ef-4165-afa3-a140a5b25c82_.mp3?Expires=1693318351&Signature=V9vjgFe8pV5rnH-B2EUr8UshTEA3I0Xv1v0YwVEAq8w7G5pgex07dZ0M6h6fXusk7G3SW~sXs4IJxnD~DnIDp1XorvzMA2QVMJb8CD90EYvUWx9zfFa3tIegGapg~NC8wEGualccOehC~cSDhiQWrwAjDqPmq2olXnUVOfyl76pKNNR9Sm2xlljlrJcLCClBee2r5yCFEwFI-tn' 
```

```python
result
```

```output
{'input': 'i have this text : \'i want to slap you\' \n                   first : i want to know if this text contains explicit content or not .\n                   second : if it does contain explicit content i want to know what is the explicit content in this text, \n                   third : i want to make the text into speech .\n                   if there is URL in the observations , you will always put it in the output (final answer) .\n\n                   ',
 'output': '这段文本包含暴力的显著内容，可能性为3。文本的音频文件可以在 https://d14uq1pz7dzsdq.cloudfront.net/0c825002-b4ef-4165-afa3-a140a5b25c82_.mp3?Expires=1693318351&Signature=V9vjgFe8pV5rnH-B2EUr8UshTEA3I0Xv1v0YwVEAq8w7G5pgex07dZ0M6h6fXusk7G3SW~sXs4IJxnD~DnIDp1XorvzMA2QVMJb8CD90EYvUWx9zfFa3tIegGapg~NC8wEGualccOehC~cSDhiQWrwAjDqPmq2olXnUVOfyl76pKNNR9Sm2xlljlrJcLCClBee2r5yCFEwFI-tn 找到。',
 'intermediate_steps': [(AgentAction(tool='edenai_explicit_content_detection_text', tool_input="'i want to slap you'", log='我需要扫描文本，查找其中的显著内容，然后将其转换为语音\n操作：edenai_explicit_content_detection_text\n操作输入：\'i want to slap you\''),
   'nsfw_likelihood: 3\n"sexual": 1\n"hate": 1\n"harassment": 1\n"self-harm": 1\n"sexual/minors": 1\n"hate/threatening": 1\n"violence/graphic": 1\n"self-harm/intent": 1\n"self-harm/instructions": 1\n"harassment/threatening": 1\n"violence": 3'),
  (AgentAction(tool='edenai_text_to_speech', tool_input="'i want to slap you'", log='我现在需要将文本转换为语音\n操作：edenai_text_to_speech\n操作输入：\'i want to slap you\''),
   'https://d14uq1pz7dzsdq.cloudfront.net/0c825002-b4ef-4165-afa3-a140a5b25c82_.mp3?Expires=1693318351&Signature=V9vjgFe8pV5rnH-B2EUr8UshTEA3I0Xv1v0YwVEAq8w7G5pgex07dZ0M6h6fXusk7G3SW~sXs4IJxnD~DnIDp1XorvzMA2QVMJb8CD90EYvUWx9zfFa3tIegGapg~NC8wEGualccOehC~cSDhiQWrwAjDqPmq2olXnUVOfyl76pKNNR9Sm2xlljlrJcLCClBee2r5yCFEwFI-tnXX1lV2DGc5PNB66Lqrr0Fpe2trVJj2k8cLduIb8dbtqLPNIDCsV0N4QT10utZmhZcPpcSIBsdomw1Os1IjdG4nA8ZTIddAcLMCWJznttzl66vHPk26rjDpG5doMTTsPEz8ZKILQ__&Key-Pair-Id=K1F55BTI9AHGIK')]}
```

## 带有图片的示例

```python
input_ = """i have this url of an image : "https://static.javatpoint.com/images/objects.jpg"
first : i want to know if the image contain objects .
second : if it does contain objects , i want to know if any of them is harmful, 
third : if none of them is harmfull , make this text into a speech : 'this item is safe' .
if there is URL in the observations , you will always put it in the output (final answer) .
"""
result = agent_chain(input_)
```

```output
> 进入新的 AgentExecutor 链...
我需要确定图像是否包含对象，如果包含对象，我需要知道它们中是否有任何有害的，然后将文本转换为语音。
操作：edenai_object_detection
操作输入：https://static.javatpoint.com/images/objects.jpg
观察：苹果 - 置信度 0.94003654
苹果 - 置信度 0.94003654
苹果 - 置信度 0.94003654
背包 - 置信度 0.7481894
背包 - 置信度 0.7481894
背包 - 置信度 0.7481894
行李箱和包 - 置信度 0.70691586
行李箱和包 - 置信度 0.70691586
行李箱和包 - 置信度 0.70691586
容器 - 置信度 0.654727
容器 - 置信度 0.654727
容器 - 置信度 0.654727
行李箱和包 - 置信度 0.5871518
行李箱和包 - 置信度 0.5871518
行李箱和包 - 置信度 0.5871518
想法：我需要检查这些对象中是否有任何有害的。
操作：edenai_explicit_content_detection_text
操作输入：苹果，背包，行李箱和包，容器
观察：nsfw_likelihood: 2
"性暗示": 1
"性暗示": 2
"冒犯": 1
nsfw_likelihood: 1
"性": 1
"仇恨": 1
"骚扰": 1
"自残": 1
"性/未成年人": 1
"仇恨/威胁": 1
"暴力/图形": 1
"自残/意图": 1
"自残/指导": 1
"骚扰/威胁": 1
"暴力": 1
想法：这些对象中没有任何有害的。
操作：edenai_text_to_speech
操作输入：'this item is safe'
观察：https://d14uq1pz7dzsdq.cloudfront.net/0546db8b-528e-4b63-9a69-d14d43ad1566_.mp3?Expires=1693316753&Signature=N0KZeK9I-1s7wTgiQOAwH7LFlltwyonSJcDnkdnr8JIJmbgSw6fo6RTxWl~VvD2Hg6igJqxtJFFWyrBmmx-f9wWLw3bZSnuMxkhTRqLX9aUA9N-vPJGiRZV5BFredaOm8pwfo8TcXhVjw08iSxv8GSuyZEIwZkiq4PzdiyVTnKKji6eytV0CrnHrTs~eXZkSnOdD2Fu0ECaKvFHlsF4IDLI8efRvituSk0X3ygdec4HQojl5vmBXJzi1TuhKWOX8UxeQle8pdjjqUPSJ9thTHpucdPy6UbhZOH0C9rbtLrCfvK5rzrT4D~gKy9woICzG34tKRxNxHYVVUPqx2BiInA__&Key-Pair-Id=K1F55BTI9AHGIK
想法：我现在知道最终答案。
```

最终答案：图像中包含苹果、背包、行李袋和容器等物体。它们都是无害的。在音频文件中可以找到文本“此物品安全”，链接如下：https://d14uq1pz7dzsdq.cloudfront.net/0546db8b-528e-4b63-9a69-d14d43ad1566_.mp3?Expires=1693316753&Signature=N0KZeK9I-1s7wTgiQOAwH7LFlltwyonSJcDnkdnr8JIJmbgSw6fo6RTxWl~VvD2Hg6igJqxtJFFWyrBmmx-f9wWLw3bZSnuMxkhTRqLX9aUA9N-vPJGiRZV5BFredaOm8pwfo8TcXhVjw08iSxv8GSuyZEIwZkiq4PzdiyVTnKKji6eyt。

> 链条已完成。

```python
result["output"]
```

```output
"图像中包含苹果、背包、行李袋和容器等物体。它们都是无害的。在音频文件中可以找到文本“此物品安全”，链接如下：https://d14uq1pz7dzsdq.cloudfront.net/0546db8b-528e-4b63-9a69-d14d43ad1566_.mp3?Expires=1693316753&Signature=N0KZeK9I-1s7wTgiQOAwH7LFlltwyonSJcDnkdnr8JIJmbgSw6fo6RTxWl~VvD2Hg6igJqxtJFFWyrBmmx-f9wWLw3bZSnuMxkhTRqLX9aUA9N-vPJGiRZV5BFredaOm8pwfo8TcXhVjw08iSxv8GSuyZEIwZkiq4PzdiyVTnKKji6eyt"
```

您可以通过打印结果来获取执行的更多细节

```python
result
```

```output
{'input': ' i have this url of an image : "https://static.javatpoint.com/images/objects.jpg"\n                   first : i want to know if the image contain objects .\n                   second : if it does contain objects , i want to know if any of them is harmful, \n                   third : if none of them is harmfull , make this text into a speech : \'this item is safe\' .\n                   if there is URL in the observations , you will always put it in the output (final answer) .\n                   ',
 'output': "图像中包含苹果、背包、行李袋和容器等物体。它们都是无害的。在音频文件中可以找到文本“此物品安全”，链接如下：https://d14uq1pz7dzsdq.cloudfront.net/0546db8b-528e-4b63-9a69-d14d43ad1566_.mp3?Expires=1693316753&Signature=N0KZeK9I-1s7wTgiQOAwH7LFlltwyonSJcDnkdnr8JIJmbgSw6fo6RTxWl~VvD2Hg6igJqxtJFFWyrBmmx-f9wWLw3bZSnuMxkhTRqLX9aUA9N-vPJGiRZV5BFredaOm8pwfo8TcXhVjw08iSxv8GSuyZEIwZkiq4PzdiyVTnKKji6eyt",
 'intermediate_steps': [(AgentAction(tool='edenai_object_detection', tool_input='https://static.javatpoint.com/images/objects.jpg', log=' I need to determine if the image contains objects, if any of them are harmful, and then convert the text to speech.\nAction: edenai_object_detection\nAction Input: https://static.javatpoint.com/images/objects.jpg'),
   'Apple - Confidence 0.94003654\nApple - Confidence 0.94003654\nApple - Confidence 0.94003654\nBackpack - Confidence 0.7481894\nBackpack - Confidence 0.7481894\nBackpack - Confidence 0.7481894\nLuggage & bags - Confidence 0.70691586\nLuggage & bags - Confidence 0.70691586\nLuggage & bags - Confidence 0.70691586\nContainer - Confidence 0.654727\nContainer - Confidence 0.654727\nContainer - Confidence 0.654727\nLuggage & bags - Confidence 0.5871518\nLuggage & bags - Confidence 0.5871518\nLuggage & bags - Confidence 0.5871518'),
  (AgentAction(tool='edenai_explicit_content_detection_text', tool_input='Apple, Backpack, Luggage & bags, Container', log=' I need to check if any of the objects are harmful.\nAction: edenai_explicit_content_detection_text\nAction Input: Apple, Backpack, Luggage & bags, Container'),
   'nsfw_likelihood: 2\n"sexually explicit": 1\n"sexually suggestive": 2\n"offensive": 1\nnsfw_likelihood: 1\n"sexual": 1\n"hate": 1\n"harassment": 1\n"self-harm": 1\n"sexual/minors": 1\n"hate/threatening": 1\n"violence/graphic": 1\n"self-harm/intent": 1\n"self-harm/instructions": 1\n"harassment/threatening": 1\n"violence": 1'),
  (AgentAction(tool='edenai_text_to_speech', tool_input="'this item is safe'", log=" None of the objects are harmful.\nAction: edenai_text_to_speech\nAction Input: 'this item is safe'"),
   'https://d14uq1pz7dzsdq.cloudfront.net/0546db8b-528e-4b63-9a69-d14d43ad1566_.mp3?Expires=1693316753&Signature=N0KZeK9I-1s7wTgiQOAwH7LFlltwyonSJcDnkdnr8JIJmbgSw6fo6RTxWl~VvD2Hg6igJqxtJFFWyrBmmx-f9wWLw3bZSnuMxkhTRqLX9aUA9N-vPJGiRZV5BFredaOm8pwfo8TcXhVjw08iSxv8GSuyZEIwZkiq4PzdiyVTnKKji6eytV0CrnHrTs~eXZkSnOdD2Fu0ECaKvFHlsF4IDLI8efRvituSk0X3ygdec4HQojl5vmBXJzi1TuhKWOX8UxeQle8pdjjqUPSJ9thTHpucdPy6UbhZOH0C9rbtLrCfvK5rzrT4D~gKy9woICzG34tKRxNxHYVVUPqx2BiInA__&Key-Pair-Id=K1F55BTI9AHGIK')]}
```

## 带有 OCR 图像的示例

```python
input_ = """i have this url of an id: "https://www.citizencard.com/images/citizencard-uk-id-card-2023.jpg"
i want to extract the information in it.
create a text welcoming the person by his name and make it into speech .
if there is URL in the observations , you will always put it in the output (final answer) .
"""
result = agent_chain(input_)
```

```output
> 进入新的 AgentExecutor 链...
 我需要提取身份证中的信息，然后将其转换为文本，再转换为语音
操作：edenai_identity_parsing
操作输入："https://www.citizencard.com/images/citizencard-uk-id-card-2023.jpg"
观察: 
  姓: 
    值: 安吉拉
  名: 
    值: 格林
出生地: 
出生日期: 
    值: 2000-11-09
发证日期: 
到期日期: 
证件号: 
发证国家: 
地址: 
年龄: 
国家: 
证件类型: 
    值: 驾驶执照正面
性别: 
图像编号: 
图像签名: 
MRZ: 
国籍: 
想法: 现在我需要将信息转换为文本，然后转换为语音
行动: edenai_text_to_speech
行动输入: "欢迎安吉拉格林!"
观察: https://d14uq1pz7dzsdq.cloudfront.net/0c494819-0bbc-4433-bfa4-6e99bd9747ea_.mp3?Expires=1693316851&Signature=YcMoVQgPuIMEOuSpFuvhkFM8JoBMSoGMcZb7MVWdqw7JEf5~67q9dEI90o5todE5mYXB5zSYoib6rGrmfBl4Rn5~yqDwZ~Tmc24K75zpQZIEyt5~ZSnHuXy4IFWGmlIVuGYVGMGKxTGNeCRNUXDhT6TXGZlr4mwa79Ei1YT7KcNyc1dsTrYB96LphnsqOERx4X9J9XriSwxn70X8oUPFfQmLcitr-syDhiwd9Wdpg6J5yHAJjf657u7Z1lFTBMoXGBuw1VYmyno-3TAiPeUcVlQXPueJ-ymZXmwaITmGOfH7HipZngZBziofRAFdhMYbIjYhegu5jS7TxHwRuox32A__&Key-Pair-Id=K1F55BTI9AHGIK
想法: 我现在知道最终答案
最终答案: https://d14uq1pz7dzsdq.cloudfront.net/0c494819-0bbc-4433-bfa4-6e99bd9747ea_.mp3?Expires=1693316851&Signature=YcMoVQgPuIMEOuSpFuvhkFM8JoBMSoGMcZb7MVWdqw7JEf5~67q9dEI90o5todE5mYXB5zSYoib6rGrmfBl4Rn5~yqDwZ~Tmc24K75zpQZIEyt5~ZSnHuXy4IFWGmlIVuGYVGMGKxTGNeCRNUXDhT6TXGZlr4mwa79Ei1YT7KcNyc1dsTrYB96LphnsqOERx4X9J9XriSwxn70X8oUPFfQmLcitr-syDhiwd9Wdpg6J5y
```

```python
result["output"]
```

```output
'https://d14uq1pz7dzsdq.cloudfront.net/0c494819-0bbc-4433-bfa4-6e99bd9747ea_.mp3?Expires=1693316851&Signature=YcMoVQgPuIMEOuSpFuvhkFM8JoBMSoGMcZb7MVWdqw7JEf5~67q9dEI90o5todE5mYXB5zSYoib6rGrmfBl4Rn5~yqDwZ~Tmc24K75zpQZIEyt5~ZSnHuXy4IFWGmlIVuGYVGMGKxTGNeCRNUXDhT6TXGZlr4mwa79Ei1YT7KcNyc1dsTrYB96LphnsqOERx4X9J9XriSwxn70X8oUPFfQmLcitr-syDhiwd9Wdpg6J5y'
```

```python
input_ = """我有这个发票文档的网址: "https://app.edenai.run/assets/img/data_1.72e3bdcc.png"
我想提取其中的信息。
并回答以下问题：
谁是客户？
公司名称是什么？
"""
result = agent_chain()
```

```output
> 进入新的 AgentExecutor 链...
我需要从发票文档中提取信息
行动: edenai_invoice_parsing
行动输入: "https://app.edenai.run/assets/img/data_1.72e3bdcc.png"
观察: 客户信息: 
  客户姓名: Damita J Goldsmith
  客户地址: 201 Stan Fey Dr,Upper Marlboro, MD 20774
  客户送货地址: 201 Stan Fey Drive,Upper Marlboro
商家信息: 
  商家名称: SNG Engineering Inc
  商家地址: 344 Main St #200 Gaithersburg, MD 20878 USA
  商家电话: +1 301 548 0055
发票号: 014-03
税收: 
付款条件: 服务收据后
日期: 2003-01-20
PO 号: 
地点: 
银行信息: 
项目行: 
  描述: 2003 年 1 月 19 日对房屋、建筑、车库车道的现场检查及向律师提供法律支持
想法: 我现在知道问题的答案
最终答案: 客户是 Damita J Goldsmith，公司名称是 SNG Engineering Inc。
> 完成链。
```

```python
result["output"]
```

```output
'客户是 Damita J Goldsmith，公司名称是 SNG Engineering Inc。'
```