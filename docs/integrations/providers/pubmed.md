# PubMed

>[PubMed®](https://pubmed.ncbi.nlm.nih.gov/)由`国家生物技术信息中心，国家医学图书馆`提供，包括来自`MEDLINE`、生命科学期刊和在线图书的超过3500万条生物医学文献引用。引用可能包括指向`PubMed Central`和出版商网站的全文内容的链接。

## 设置

您需要安装一个Python包。

```bash
pip install xmltodict
```

### 检索器

查看[使用示例](/docs/integrations/retrievers/pubmed)。

```python
from langchain.retrievers import PubMedRetriever
```

### 文档加载器

查看[使用示例](/docs/integrations/document_loaders/pubmed)。

```python
from langchain_community.document_loaders import PubMedLoader
```