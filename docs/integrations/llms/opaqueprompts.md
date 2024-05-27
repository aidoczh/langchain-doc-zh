# OpaquePrompts
[OpaquePrompts](https://opaqueprompts.readthedocs.io/en/latest/) 是一个服务，使应用程序能够利用语言模型的强大功能，同时不会损害用户的隐私。OpaquePrompts旨在实现可组合性，并且易于集成到现有的应用程序和服务中，用户可以通过一个简单的Python库或LangChain来使用OpaquePrompts。或许更重要的是，OpaquePrompts利用[机密计算](https://en.wikipedia.org/wiki/Confidential_computing)的能力，确保即使是OpaquePrompts服务本身也无法访问它所保护的数据。
这篇笔记将介绍如何使用LangChain与`OpaquePrompts`进行交互。
```python
# 安装 opaqueprompts 和 langchain 包
%pip install --upgrade --quiet  opaqueprompts langchain
```
访问 OpaquePrompts API 需要一个 API 密钥，您可以通过在[OpaquePrompts网站](https://opaqueprompts.opaque.co/)上创建帐户来获取。一旦您拥有帐户，您可以在[API密钥页面](https:opaqueprompts.opaque.co/api-keys)上找到您的API密钥。
```python
import os
# 设置 API 密钥
os.environ["OPAQUEPROMPTS_API_KEY"] = "<OPAQUEPROMPTS_API_KEY>"
os.environ["OPENAI_API_KEY"] = "<OPENAI_API_KEY>"
```
# 使用 OpaquePrompts LLM Wrapper
将 OpaquePrompts 应用于您的应用程序可能就像用 OpaquePrompts 类包装您的LLM一样简单，只需将`llm=OpenAI()`替换为`llm=OpaquePrompts(base_llm=OpenAI())`。
```python
from langchain.chains import LLMChain
from langchain.globals import set_debug, set_verbose
from langchain.memory import ConversationBufferWindowMemory
from langchain_community.llms import OpaquePrompts
from langchain_core.callbacks import StdOutCallbackHandler
from langchain_core.prompts import PromptTemplate
from langchain_openai import OpenAI
set_debug(True)
set_verbose(True)
prompt_template = """
作为人工智能助手，您将根据给定的上下文回答问题。
问题中的敏感个人信息已被掩盖以保护隐私。
例如，如果原始文本中说“Giana is good”，它将被更改为“PERSON_998 is good”。
以下是处理这些更改的方法：
* 将这些掩盖的短语视为占位符，但在回答时仍以相关方式引用它们。
* 可能不同的掩盖术语可能表示相同的含义。
坚持使用给定的术语，不要修改它。
* 所有掩盖的术语遵循“TYPE_ID”模式。
* 请不要发明新的掩盖术语。例如，如果您看到“PERSON_998”，不要想出“PERSON_997”或“PERSON_999”，除非它们已经在问题中。
对话历史：```{history}```
上下文：```在我们2023年2月23日上午10:30的最近会议中，John Doe向我提供了他的个人详细信息。他的电子邮件是johndoe@example.com，联系电话是650-456-7890。他住在美国纽约市，属于美国国籍，信仰基督教，倾向于民主党。他提到最近使用他的信用卡4111 1111 1111 1111进行了交易，并将比特币转账到钱包地址1A1zP1eP5QGefi2DMPTfTL5SLmv7DivfNa。在讨论他的欧洲之行时，他记下了他的IBAN为GB29 NWBK 6016 1331 9268 19。此外，他提供了他的网站https://johndoeportfolio.com。John还讨论了一些他的美国特定细节。他说他的银行账号是1234567890123456，驾驶执照号码是Y12345678。他的ITIN是987-65-4321，最近他更新了他的护照，护照号码是123456789。他强调不要分享他的社会安全号码，即669-45-6789。此外，他提到他通过IP 192.168.1.1远程访问工作文件，并拥有医疗执照号码MED-123456。```
问题：```{question}```
"""
chain = LLMChain(
    prompt=PromptTemplate.from_template(prompt_template),
    llm=OpaquePrompts(base_llm=OpenAI()),
    memory=ConversationBufferWindowMemory(k=2),
    verbose=True,
)
print(
    chain.run(
        {
            "question": """给 John 写一条消息，提醒他为他的网站进行密码重置以保持安全。"""
        },
        callbacks=[StdOutCallbackHandler()],
    )
)
```
从输出中，您可以看到用户输入的上下文中包含敏感数据。
``` 
# 用户输入的上下文
在我们2023年2月23日上午10:30的最近会议中，John Doe向我提供了他的个人详细信息。他的电子邮件是johndoe@example.com，联系电话是650-456-7890。他住在美国纽约市，属于美国国籍，信仰基督教，倾向于民主党。他提到最近使用他的信用卡4111 1111 1111 1111进行了交易，并将比特币转账到钱包地址1A1zP1eP5QGefi2DMPTfTL5SLmv7DivfNa。在讨论他的欧洲之行时，他记下了他的IBAN为GB29 NWBK 6016 1331 9268 19。此外，他提供了他的网站https://johndoeportfolio.com。John还讨论了一些他的美国特定细节。他说他的银行账号是1234567890123456，驾驶执照号码是Y12345678。他的ITIN是987-65-4321，最近他更新了他的护照，护照号码是123456789。他强调不要分享他的社会安全号码，即669-45-6789。此外，他提到他通过IP 192.168.1.1远程访问工作文件，并拥有医疗执照号码MED-123456。```
```
OpaquePrompts 会自动检测敏感数据并用占位符替换。
```
# OpaquePrompts 处理后的上下文
在 DATE_TIME_2 的 DATE_TIME_3，PERSON_3 向我提供了他的个人详细信息。他的电子邮件是 EMAIL_ADDRESS_1，联系电话是 PHONE_NUMBER_1。他住在 LOCATION_2 的 LOCATION_3，属于 NRP_3 国籍，持有 NRP_2 信仰，并倾向于民主党。他提到最近使用他的信用卡 CREDIT_CARD_1 进行了交易，并向加密货币地址 CRYPTO_1 转账比特币。在讨论他的 NRP_1 旅行时，他记下了他的 IBAN 为 IBAN_CODE_1。此外，他提供了他的网站链接为 URL_1。PERSON_2 也讨论了一些关于他的 LOCATION_1 的具体细节。他说他的银行账号是 US_BANK_NUMBER_1，驾驶执照是 US_DRIVER_LICENSE_2。他的 ITIN 是 US_ITIN_1，最近他更新了护照，编号为 DATE_TIME_1。他强调不要分享他的 SSN，即 US_SSN_1。此外，他提到他通过 IP IP_ADDRESS_1 远程访问工作文件，并拥有医疗执照号码 MED-US_DRIVER_LICENSE_1。
```
在 LLM 回复中使用了占位符。
```
# OpaquePrompts 反馈的 LLM 回复
嘿 John，只是想提醒你通过你的电子邮件 johndoe@example.com 为你的网站 https://johndoeportfolio.com 进行密码重置。保持在线安全很重要，所以不要忘记这么做！
```
通过替换占位符来去除敏感数据的 LLM 反馈。
```
# 从 OpaquePrompts 反馈的去敏感化 LLM 回复
嘿 John，只是想提醒你通过你的电子邮件 johndoe@example.com 为你的网站 https://johndoeportfolio.com 进行密码重置。保持在线安全很重要，所以不要忘记这么做！
```
# 在 LangChain 表达式中使用 OpaquePrompts
如果直接替换无法提供所需的灵活性，LangChain 表达式也可以与函数一起使用。
```python
import langchain_community.utilities.opaqueprompts as op
from langchain_core.output_parsers import StrOutputParser
from langchain_core.runnables import RunnablePassthrough
prompt = (PromptTemplate.from_template(prompt_template),)
llm = OpenAI()
pg_chain = (
    op.sanitize
    | RunnablePassthrough.assign(
        response=(lambda x: x["sanitized_input"]) | prompt | llm | StrOutputParser(),
    )
    | (lambda x: op.desanitize(x["response"], x["secure_context"]))
)
pg_chain.invoke(
    {
        "question": "Write a text message to remind John to do password reset for his website through his email to stay secure.",
        "history": "",
    }
)
```