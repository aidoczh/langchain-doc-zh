# MHTML

MHTML 既可用于电子邮件，也可用于存档网页。MHTML，有时也称为 MHT，代表 MIME HTML，是一个包含整个网页的单个文件。当将网页保存为 MHTML 格式时，该文件扩展名将包含 HTML 代码、图像、音频文件、Flash 动画等。

```python
from langchain_community.document_loaders import MHTMLLoader
```

```python
# 创建一个 MHTML 文件的新加载器对象
loader = MHTMLLoader(
    file_path="../../../../../../tests/integration_tests/examples/example.mht"
)
# 从文件中加载文档
documents = loader.load()
# 打印文档以查看结果
for doc in documents:
    print(doc)
```

```output
page_content='LangChain\nLANG CHAIN 🦜️🔗Official Home Page\xa0\n\n\n\n\n\n\n\nIntegrations\n\n\n\nFeatures\n\n\n\n\nBlog\n\n\n\nConceptual Guide\n\n\n\n\nPython Repo\n\n\nJavaScript Repo\n\n\n\nPython Documentation \n\n\nJavaScript Documentation\n\n\n\n\nPython ChatLangChain \n\n\nJavaScript ChatLangChain\n\n\n\n\nDiscord \n\n\nTwitter\n\n\n\n\nIf you have any comments about our WEB page, you can \nwrite us at the address shown above.  However, due to \nthe limited number of personnel in our corporate office, we are unable to \nprovide a direct response.\n\nCopyright © 2023-2023 LangChain Inc.\n\n\n' metadata={'source': '../../../../../../tests/integration_tests/examples/example.mht', 'title': 'LangChain'}
```
