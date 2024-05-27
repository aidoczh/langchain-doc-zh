# YouTube音频

在YouTube视频上构建聊天或问答应用程序是一个非常感兴趣的话题。

下面我们将展示如何从`YouTube链接`到`视频的音频`再到`文本`最后到`聊天`！

我们将使用`OpenAIWhisperParser`，它将使用OpenAI Whisper API将音频转录为文本，

以及`OpenAIWhisperParserLocal`用于本地支持和在私有云或本地运行。

注意：您需要提供一个`OPENAI_API_KEY`。

```python
from langchain_community.document_loaders.blob_loaders.youtube_audio import (
    YoutubeAudioLoader,
)
from langchain_community.document_loaders.generic import GenericLoader
from langchain_community.document_loaders.parsers import (
    OpenAIWhisperParser,
    OpenAIWhisperParserLocal,
)
```

我们将使用`yt_dlp`下载YouTube链接的音频。

我们将使用`pydub`来拆分下载的音频文件（以符合Whisper API的25MB文件大小限制）。

```python
%pip install --upgrade --quiet  yt_dlp
%pip install --upgrade --quiet  pydub
%pip install --upgrade --quiet  librosa
```

### YouTube链接转文本

使用`YoutubeAudioLoader`来获取/下载音频文件。

然后，使用`OpenAIWhisperParser()`将它们转录为文本。

让我们以Andrej Karpathy的YouTube课程的第一讲为例！

```python
# 设置一个标志以在本地和远程解析之间切换
# 如果要使用本地解析，请将此标志更改为True
local = False
```

```python
# 两个Karpathy讲座视频
urls = ["https://youtu.be/kCc8FmEb1nY", "https://youtu.be/VMj-3S1tku0"]
# 保存音频文件的目录
save_dir = "~/Downloads/YouTube"
# 将视频转录为文本
if local:
    loader = GenericLoader(
        YoutubeAudioLoader(urls, save_dir), OpenAIWhisperParserLocal()
    )
else:
    loader = GenericLoader(YoutubeAudioLoader(urls, save_dir), OpenAIWhisperParser())
docs = loader.load()
```

```output
[youtube] Extracting URL: https://youtu.be/kCc8FmEb1nY
[youtube] kCc8FmEb1nY: Downloading webpage
[youtube] kCc8FmEb1nY: Downloading android player API JSON
[info] kCc8FmEb1nY: Downloading 1 format(s): 140
[dashsegments] Total fragments: 11
[download] Destination: /Users/31treehaus/Desktop/AI/langchain-fork/docs/modules/indexes/document_loaders/examples/Let's build GPT： from scratch, in code, spelled out..m4a
[download] 100% of  107.73MiB in 00:00:18 at 5.92MiB/s
[FixupM4a] Correcting container of "/Users/31treehaus/Desktop/AI/langchain-fork/docs/modules/indexes/document_loaders/examples/Let's build GPT： from scratch, in code, spelled out..m4a"
[ExtractAudio] Not converting audio /Users/31treehaus/Desktop/AI/langchain-fork/docs/modules/indexes/document_loaders/examples/Let's build GPT： from scratch, in code, spelled out..m4a; file is already in target format m4a
[youtube] Extracting URL: https://youtu.be/VMj-3S1tku0
[youtube] VMj-3S1tku0: Downloading webpage
[youtube] VMj-3S1tku0: Downloading android player API JSON
[info] VMj-3S1tku0: Downloading 1 format(s): 140
[download] /Users/31treehaus/Desktop/AI/langchain-fork/docs/modules/indexes/document_loaders/examples/The spelled-out intro to neural networks and backpropagation： building micrograd.m4a has already been downloaded
[download] 100% of  134.98MiB
[ExtractAudio] Not converting audio /Users/31treehaus/Desktop/AI/langchain-fork/docs/modules/indexes/document_loaders/examples/The spelled-out intro to neural networks and backpropagation： building micrograd.m4a; file is already in target format m4a
```

```python
# 返回一个文档列表，可以轻松查看或解析
docs[0].page_content[0:500]
```

```output
"Hello, my name is Andrej and I've been training deep neural networks for a bit more than a decade. And in this lecture I'd like to show you what neural network training looks like under the hood. So in particular we are going to start with a blank Jupyter notebook and by the end of this lecture we will define and train a neural net and you'll get to see everything that goes on under the hood and exactly sort of how that works on an intuitive level. Now specifically what I would like to do is I w"
```

### 从YouTube视频构建聊天应用

给定`Documents`，我们可以轻松地启用聊天/问答。

```python
from langchain.chains import RetrievalQA
from langchain_community.vectorstores import FAISS
from langchain_openai import ChatOpenAI, OpenAIEmbeddings
from langchain_text_splitters import RecursiveCharacterTextSplitter
```

```python
# 合并文档
combined_docs = [doc.page_content for doc in docs]
text = " ".join(combined_docs)
```

```python
# 拆分文本
text_splitter = RecursiveCharacterTextSplitter(chunk_size=1500, chunk_overlap=150)
splits = text_splitter.split_text(text)
```

```python
# 构建索引
embeddings = OpenAIEmbeddings()
vectordb = FAISS.from_texts(splits, embeddings)
```

```python
# 构建QA链
qa_chain = RetrievalQA.from_chain_type(
    llm=ChatOpenAI(model="gpt-3.5-turbo", temperature=0),
    chain_type="stuff",
    retriever=vectordb.as_retriever(),
)
```

```python
# 提出一个问题！
query = "为什么我们需要在每一步反向传播之前将梯度清零？"
qa_chain.run(query)
```

```output
"我们需要在每一步反向传播之前将梯度清零，因为反向传播会在每个参数的 grad 属性中累积梯度。如果我们不在每次反向传播之前将 grad 重置为零，梯度将会累积并相加，导致更新不正确并且收敛速度变慢。通过在每次反向传播之前将 grad 重置为零，我们确保梯度被正确计算，并且优化过程能够正常进行。"
```

```python
query = "编码器和解码器有什么区别？"
qa_chain.run(query)
```

```output
'在transformer的背景下，编码器是一个组件，用于读取输入标记序列并生成隐藏表示序列。另一方面，解码器是一个组件，用于接收隐藏表示序列并生成输出标记序列。两者之间的主要区别在于编码器用于将输入序列编码为固定长度表示，而解码器用于将固定长度表示解码为输出序列。例如，在机器翻译中，编码器读取源语言句子并生成固定长度表示，然后解码器使用该表示生成目标语言句子。'
```

```python
query = "对于任何标记，x、k、v 和 q 分别是什么？"
qa_chain.run(query)
```

```output
'对于任何标记，x 是包含该标记私有信息的输入向量，k 和 q 分别是键和查询向量，它们是通过在 x 上进行线性模块前向传播而产生的，v 是通过再次在 x 上进行线性模块前向传播而计算出的向量。键向量表示标记包含的信息，查询向量表示标记正在寻找的内容。如果标记发现其他标记有趣，向量 v 就是它将要传达给其他标记的信息，并且它会被用于自注意力机制的聚合。'
```