---

sidebar_position: 4

---

# 如何部分格式化提示模板

:::info 先决条件

本指南假定您熟悉以下概念：

- [提示模板](/docs/concepts/#prompt-templates)

:::

就像部分绑定函数参数一样，对提示模板进行“部分化”也是有意义的 - 例如，传入所需值的子集，以创建一个新的提示模板，该模板仅期望剩余的值子集。

LangChain 支持两种方式实现这一点：

1. 使用字符串值进行部分格式化。

2. 使用返回字符串值的函数进行部分格式化。

在下面的示例中，我们将讨论这两种用例的动机以及如何在 LangChain 中实现。

## 使用字符串进行部分化

想要部分化提示模板的一个常见用例是，如果您在链中的某些变量比其他变量更早获得访问权限。例如，假设您有一个需要两个变量 `foo` 和 `baz` 的提示模板。如果您在链的早期获得了 `foo` 值，但稍后才获得 `baz` 值，那么通过整个链传递这两个变量可能会很不方便。相反，您可以使用 `foo` 值对提示模板进行部分化，然后传递部分化的提示模板，并只使用那个。以下是执行此操作的示例：

```python
from langchain_core.prompts import PromptTemplate
prompt = PromptTemplate.from_template("{foo}{bar}")
partial_prompt = prompt.partial(foo="foo")
print(partial_prompt.format(bar="baz"))
```

```output
foobaz
```

您也可以直接使用部分化的变量初始化提示。

```python
prompt = PromptTemplate(
    template="{foo}{bar}", input_variables=["bar"], partial_variables={"foo": "foo"}
)
print(prompt.format(bar="baz"))
```

```output
foobaz
```

## 使用函数进行部分化

另一个常见用例是使用函数进行部分化。这种情况的用例是，当您知道您总是希望以常见方式获取某个变量时。一个典型的例子是日期或时间。想象一下，您有一个提示，您总是希望其中包含当前日期。您不能在提示中硬编码它，并且将其与其他输入变量一起传递是不方便的。在这种情况下，能够使用始终返回当前日期的函数对提示进行部分化是很方便的。

```python
from datetime import datetime
def _get_datetime():
    now = datetime.now()
    return now.strftime("%m/%d/%Y, %H:%M:%S")
prompt = PromptTemplate(
    template="告诉我一个{adjective}关于{date}的笑话",
    input_variables=["adjective", "date"],
)
partial_prompt = prompt.partial(date=_get_datetime)
print(partial_prompt.format(adjective="有趣的"))
```

```output
告诉我一个有趣的关于04/21/2024, 19:43:57的笑话
```

您也可以直接使用部分化的变量初始化提示，这在这种工作流程中通常更有意义。

```python
prompt = PromptTemplate(
    template="告诉我一个{adjective}关于{date}的笑话",
    input_variables=["adjective"],
    partial_variables={"date": _get_datetime},
)
print(prompt.format(adjective="有趣的"))
```

```output
告诉我一个有趣的关于04/21/2024, 19:43:57的笑话
```

## 下一步

您现在已经学会了如何将变量部分应用到您的提示模板中。

接下来，请查看本节中关于提示模板的其他操作指南，比如[向您的提示模板添加少量示例](/docs/how_to/few_shot_examples_chat)。