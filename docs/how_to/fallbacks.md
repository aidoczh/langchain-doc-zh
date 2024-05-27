---

keywords: [LCEL, fallbacks]

---

# 如何为可运行程序添加回退机制

在使用语言模型时，您可能经常会遇到来自底层 API 的问题，无论是速率限制还是停机时间。因此，当您将您的 LLM 应用程序投入生产时，更加重要的是要防范这些问题。这就是为什么我们引入了回退机制的概念。

**回退** 是在紧急情况下可能使用的替代方案。

重要的是，回退不仅可以应用在 LLM 层面上，还可以应用在整个可运行程序的层面上。这很重要，因为通常不同的模型需要不同的提示。因此，如果您对 OpenAI 的调用失败了，您不希望只是将相同的提示发送给 Anthropic - 您可能希望在那里使用不同的提示模板并发送不同的版本。

## LLM API 错误的回退

这可能是回退的最常见用例。对 LLM API 的请求可能因各种原因而失败 - API 可能宕机，您可能已达到速率限制，任何数量的原因。因此，使用回退可以帮助防范这些类型的问题。

重要提示: 默认情况下，许多 LLM 封装器会捕获错误并重试。在使用回退时，您很可能希望将其关闭。否则，第一个封装器将继续重试而不会失败。

```python
%pip install --upgrade --quiet  langchain langchain-openai
```

```python
from langchain_anthropic import ChatAnthropic
from langchain_openai import ChatOpenAI
```

首先，让我们模拟一下如果我们从 OpenAI 遇到 RateLimitError 会发生什么

```python
from unittest.mock import patch
import httpx
from openai import RateLimitError
request = httpx.Request("GET", "/")
response = httpx.Response(200, request=request)
error = RateLimitError("rate limit", response=response, body="")
```

```python
# 请注意，我们将 max_retries = 0 设置为避免在速率限制等情况下重试
openai_llm = ChatOpenAI(model="gpt-3.5-turbo-0125", max_retries=0)
anthropic_llm = ChatAnthropic(model="claude-3-haiku-20240307")
llm = openai_llm.with_fallbacks([anthropic_llm])
```

```python
# 让我们首先只使用 OpenAI LLM，以展示我们遇到错误
with patch("openai.resources.chat.completions.Completions.create", side_effect=error):
    try:
        print(openai_llm.invoke("Why did the chicken cross the road?"))
    except RateLimitError:
        print("遇到错误")
```

```output
遇到错误
```

```python
# 现在让我们尝试使用 Anthropic 的回退
with patch("openai.resources.chat.completions.Completions.create", side_effect=error):
    try:
        print(llm.invoke("Why did the chicken cross the road?"))
    except RateLimitError:
        print("遇到错误")
```

```output
content=' I don\'t actually know why the chicken crossed the road, but here are some possible humorous answers:\n\n- To get to the other side!\n\n- It was too chicken to just stand there. \n\n- It wanted a change of scenery.\n\n- It wanted to show the possum it could be done.\n\n- It was on its way to a poultry farmers\' convention.\n\nThe joke plays on the double meaning of "the other side" - literally crossing the road to the other side, or the "other side" meaning the afterlife. So it\'s an anti-joke, with a silly or unexpected pun as the answer.' additional_kwargs={} example=False
```

我们可以像使用普通 LLM 一样使用我们的“带回退的 LLM”。

```python
from langchain_core.prompts import ChatPromptTemplate
prompt = ChatPromptTemplate.from_messages(
    [
        (
            "system",
            "You're a nice assistant who always includes a compliment in your response",
        ),
        ("human", "Why did the {animal} cross the road"),
    ]
)
chain = prompt | llm
with patch("openai.resources.chat.completions.Completions.create", side_effect=error):
    try:
        print(chain.invoke({"animal": "kangaroo"}))
    except RateLimitError:
        print("遇到错误")
```

```output
content=" I don't actually know why the kangaroo crossed the road, but I can take a guess! Here are some possible reasons:\n\n- To get to the other side (the classic joke answer!)\n\n- It was trying to find some food or water \n\n- It was trying to find a mate during mating season\n\n- It was fleeing from a predator or perceived threat\n\n- It was disoriented and crossed accidentally \n\n- It was following a herd of other kangaroos who were crossing\n\n- It wanted a change of scenery or environment \n\n- It was trying to reach a new habitat or territory\n\nThe real reason is unknown without more context, but hopefully one of those potential explanations does the joke justice! Let me know if you have any other animal jokes I can try to decipher." additional_kwargs={} example=False
```

