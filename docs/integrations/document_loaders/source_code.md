# 源代码

本文介绍了如何使用语言解析的特殊方法加载源代码文件：代码中的每个顶层函数和类都会被加载到单独的文档中。任何剩余的顶层代码（即已加载的函数和类之外的代码）将被加载到单独的文档中。

这种方法可以潜在地提高对源代码的问答模型的准确性。

支持的代码解析语言有：

- C (*)

- C++ (*)

- C# (*)

- COBOL

- Go (*)

- Java (*)

- JavaScript（需要包`esprima`）

- Kotlin (*)

- Lua (*)

- Perl (*)

- Python

- Ruby (*)

- Rust (*)

- Scala (*)

- TypeScript (*)

带有(*)标记的项目需要安装`tree_sitter`和`tree_sitter_languages`包。

可以通过`tree_sitter`轻松添加对其他语言的支持，尽管目前需要修改LangChain。

解析使用的语言可以进行配置，还可以配置基于语法激活分割的最小行数。

如果没有明确指定语言，`LanguageParser`将根据文件扩展名推断语言。

```python
%pip install -qU esprima esprima tree_sitter tree_sitter_languages
```

```python
import warnings
warnings.filterwarnings("ignore")
from pprint import pprint
from langchain_community.document_loaders.generic import GenericLoader
from langchain_community.document_loaders.parsers import LanguageParser
from langchain_text_splitters import Language
```

```python
loader = GenericLoader.from_filesystem(
    "./example_data/source_code",
    glob="*",
    suffixes=[".py", ".js"],
    parser=LanguageParser(),
)
docs = loader.load()
```

```python
len(docs)
```

```output
6
```

```python
for document in docs:
    pprint(document.metadata)
```

```output
{'content_type': 'functions_classes',
 'language': <Language.PYTHON: 'python'>,
 'source': 'example_data/source_code/example.py'}
{'content_type': 'functions_classes',
 'language': <Language.PYTHON: 'python'>,
 'source': 'example_data/source_code/example.py'}
{'content_type': 'simplified_code',
 'language': <Language.PYTHON: 'python'>,
 'source': 'example_data/source_code/example.py'}
{'content_type': 'functions_classes',
 'language': <Language.JS: 'js'>,
 'source': 'example_data/source_code/example.js'}
{'content_type': 'functions_classes',
 'language': <Language.JS: 'js'>,
 'source': 'example_data/source_code/example.js'}
{'content_type': 'simplified_code',
 'language': <Language.JS: 'js'>,
 'source': 'example_data/source_code/example.js'}
```

```python
print("\n\n--8<--\n\n".join([document.page_content for document in docs]))
```

```output
class MyClass:
    def __init__(self, name):
        self.name = name
    def greet(self):
        print(f"Hello, {self.name}!")
--8<--
def main():
    name = input("Enter your name: ")
    obj = MyClass(name)
    obj.greet()
--8<--
# Code for: class MyClass:
# Code for: def main():
if __name__ == "__main__":
    main()
--8<--
class MyClass {
  constructor(name) {
    this.name = name;
  }
  greet() {
    console.log(`Hello, ${this.name}!`);
  }
}
--8<--
function main() {
  const name = prompt("Enter your name:");
  const obj = new MyClass(name);
  obj.greet();
}
--8<--
// Code for: class MyClass {
// Code for: function main() {
main();
```

对于小文件，可以禁用解析器。

参数`parser_threshold`表示源代码文件必须具有的最小行数，才能使用解析器进行分割。

```python
loader = GenericLoader.from_filesystem(
    "./example_data/source_code",
    glob="*",
    suffixes=[".py"],
    parser=LanguageParser(language=Language.PYTHON, parser_threshold=1000),
)
docs = loader.load()
```

```python
len(docs)
```

```output
1
```

```python
print(docs[0].page_content)
```

```output
class MyClass:
    def __init__(self, name):
        self.name = name
    def greet(self):
        print(f"Hello, {self.name}!")
def main():
    name = input("Enter your name: ")
    obj = MyClass(name)
    obj.greet()
if __name__ == "__main__":
    main()
```

## 分割

对于那些太大的函数、类或脚本，可能需要进行额外的分割。

```python
loader = GenericLoader.from_filesystem(
    "./example_data/source_code",
    glob="*",
    suffixes=[".js"],
    parser=LanguageParser(language=Language.JS),
)
docs = loader.load()
```

```python
from langchain_text_splitters import (
    Language,
    RecursiveCharacterTextSplitter,
)
```

```python
js_splitter = RecursiveCharacterTextSplitter.from_language(
    language=Language.JS, chunk_size=60, chunk_overlap=0
)
```

```python
result = js_splitter.split_documents(docs)
```

```python
len(result)
```

```output
7
```

```python
print("\n\n--8<--\n\n".join([document.page_content for document in result]))
```

```output
class MyClass {
  constructor(name) {
    this.name = name;
--8<--
}
--8<--
greet() {
    console.log(`Hello, ${this.name}!`);
  }
}
--8<--
function main() {
  const name = prompt("Enter your name:");
--8<--
const obj = new MyClass(name);
  obj.greet();
}
--8<--
// Code for: class MyClass {
// Code for: function main() {
--8<--
main();
```

## 使用 Tree-sitter 模板添加语言

使用 Tree-sitter 模板扩展语言支持需要完成以下几个关键步骤：

1. **创建新的语言文件**：

    - 首先在指定目录 (langchain/libs/community/langchain_community/document_loaders/parsers/language) 中创建一个新文件。

    - 参照现有的 **`cpp.py`** 文件的结构和解析逻辑来建立这个文件。

    - 你还需要在 langchain 目录下创建一个文件 (langchain/libs/langchain/langchain/document_loaders/parsers/language)。

2. **解析特定语言**：

    - 模仿 **`cpp.py`** 文件中使用的结构，调整以适应你要引入的语言。

    - 主要的修改涉及调整块查询数组，以适应你要解析的语言的语法和结构。

3. **测试语言解析器**：

    - 为了全面验证，生成一个特定于新语言的测试文件。在指定目录 (langchain/libs/community/tests/unit_tests/document_loaders/parsers/language) 中创建 **`test_language.py`**。

    - 参照 **`test_cpp.py`** 的示例，为新语言中解析的元素建立基本测试。

4. **集成到解析器和文本拆分器中**：

    - 在 **`language_parser.py`** 文件中整合你的新语言。确保更新 LANGUAGE_EXTENSIONS 和 LANGUAGE_SEGMENTERS，以及 LanguageParser 的文档字符串，以便识别和处理添加的语言。

    - 同样，确认你的语言已包含在 **`text_splitter.py`** 的 Language 类中，以便进行正确的解析。

通过遵循这些步骤，并确保全面测试和集成，你将成功地使用 Tree-sitter 模板扩展语言支持。

祝你好运！

```