# 如何根据长度选择示例

该示例选择器根据长度选择要使用的示例。当您担心构建的提示会超过上下文窗口的长度时，这将非常有用。对于较长的输入，它会选择较少的示例进行包含，而对于较短的输入，它会选择更多的示例。

```python
from langchain_core.example_selectors import LengthBasedExampleSelector
from langchain_core.prompts import FewShotPromptTemplate, PromptTemplate
# 创建反义词的假任务示例。
examples = [
    {"input": "happy", "output": "sad"},
    {"input": "tall", "output": "short"},
    {"input": "energetic", "output": "lethargic"},
    {"input": "sunny", "output": "gloomy"},
    {"input": "windy", "output": "calm"},
]
example_prompt = PromptTemplate(
    input_variables=["input", "output"],
    template="输入：{input}\n输出：{output}",
)
example_selector = LengthBasedExampleSelector(
    # 可供选择的示例。
    examples=examples,
    # 用于格式化示例的 PromptTemplate。
    example_prompt=example_prompt,
    # 格式化示例的最大长度。
    # 长度由下面的 get_text_length 函数测量。
    max_length=25,
    # 用于获取字符串长度的函数，用于确定要包含的示例。
    # 如果未指定，则提供一个默认值。
    # get_text_length: Callable[[str], int] = lambda x: len(re.split("\n| ", x))
)
dynamic_prompt = FewShotPromptTemplate(
    # 我们提供一个 ExampleSelector 而不是示例。
    example_selector=example_selector,
    example_prompt=example_prompt,
    prefix="给出每个输入的反义词",
    suffix="输入：{adjective}\n输出：",
    input_variables=["adjective"],
)
```

```python
# 一个输入较小的示例，因此它选择所有示例。
print(dynamic_prompt.format(adjective="big"))
```

```output
给出每个输入的反义词
输入：happy
输出：sad
输入：tall
输出：short
输入：energetic
输出：lethargic
输入：sunny
输出：gloomy
输入：windy
输出：calm
输入：big
输出：
```

```python
# 一个输入较长的示例，因此它只选择一个示例。
long_string = "big and huge and massive and large and gigantic and tall and much much much much much bigger than everything else"
print(dynamic_prompt.format(adjective=long_string))
```

```output
给出每个输入的反义词
输入：happy
输出：sad
输入：big and huge and massive and large and gigantic and tall and much much much much much bigger than everything else
输出：
```

```python
# 您也可以将示例添加到示例选择器中。
new_example = {"input": "big", "output": "small"}
dynamic_prompt.example_selector.add_example(new_example)
print(dynamic_prompt.format(adjective="enthusiastic"))
```

```output
给出每个输入的反义词
输入：happy
输出：sad
输入：tall
输出：short
输入：energetic
输出：lethargic
输入：sunny
输出：gloomy
输入：windy
输出：calm
输入：big
输出：small
输入：enthusiastic
输出：
```