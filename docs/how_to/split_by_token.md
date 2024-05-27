

# 如何通过标记来分割文本

语言模型有一个标记限制。您不应超过标记限制。因此，当您将文本分成块时，最好计算标记数。有许多标记器。在计算文本中的标记数时，应使用与语言模型中使用的相同的标记器。

## tiktoken

:::note

[tiktoken](https://github.com/openai/tiktoken) 是由 OpenAI 创建的快速 `BPE` 标记器。

:::

我们可以使用 `tiktoken` 来估算使用的标记数。对于 OpenAI 模型，这可能会更准确。

1. 文本如何分割：按传入的字符进行分割。

2. 如何测量块大小：通过 `tiktoken` 标记器。

[CharacterTextSplitter](https://api.python.langchain.com/en/latest/character/langchain_text_splitters.character.CharacterTextSplitter.html)、[RecursiveCharacterTextSplitter](https://api.python.langchain.com/en/latest/character/langchain_text_splitters.character.RecursiveCharacterTextSplitter.html) 和 [TokenTextSplitter](https://api.python.langchain.com/en/latest/base/langchain_text_splitters.base.TokenTextSplitter.html) 可以直接与 `tiktoken` 一起使用。

```python
%pip install --upgrade --quiet langchain-text-splitters tiktoken
```

```python
from langchain_text_splitters import CharacterTextSplitter
# 这是一个长文档，我们可以将其分割。
with open("state_of_the_union.txt") as f:
    state_of_the_union = f.read()
```

要使用 [CharacterTextSplitter](https://api.python.langchain.com/en/latest/character/langchain_text_splitters.character.CharacterTextSplitter.html) 进行分割，然后使用 `tiktoken` 合并块，请使用其 `.from_tiktoken_encoder()` 方法。请注意，此方法生成的分割可能比 `tiktoken` 标记器测量的块大小要大。

`.from_tiktoken_encoder()` 方法接受 `encoding_name`（例如 `cl100k_base`）或 `model_name`（例如 `gpt-4`）作为参数。所有额外的参数，如 `chunk_size`、`chunk_overlap` 和 `separators`，都用于实例化 `CharacterTextSplitter`：

```python
text_splitter = CharacterTextSplitter.from_tiktoken_encoder(
    encoding_name="cl100k_base", chunk_size=100, chunk_overlap=0
)
texts = text_splitter.split_text(state_of_the_union)
```

```python
print(texts[0])
```

```output
女士们、先生们，我们的第一夫人和第二先生。国会议员和内阁成员。最高法院大法官。我的美国同胞。
去年，COVID-19让我们分开。今年，我们终于又在一起了。
今晚，我们作为民主党人、共和党人和独立人士相聚一堂。但更重要的是作为美国人。
我们有责任对彼此，对美国人民，对宪法。
```

要对块大小实施硬约束，我们可以使用 `RecursiveCharacterTextSplitter.from_tiktoken_encoder`，如果块大小较大，则会递归分割每个块：

```python
from langchain_text_splitters import RecursiveCharacterTextSplitter
text_splitter = RecursiveCharacterTextSplitter.from_tiktoken_encoder(
    model_name="gpt-4",
    chunk_size=100,
    chunk_overlap=0,
)
```

我们还可以加载一个 `TokenTextSplitter` 分割器，它直接与 `tiktoken` 一起使用，并确保每个分割块都比块大小小。

```python
from langchain_text_splitters import TokenTextSplitter
text_splitter = TokenTextSplitter(chunk_size=10, chunk_overlap=0)
texts = text_splitter.split_text(state_of_the_union)
print(texts[0])
```

```output
女士们、先生们，我们的
```

一些书面语言（例如中文和日文）的字符编码为 2 个或更多个标记。直接使用 `TokenTextSplitter` 可能会导致字符的标记在两个块之间分割，从而导致不正确的 Unicode 字符。请使用 `RecursiveCharacterTextSplitter.from_tiktoken_encoder` 或 `CharacterTextSplitter.from_tiktoken_encoder` 来确保块包含有效的 Unicode 字符。

## spaCy

:::note

[spaCy](https://spacy.io/) 是一个用 Python 和 Cython 编写的高级自然语言处理开源软件库。

:::

LangChain 实现了基于 [spaCy 标记器](https://spacy.io/api/tokenizer)的分割器。

1. 文本如何分割：通过 `spaCy` 标记器。

2. 如何测量块大小：按字符数。

```python
%pip install --upgrade --quiet  spacy
```

```python
# 这是一个长文档，我们可以将其分割。
with open("state_of_the_union.txt") as f:
    state_of_the_union = f.read()
```

```python
from langchain_text_splitters import SpacyTextSplitter
text_splitter = SpacyTextSplitter(chunk_size=1000)
texts = text_splitter.split_text(state_of_the_union)
print(texts[0])
```

```output
女士们、先生们，我们的第一夫人和第二先生。
国会议员和内阁成员。
最高法院大法官。
我的美国同胞。
去年，COVID-19让我们分开。
今年，我们终于又在一起了。
今晚，我们作为民主党人、共和党人和独立人士相聚一堂。
但更重要的是作为美国人。
我们有责任对彼此，对美国人民，对宪法。
并且坚定地认为自由永远会战胜暴政。
六天前，俄罗斯的弗拉基米尔·普京试图动摇自由世界的基础，认为他可以让其屈服于他威胁的方式。
但他严重误判了。
他以为可以进入乌克兰，而世界会屈服。
相反，他遇到了他从未想象过的强大力量。
他遇到了乌克兰人民。
从泽连斯基总统到每个乌克兰人，他们的无畏、勇气和决心激励着世界。
```

## SentenceTransformers

[SentenceTransformersTokenTextSplitter](https://api.python.langchain.com/en/latest/sentence_transformers/langchain_text_splitters.sentence_transformers.SentenceTransformersTokenTextSplitter.html) 是专为句子转换模型设计的专用文本分割器。默认行为是将文本分割成符合您想要使用的句子转换模型的标记窗口的块。

要根据句子转换器的标记器拆分文本并限制标记数，请实例化 `SentenceTransformersTokenTextSplitter`。您可以选择指定：

- `chunk_overlap`：标记重叠的整数计数；

- `model_name`：句子转换模型名称，默认为`"sentence-transformers/all-mpnet-base-v2"`；

- `tokens_per_chunk`：每个块的期望标记计数。

```python
from langchain_text_splitters import SentenceTransformersTokenTextSplitter
splitter = SentenceTransformersTokenTextSplitter(chunk_overlap=0)
text = "Lorem "
count_start_and_stop_tokens = 2
text_token_count = splitter.count_tokens(text=text) - count_start_and_stop_tokens
print(text_token_count)
```

```output
2
```

```python
token_multiplier = splitter.maximum_tokens_per_chunk // text_token_count + 1
# `text_to_split` 无法放入单个块中
text_to_split = text * token_multiplier
print(f"tokens in text to split: {splitter.count_tokens(text=text_to_split)}")
```

```output
tokens in text to split: 514
```

```python
text_chunks = splitter.split_text(text=text_to_split)
print(text_chunks[1])
```

```output
lorem
```

## NLTK

:::note

[自然语言工具包](https://en.wikipedia.org/wiki/Natural_Language_Toolkit)（The Natural Language Toolkit），更常称为 [NLTK](https://www.nltk.org/)，是用Python编程语言编写的一套用于符号和统计自然语言处理（NLP）的库和程序。

:::

与其仅仅在"\n\n"上拆分不同，我们可以使用 `NLTK` 基于 [NLTK tokenizers](https://www.nltk.org/api/nltk.tokenize.html) 进行拆分。

1. 文本如何拆分：通过 `NLTK` 分词器。

2. 块大小如何测量：按字符数。

```python
# pip install nltk
```

```python
# 这是一个可以拆分的长文档。
with open("state_of_the_union.txt") as f:
    state_of_the_union = f.read()
```

```python
from langchain_text_splitters import NLTKTextSplitter
text_splitter = NLTKTextSplitter(chunk_size=1000)
```

```python
texts = text_splitter.split_text(state_of_the_union)
print(texts[0])
```

```output
女士们先生们，副总统女士，我们的第一夫人和第二绅士。
国会议员和内阁成员。
最高法院法官们。
我的美国同胞们。
去年，COVID-19让我们分开。
今年，我们终于又聚在一起了。
今晚，我们作为民主党人、共和党人和独立人士相聚。
但更重要的是作为美国人。
我们对彼此、对美国人民、对宪法负有责任。
并怀着坚定的决心，自由将永远战胜暴政。
六天前，俄罗斯的弗拉基米尔·普京试图动摇自由世界的基础，以为他可以让其屈服于他的威胁方式。
但他严重误判了。
他以为可以轻易进入乌克兰，而世界会屈服。
相反，他遇到了他从未想象过的坚强墙壁。
他遇到了乌克兰人民。
从泽连斯基总统到每一位乌克兰人，他们的无畏、勇气和决心激励着世界。
一群公民用自己的身体挡住坦克。
```

## KoNLPY

:::note

[KoNLPy: Korean NLP in Python](https://konlpy.org/en/latest/) 是用于处理韩语自然语言处理（NLP）的Python包。

:::

标记拆分涉及将文本分割成更小、更易处理的单元，称为标记。这些标记通常是单词、短语、符号或其他对进一步处理和分析至关重要的有意义元素。在英语等语言中，标记拆分通常涉及通过空格和标点符号分隔单词。标记拆分的有效性在很大程度上取决于标记器对语言结构的理解，确保生成有意义的标记。由于为英语设计的标记器无法理解韩语等其他语言的独特语义结构，因此无法有效用于韩语处理。

### 使用KoNLPy的Kkma分析器进行韩语的标记拆分

对于韩语文本，KoNLPY包含一个称为 `Kkma`（韩语知识形态分析器）的形态分析器。`Kkma`提供韩语文本的详细形态分析。它将句子分解为单词，将单词分解为各自的形态素，为每个标记识别词性。它可以将一块文本分割成单独的句子，这对于处理长文本特别有用。

### 使用注意事项

尽管 `Kkma` 以其详细的分析而闻名，但重要的是要注意这种精度可能会影响处理速度。因此，`Kkma` 最适用于将分析深度置于快速文本处理之上的应用程序。

```python
# 使用 pip 安装 konlpy
```

```python
# 这是一个长篇的韩文文档，我们希望将其拆分成各个句子。
with open("./your_korean_doc.txt") as f:
    korean_document = f.read()
```

```python
from langchain_text_splitters import KonlpyTextSplitter
text_splitter = KonlpyTextSplitter()
```

```python
texts = text_splitter.split_text(korean_document)
# 句子之间以 "\n\n" 字符分隔。
print(texts[0])
```

```output
춘향전 옛날에 남원에 이 도령이라는 벼슬아치 아들이 있었다。
그의 외모는 빛나는 달처럼 잘생겼고, 그의 학식과 기예는 남보다 뛰어났다。
한편, 이 마을에는 춘향이라는 절세 가인이 살고 있었다。
춘 향의 아름다움은 꽃과 같아 마을 사람들 로부터 많은 사랑을 받았다。
어느 봄날, 도령은 친구들과 놀러 나갔다가 춘 향을 만 나 첫 눈에 반하고 말았다。
두 사람은 서로 사랑하게 되었고, 이내 비밀스러운 사랑의 맹세를 나누었다。
하지만 좋은 날들은 오래가지 않았다。
도령의 아버지가 다른 곳으로 전근을 가게 되어 도령도 떠나 야만 했다。
이별의 아픔 속에서도, 두 사람은 재회를 기약하며 서로를 믿고 기다리기로 했다。
그러나 새로 부임한 관아의 사또가 춘 향의 아름다움에 욕심을 내 어 그녀에게 강요를 시작했다。
춘 향 은 도령에 대한 자신의 사랑을 지키기 위해, 사또의 요구를 단호히 거절했다。
이에 분노한 사또는 춘 향을 감옥에 가두고 혹독한 형벌을 내렸다。
이야기는 이 도령이 고위 관직에 오른 후, 춘 향을 구해 내는 것으로 끝난다。
두 사람은 오랜 시련 끝에 다시 만나게 되고, 그들의 사랑은 온 세상에 전해 지며 후세에까지 이어진다。
- 춘향전 (The Tale of Chunhyang)
```

## Hugging Face 分词器

[Hugging Face](https://huggingface.co/docs/tokenizers/index) 有许多分词器。

我们使用 Hugging Face 分词器，[GPT2TokenizerFast](https://huggingface.co/Ransaka/gpt2-tokenizer-fast) 来计算文本长度（以 token 为单位）。

1. 文本如何分割：按字符分割。

2. 块大小如何测量：由 `Hugging Face` 分词器计算的 token 数量。

```python
from transformers import GPT2TokenizerFast
tokenizer = GPT2TokenizerFast.from_pretrained("gpt2")
```

```python
# 这是一个我们可以拆分的长文档。
with open("state_of_the_union.txt") as f:
    state_of_the_union = f.read()
from langchain_text_splitters import CharacterTextSplitter
```

```python
text_splitter = CharacterTextSplitter.from_huggingface_tokenizer(
    tokenizer, chunk_size=100, chunk_overlap=0
)
texts = text_splitter.split_text(state_of_the_union)
```

```python
print(texts[0])
```

```output
Madam Speaker, Madam Vice President, our First Lady and Second Gentleman. Members of Congress and the Cabinet. Justices of the Supreme Court. My fellow Americans.  
Last year COVID-19 kept us apart. This year we are finally together again. 
Tonight, we meet as Democrats Republicans and Independents. But most importantly as Americans. 
With a duty to one another to the American people to the Constitution.
```