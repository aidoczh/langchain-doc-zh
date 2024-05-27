# 如何通过字符递归分割文本

这个文本分割器是用于通用文本的推荐工具。它接受一个字符列表作为参数。它会按顺序尝试在这些字符上进行分割，直到块足够小。默认的字符列表是 `["\n\n", "\n", " ", ""]`。这样做的效果是尽可能保持所有段落（然后是句子，再然后是单词）在一起，因为这些通常看起来是语义上相关的文本块。

1. 文本如何分割：根据字符列表。

2. 块大小如何衡量：根据字符数量。

下面我们展示一个使用示例。

要直接获取字符串内容，请使用 `.split_text`。

要创建 LangChain [Document](https://api.python.langchain.com/en/latest/documents/langchain_core.documents.base.Document.html) 对象（例如，用于下游任务），请使用 `.create_documents`。

```python
%pip install -qU langchain-text-splitters
```

```python
from langchain_text_splitters import RecursiveCharacterTextSplitter
# 加载示例文档
with open("state_of_the_union.txt") as f:
    state_of_the_union = f.read()
text_splitter = RecursiveCharacterTextSplitter(
    # 设置一个非常小的块大小，只是为了展示。
    chunk_size=100,
    chunk_overlap=20,
    length_function=len,
    is_separator_regex=False,
)
texts = text_splitter.create_documents([state_of_the_union])
print(texts[0])
print(texts[1])
```

```output
page_content='Madam Speaker, Madam Vice President, our First Lady and Second Gentleman. Members of Congress and'
page_content='of Congress and the Cabinet. Justices of the Supreme Court. My fellow Americans.'
```

```python
text_splitter.split_text(state_of_the_union)[:2]
```

```output
['Madam Speaker, Madam Vice President, our First Lady and Second Gentleman. Members of Congress and',
 'of Congress and the Cabinet. Justices of the Supreme Court. My fellow Americans.']
```

让我们来看看上述 `RecursiveCharacterTextSplitter` 的参数设置：

- `chunk_size`：块的最大大小，大小由 `length_function` 决定。

- `chunk_overlap`：块之间的目标重叠。重叠的块有助于在上下文分割时减少信息丢失。

- `length_function`：确定块大小的函数。

- `is_separator_regex`：分隔符列表（默认为 `["\n\n", "\n", " ", ""]`）是否应被解释为正则表达式。

## 从没有词边界的语言中分割文本

一些书写系统没有[词边界](https://en.wikipedia.org/wiki/Category:Writing_systems_without_word_boundaries)，例如中文、日文和泰文。使用默认分隔符列表 `["\n\n", "\n", " ", ""]` 分割文本可能会导致单词被分割在不同块之间。为了保持单词在一起，您可以覆盖分隔符列表，包括额外的标点符号：

* 添加 ASCII 句号 "`.`"，[Unicode 全角](https://en.wikipedia.org/wiki/Halfwidth_and_Fullwidth_Forms_(Unicode_block)) 句号 "`．`"（用于中文文本），以及[表意句号](https://en.wikipedia.org/wiki/CJK_Symbols_and_Punctuation) "`。`"（用于日文和中文）

* 添加[零宽空格](https://en.wikipedia.org/wiki/Zero-width_space) 用于泰文、缅甸文、高棉文和日文。

* 添加 ASCII 逗号 "`,`"，Unicode 全角逗号 "`，`"，以及 Unicode 表意逗号 "`、`"

```python
text_splitter = RecursiveCharacterTextSplitter(
    separators=[
        "\n\n",
        "\n",
        " ",
        ".",
        ",",
        "\u200b",  # 零宽空格
        "\uff0c",  # 全角逗号
        "\u3001",  # 表意逗号
        "\uff0e",  # 全角句号
        "\u3002",  # 表意句号
        "",
    ],
    # 已有的参数
)
```