# 文件系统

LangChain提供了与本地文件系统交互的工具。本文将介绍其中一些工具。

**注意：** 不建议在受控环境之外使用这些工具！

首先，我们将导入这些工具。

```python
from tempfile import TemporaryDirectory
from langchain_community.agent_toolkits import FileManagementToolkit
# 我们将创建一个临时目录，以避免混乱
working_directory = TemporaryDirectory()
```

## 文件管理工具包

如果您想为您的代理提供所有文件工具，使用该工具包非常简单。我们将临时目录作为根目录传递给LLM作为工作空间。

建议始终传入一个根目录，因为如果没有根目录，LLM很容易污染工作目录，并且没有根目录，就无法对直接提示注入进行验证。

```python
toolkit = FileManagementToolkit(
    root_dir=str(working_directory.name)
)  # 如果不提供root_dir，操作将默认为当前工作目录
toolkit.get_tools()
```

```output
[CopyFileTool(root_dir='/tmp/tmprdvsw3tg'),
 DeleteFileTool(root_dir='/tmp/tmprdvsw3tg'),
 FileSearchTool(root_dir='/tmp/tmprdvsw3tg'),
 MoveFileTool(root_dir='/tmp/tmprdvsw3tg'),
 ReadFileTool(root_dir='/tmp/tmprdvsw3tg'),
 WriteFileTool(root_dir='/tmp/tmprdvsw3tg'),
 ListDirectoryTool(root_dir='/tmp/tmprdvsw3tg')]
```

### 选择文件系统工具

如果您只想选择特定的工具，可以在初始化工具包时将它们作为参数传入，或者可以单独初始化所需的工具。

```python
tools = FileManagementToolkit(
    root_dir=str(working_directory.name),
    selected_tools=["read_file", "write_file", "list_directory"],
).get_tools()
tools
```

```output
[ReadFileTool(root_dir='/tmp/tmprdvsw3tg'),
 WriteFileTool(root_dir='/tmp/tmprdvsw3tg'),
 ListDirectoryTool(root_dir='/tmp/tmprdvsw3tg')]
```

```python
read_tool, write_tool, list_tool = tools
write_tool.invoke({"file_path": "example.txt", "text": "Hello World!"})
```

```output
'文件已成功写入到 example.txt。'
```

```python
# 列出工作目录中的文件
list_tool.invoke({})
```

```output
'example.txt'
```