## 序列的回退

---

我们还可以为序列创建回退（fallback），这些回退本身也是序列。在这里，我们使用了两个不同的模型：ChatOpenAI，然后是普通的OpenAI（不使用聊天模型）。因为OpenAI不是聊天模型，所以您可能需要一个不同的提示。

```python
# 首先让我们创建一个带有ChatModel的链条
# 在这里我们添加了一个字符串输出解析器，以使两者的输出类型相同
from langchain_core.output_parsers import StrOutputParser
chat_prompt = ChatPromptTemplate.from_messages(
    [
        (
            "system",
            "You're a nice assistant who always includes a compliment in your response",
        ),
        ("human", "Why did the {animal} cross the road"),
    ]
)
# 在这里，我们将使用一个错误的模型名称来轻松创建一个会出错的链条
chat_model = ChatOpenAI(model="gpt-fake")
bad_chain = chat_prompt | chat_model | StrOutputParser()
```

```python
# 现在让我们创建一个使用普通OpenAI模型的链条
from langchain_core.prompts import PromptTemplate
from langchain_openai import OpenAI
prompt_template = """Instructions: You should always include a compliment in your response.
Question: Why did the {animal} cross the road?"""
prompt = PromptTemplate.from_template(prompt_template)
llm = OpenAI()
good_chain = prompt | llm
```

```python
# 现在我们可以创建一个最终的链条，将两者结合起来
chain = bad_chain.with_fallbacks([good_chain])
chain.invoke({"animal": "turtle"})
```

```output
'\n\nAnswer: The turtle crossed the road to get to the other side, and I have to say he had some impressive determination.'
```

## 长输入的回退

LLM的一个重要限制因素是其上下文窗口的大小。通常，在将提示发送给LLM之前，您可以计算和跟踪提示的长度，但在某些情况下，这可能很困难/复杂，您可以回退到具有更长上下文长度的模型。

```python
short_llm = ChatOpenAI()
long_llm = ChatOpenAI(model="gpt-3.5-turbo-16k")
llm = short_llm.with_fallbacks([long_llm])
```

```python
inputs = "What is the next number: " + ", ".join(["one", "two"] * 3000)
```

```python
try:
    print(short_llm.invoke(inputs))
except Exception as e:
    print(e)
```

```output
This model's maximum context length is 4097 tokens. However, your messages resulted in 12012 tokens. Please reduce the length of the messages.
```

```python
try:
    print(llm.invoke(inputs))
except Exception as e:
    print(e)
```

```output
content='The next number in the sequence is two.' additional_kwargs={} example=False
```

## 回退到更好的模型

通常我们要求模型以特定的格式（如JSON）输出。像GPT-3.5这样的模型可以做到这一点，但有时会遇到困难。这自然而然地指向了回退 - 我们可以尝试使用GPT-3.5（更快，更便宜），但如果解析失败，我们可以使用GPT-4。

```python
from langchain.output_parsers import DatetimeOutputParser
```

```python
prompt = ChatPromptTemplate.from_template(
    "what time was {event} (in %Y-%m-%dT%H:%M:%S.%fZ format - only return this value)"
)
```

```python
# 在这种情况下，我们将在LLM +输出解析器级别上进行回退
# 因为错误将在OutputParser中引发
openai_35 = ChatOpenAI() | DatetimeOutputParser()
openai_4 = ChatOpenAI(model="gpt-4") | DatetimeOutputParser()
```

```python
only_35 = prompt | openai_35
fallback_4 = prompt | openai_35.with_fallbacks([openai_4])
```

```python
try:
    print(only_35.invoke({"event": "the superbowl in 1994"}))
except Exception as e:
    print(f"Error: {e}")
```

```output
Error: Could not parse datetime string: The Super Bowl in 1994 took place on January 30th at 3:30 PM local time. Converting this to the specified format (%Y-%m-%dT%H:%M:%S.%fZ) results in: 1994-01-30T15:30:00.000Z
```

```python
try:
    print(fallback_4.invoke({"event": "the superbowl in 1994"}))
except Exception as e:
    print(f"Error: {e}")
```

```output
1994-01-30 15:30:00
```