# 印象笔记

[EverNote](https://evernote.com/) 旨在存档和创建笔记，其中可以嵌入照片、音频和保存的网页内容。笔记存储在虚拟的 "笔记本" 中，可以进行标记、注释、编辑、搜索和导出。

这个笔记本展示了如何从磁盘加载 `Evernote` [导出](https://help.evernote.com/hc/en-us/articles/209005557-Export-notes-and-notebooks-as-ENEX-or-HTML) 文件（.enex）。

每个笔记都会创建一个文档。

```python
# 需要使用 lxml 和 html2text 来解析 EverNote 笔记
%pip install --upgrade --quiet  lxml
%pip install --upgrade --quiet  html2text
```

```python
from langchain_community.document_loaders import EverNoteLoader
# 默认情况下，所有笔记都会合并成一个文档
loader = EverNoteLoader("example_data/testing.enex")
loader.load()
```

```output
[Document(page_content='testing this\n\nwhat happens?\n\nto the world?**Jan - March 2022**', metadata={'source': 'example_data/testing.enex'})]
```

```python
# 返回每个笔记的文档可能更有用
loader = EverNoteLoader("example_data/testing.enex", load_single_document=False)
loader.load()
```

```output
[Document(page_content='testing this\n\nwhat happens?\n\nto the world?', metadata={'title': 'testing', 'created': time.struct_time(tm_year=2023, tm_mon=2, tm_mday=9, tm_hour=3, tm_min=47, tm_sec=46, tm_wday=3, tm_yday=40, tm_isdst=-1), 'updated': time.struct_time(tm_year=2023, tm_mon=2, tm_mday=9, tm_hour=3, tm_min=53, tm_sec=28, tm_wday=3, tm_yday=40, tm_isdst=-1), 'note-attributes.author': 'Harrison Chase', 'source': 'example_data/testing.enex'}),
 Document(page_content='**Jan - March 2022**', metadata={'title': 'Summer Training Program', 'created': time.struct_time(tm_year=2022, tm_mon=12, tm_mday=27, tm_hour=1, tm_min=59, tm_sec=48, tm_wday=1, tm_yday=361, tm_isdst=-1), 'note-attributes.author': 'Mike McGarry', 'note-attributes.source': 'mobile.iphone', 'source': 'example_data/testing.enex'})]
```