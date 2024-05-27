# 如何从目录加载文档

LangChain的[DirectoryLoader](https://api.python.langchain.com/en/latest/document_loaders/langchain_community.document_loaders.directory.DirectoryLoader.html)实现了从磁盘读取文件到LangChain [Document](https://api.python.langchain.com/en/latest/documents/langchain_core.documents.base.Document.html#langchain_core.documents.base.Document)对象的功能。以下是演示内容：

- 如何从文件系统加载，包括使用通配符模式；

- 如何使用多线程进行文件I/O；

- 如何使用自定义加载器类来解析特定文件类型（例如代码）；

- 如何处理错误，比如由于解码而导致的错误。

```python
from langchain_community.document_loaders import DirectoryLoader
```

`DirectoryLoader`接受一个`loader_cls`关键字参数，默认为[UnstructuredLoader](/docs/integrations/document_loaders/unstructured_file)。[Unstructured](https://unstructured-io.github.io/unstructured/)支持解析多种格式，如PDF和HTML。这里我们用它来读取一个markdown（.md）文件。

我们可以使用`glob`参数来控制加载哪些文件。请注意，这里不加载`.rst`文件或`.html`文件。

```python
loader = DirectoryLoader("../", glob="**/*.md")
docs = loader.load()
len(docs)
```

```output
20
```

```python
print(docs[0].page_content[:100])
```

```output
Security
LangChain has a large ecosystem of integrations with various external resources like local
```

## 显示进度条

默认情况下不会显示进度条。要显示进度条，请安装`tqdm`库（例如`pip install tqdm`），并将`show_progress`参数设置为`True`。

```python
loader = DirectoryLoader("../", glob="**/*.md", show_progress=True)
docs = loader.load()
```

```output
100%|█████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████| 20/20 [00:00<00:00, 54.56it/s]
```

## 使用多线程

默认情况下，加载在一个线程中进行。为了利用多个线程，将`use_multithreading`标志设置为true。

```python
loader = DirectoryLoader("../", glob="**/*.md", use_multithreading=True)
docs = loader.load()
```

## 更改加载器类

默认情况下使用`UnstructuredLoader`类。要自定义加载器，请在`loader_cls`关键字参数中指定加载器类。下面我们展示一个使用[TextLoader](https://api.python.langchain.com/en/latest/document_loaders/langchain_community.document_loaders.text.TextLoader.html)的示例：

```python
from langchain_community.document_loaders import TextLoader
loader = DirectoryLoader("../", glob="**/*.md", loader_cls=TextLoader)
docs = loader.load()
```

```python
print(docs[0].page_content[:100])
```

```output
# Security
LangChain has a large ecosystem of integrations with various external resources like loc
```

请注意，虽然`UnstructuredLoader`解析Markdown标题，但`TextLoader`不会。

如果需要加载Python源代码文件，请使用`PythonLoader`：

```python
from langchain_community.document_loaders import PythonLoader
loader = DirectoryLoader("../../../../../", glob="**/*.py", loader_cls=PythonLoader)
```

## 使用TextLoader自动检测文件编码

`DirectoryLoader`可以帮助处理由文件编码变化引起的错误。下面我们将尝试加载一组文件，其中一个包含非UTF8编码。

```python
path = "../../../../libs/langchain/tests/unit_tests/examples/"
loader = DirectoryLoader(path, glob="**/*.txt", loader_cls=TextLoader)
```

### A. 默认行为

默认情况下会引发错误：

```python
loader.load()
```

```output
Error loading file ../../../../libs/langchain/tests/unit_tests/examples/example-non-utf8.txt
---------------------------------------------------------------------------
UnicodeDecodeError                        Traceback (most recent call last)
File ~/repos/langchain/libs/community/langchain_community/document_loaders/text.py:43, in TextLoader.lazy_load(self)
     42     with open(self.file_path, encoding=self.encoding) as f:
---> 43         text = f.read()
File ~/.pyenv/versions/3.10.4/lib/python3.10/codecs.py:322, in BufferedIncrementalDecoder.decode(self, input, final)
    321 data = self.buffer + input
--> 322 (result, consumed) = self._buffer_decode(data, self.errors, final)
UnicodeDecodeError: 'utf-8' codec can't decode byte 0xca in position 0: invalid continuation byte
RuntimeError                              Traceback (most recent call last)
Cell In[10], line 1
----> 1 loader.load()
File ~/repos/langchain/libs/community/langchain_community/document_loaders/directory.py:117, in DirectoryLoader.load(self)
    115 def load(self) -> List[Document]:
    116     """Load documents."""
--> 117     return list(self.lazy_load())
File ~/repos/langchain/libs/community/langchain_community/document_loaders/directory.py:182, in DirectoryLoader.lazy_load(self)
    180 else:
    181     for i in items:
--> 182         yield from self._lazy_load_file(i, p, pbar)
File ~/repos/langchain/libs/community/langchain_community/document_loaders/directory.py:220, in DirectoryLoader._lazy_load_file(self, item, path, pbar)
    218     else:
    219         logger.error(f"Error loading file {str(item)}")
--> 220         raise e
File ~/repos/langchain/libs/community/langchain_community/document_loaders/directory.py:210, in DirectoryLoader._lazy_load_file(self, item, path, pbar)
    208 loader = self.loader_cls(str(item), **self.loader_kwargs)
    209 try:
--> 210     for subdoc in loader.lazy_load():
    211         yield subdoc
    212 except NotImplementedError:
File ~/repos/langchain/libs/community/langchain_community/document_loaders/text.py:56, in TextLoader.lazy_load(self)
     54                 continue
     55     else:
---> 56         raise RuntimeError(f"Error loading {self.file_path}") from e
     57 except Exception as e:
     58     raise RuntimeError(f"Error loading {self.file_path}") from e
RuntimeError: Error loading ../../../../libs/langchain/tests/unit_tests/examples/example-non-utf8.txt
```

文件 `example-non-utf8.txt` 使用了不同的编码，因此 `load()` 函数会失败，并显示一条有用的消息，指出哪个文件解码失败了。

使用 `TextLoader` 的默认行为，如果加载文档失败，整个加载过程将失败，没有任何文档被加载。

### B. 静默失败

我们可以向 `DirectoryLoader` 传递参数 `silent_errors`，以跳过无法加载的文件并继续加载过程。

```python
loader = DirectoryLoader(
    path, glob="**/*.txt", loader_cls=TextLoader, silent_errors=True
)
docs = loader.load()
```

```output
Error loading file ../../../../libs/langchain/tests/unit_tests/examples/example-non-utf8.txt: Error loading ../../../../libs/langchain/tests/unit_tests/examples/example-non-utf8.txt
```

```python
doc_sources = [doc.metadata["source"] for doc in docs]
doc_sources
```

```output
['../../../../libs/langchain/tests/unit_tests/examples/example-utf8.txt']
```

### C. 自动检测编码

我们还可以要求 `TextLoader` 在失败之前自动检测文件编码，通过向加载器类传递 `autodetect_encoding` 参数。

```python
text_loader_kwargs = {"autodetect_encoding": True}
loader = DirectoryLoader(
    path, glob="**/*.txt", loader_cls=TextLoader, loader_kwargs=text_loader_kwargs
)
docs = loader.load()
```

```python
doc_sources = [doc.metadata["source"] for doc in docs]
doc_sources
```

```output
['../../../../libs/langchain/tests/unit_tests/examples/example-utf8.txt',
 '../../../../libs/langchain/tests/unit_tests/examples/example-non-utf8.txt']
```

```