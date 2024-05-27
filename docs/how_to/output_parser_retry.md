# 当出现解析错误时如何重试

在某些情况下，通过仅查看输出就可以修复任何解析错误，但在其他情况下则不行。一个例子是当输出不仅格式不正确，而且部分内容缺失时。请看下面的例子。

```python
from langchain.output_parsers import OutputFixingParser
from langchain_core.output_parsers import PydanticOutputParser
from langchain_core.prompts import PromptTemplate
from langchain_core.pydantic_v1 import BaseModel, Field
from langchain_openai import ChatOpenAI, OpenAI
```

```python
template = """根据用户问题，提供一个动作和动作输入，说明应该采取哪个步骤。
{format_instructions}
问题：{query}
回复："""
class Action(BaseModel):
    action: str = Field(description="要执行的动作")
    action_input: str = Field(description="动作的输入")
parser = PydanticOutputParser(pydantic_object=Action)
```

```python
prompt = PromptTemplate(
    template="回答用户的问题。\n{format_instructions}\n{query}\n",
    input_variables=["query"],
    partial_variables={"format_instructions": parser.get_format_instructions()},
)
```

```python
prompt_value = prompt.format_prompt(query="who is leo di caprios gf?")
```

```python
bad_response = '{"action": "search"}'
```

如果我们尝试直接解析这个响应，将会出现错误：

```python
parser.parse(bad_response)
```

```output
---------------------------------------------------------------------------
ValidationError                           Traceback (most recent call last)
File ~/workplace/langchain/libs/langchain/langchain/output_parsers/pydantic.py:30, in PydanticOutputParser.parse(self, text)
     29     json_object = json.loads(json_str, strict=False)
---> 30     return self.pydantic_object.parse_obj(json_object)
     32 except (json.JSONDecodeError, ValidationError) as e:
File ~/.pyenv/versions/3.10.1/envs/langchain/lib/python3.10/site-packages/pydantic/main.py:526, in pydantic.main.BaseModel.parse_obj()
File ~/.pyenv/versions/3.10.1/envs/langchain/lib/python3.10/site-packages/pydantic/main.py:341, in pydantic.main.BaseModel.__init__()
ValidationError: 1 validation error for Action
action_input
  field required (type=value_error.missing)
During handling of the above exception, another exception occurred:
OutputParserException                     Traceback (most recent call last)
Cell In[6], line 1
----> 1 parser.parse(bad_response)
File ~/workplace/langchain/libs/langchain/langchain/output_parsers/pydantic.py:35, in PydanticOutputParser.parse(self, text)
     33 name = self.pydantic_object.__name__
     34 msg = f"Failed to parse {name} from completion {text}. Got: {e}"
---> 35 raise OutputParserException(msg, llm_output=text)
OutputParserException: Failed to parse Action from completion {"action": "search"}. Got: 1 validation error for Action
action_input
  field required (type=value_error.missing)
```

如果我们尝试使用 `OutputFixingParser` 来修复这个错误，它会感到困惑 - 它不知道实际应该为动作输入放入什么内容。

```python
fix_parser = OutputFixingParser.from_llm(parser=parser, llm=ChatOpenAI())
```

```python
fix_parser.parse(bad_response)
```

```output
Action(action='search', action_input='input')
```

相反，我们可以使用 `RetryOutputParser`，它会传入提示（以及原始输出）再次尝试获得更好的响应。

```python
from langchain.output_parsers import RetryOutputParser
```

```python
retry_parser = RetryOutputParser.from_llm(parser=parser, llm=OpenAI(temperature=0))
```

```python
retry_parser.parse_with_prompt(bad_response, prompt_value)
```

```output
Action(action='search', action_input='leo di caprio girlfriend')
```

我们还可以轻松地通过自定义链将 `RetryOutputParser` 添加到一个自定义链中，将原始的 LLM/ChatModel 输出转换为更易处理的格式。

```python
from langchain_core.runnables import RunnableLambda, RunnableParallel
completion_chain = prompt | OpenAI(temperature=0)
main_chain = RunnableParallel(
    completion=completion_chain, prompt_value=prompt
) | RunnableLambda(lambda x: retry_parser.parse_with_prompt(**x))
main_chain.invoke({"query": "who is leo di caprios gf?"})
```

```output
Action(action='search', action_input='leo di caprio girlfriend')
```

查看 [RetryOutputParser](https://api.python.langchain.com/en/latest/output_parsers/langchain.output_parsers.retry.RetryOutputParser.html#langchain.output_parsers.retry.RetryOutputParser) 的 API 文档。