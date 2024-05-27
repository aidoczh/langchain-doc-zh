# 响应元数据

许多模型提供商在其聊天生成响应中包含一些元数据。可以通过 `AIMessage.response_metadata: Dict` 属性访问这些元数据。根据模型提供商和模型配置的不同，这些元数据可以包含诸如[令牌计数](/docs/how_to/chat_token_usage_tracking)、[对数概率](/docs/how_to/logprobs)等信息。

以下是几个不同提供商的响应元数据示例：

## OpenAI

```python
from langchain_openai import ChatOpenAI
llm = ChatOpenAI(model="gpt-4-turbo")
msg = llm.invoke([("human", "What's the oldest known example of cuneiform")])
msg.response_metadata
```

```output
{'token_usage': {'completion_tokens': 164,
  'prompt_tokens': 17,
  'total_tokens': 181},
 'model_name': 'gpt-4-turbo',
 'system_fingerprint': 'fp_76f018034d',
 'finish_reason': 'stop',
 'logprobs': None}
```

## Anthropic

```python
from langchain_anthropic import ChatAnthropic
llm = ChatAnthropic(model="claude-3-sonnet-20240229")
msg = llm.invoke([("human", "What's the oldest known example of cuneiform")])
msg.response_metadata
```

```output
{'id': 'msg_01CzQyD7BX8nkhDNfT1QqvEp',
 'model': 'claude-3-sonnet-20240229',
 'stop_reason': 'end_turn',
 'stop_sequence': None,
 'usage': {'input_tokens': 17, 'output_tokens': 296}}
```

## Google VertexAI

```python
from langchain_google_vertexai import ChatVertexAI
llm = ChatVertexAI(model="gemini-pro")
msg = llm.invoke([("human", "What's the oldest known example of cuneiform")])
msg.response_metadata
```

```output
{'is_blocked': False,
 'safety_ratings': [{'category': 'HARM_CATEGORY_HATE_SPEECH',
   'probability_label': 'NEGLIGIBLE',
   'blocked': False},
  {'category': 'HARM_CATEGORY_DANGEROUS_CONTENT',
   'probability_label': 'NEGLIGIBLE',
   'blocked': False},
  {'category': 'HARM_CATEGORY_HARASSMENT',
   'probability_label': 'NEGLIGIBLE',
   'blocked': False},
  {'category': 'HARM_CATEGORY_SEXUALLY_EXPLICIT',
   'probability_label': 'NEGLIGIBLE',
   'blocked': False}],
 'citation_metadata': None,
 'usage_metadata': {'prompt_token_count': 10,
  'candidates_token_count': 30,
  'total_token_count': 40}}
```

## Bedrock (Anthropic)

```python
from langchain_aws import ChatBedrock
llm = ChatBedrock(model_id="anthropic.claude-v2")
msg = llm.invoke([("human", "What's the oldest known example of cuneiform")])
msg.response_metadata
```

```output
{'model_id': 'anthropic.claude-v2',
 'usage': {'prompt_tokens': 19, 'completion_tokens': 371, 'total_tokens': 390}}
```

## MistralAI

```python
from langchain_mistralai import ChatMistralAI
llm = ChatMistralAI()
msg = llm.invoke([("human", "What's the oldest known example of cuneiform")])
msg.response_metadata
```

```output
{'token_usage': {'prompt_tokens': 19,
  'total_tokens': 141,
  'completion_tokens': 122},
 'model': 'mistral-small',
 'finish_reason': 'stop'}
```

## Groq

```python
from langchain_groq import ChatGroq
llm = ChatGroq()
msg = llm.invoke([("human", "What's the oldest known example of cuneiform")])
msg.response_metadata
```

```output
{'token_usage': {'completion_time': 0.243,
  'completion_tokens': 132,
  'prompt_time': 0.022,
  'prompt_tokens': 22,
  'queue_time': None,
  'total_time': 0.265,
  'total_tokens': 154},
 'model_name': 'mixtral-8x7b-32768',
 'system_fingerprint': 'fp_7b44c65f25',
 'finish_reason': 'stop',
 'logprobs': None}
```

## TogetherAI

```python
import os
from langchain_openai import ChatOpenAI
llm = ChatOpenAI(
    base_url="https://api.together.xyz/v1",
    api_key=os.environ["TOGETHER_API_KEY"],
    model="mistralai/Mixtral-8x7B-Instruct-v0.1",
)
msg = llm.invoke([("human", "What's the oldest known example of cuneiform")])
msg.response_metadata
```

```output
{'token_usage': {'completion_tokens': 208,
  'prompt_tokens': 20,
  'total_tokens': 228},
 'model_name': 'mistralai/Mixtral-8x7B-Instruct-v0.1',
 'system_fingerprint': None,
 'finish_reason': 'eos',
 'logprobs': None}
```

## FireworksAI

```python
from langchain_fireworks import ChatFireworks
llm = ChatFireworks(model="accounts/fireworks/models/mixtral-8x7b-instruct")
msg = llm.invoke([("human", "What's the oldest known example of cuneiform")])
msg.response_metadata
```

```output
{'token_usage': {'prompt_tokens': 19,
  'total_tokens': 219,
  'completion_tokens': 200},
 'model_name': 'accounts/fireworks/models/mixtral-8x7b-instruct',
 'system_fingerprint': '',
 'finish_reason': 'length',
 'logprobs': None}
```