# BibTeX

[BibTeX](https://www.ctan.org/pkg/bibtex) 是一种文件格式和参考文献管理系统，通常与 `LaTeX` 排版一起使用。它用于组织和存储学术和研究文档的文献信息。

`BibTeX` 文件的扩展名为 `.bib`，由表示各种出版物（如书籍、文章、会议论文、论文等）的纯文本条目组成。每个 `BibTeX` 条目都遵循特定的结构，并包含不同的文献详细信息字段，如作者姓名、出版物标题、期刊或书籍标题、出版年份、页码等。

BibTeX 文件还可以存储文档的路径，例如可以检索的 `.pdf` 文件。

## 安装

首先，您需要安装 `bibtexparser` 和 `PyMuPDF`。

```python
%pip install --upgrade --quiet  bibtexparser pymupdf
```

## 示例

`BibtexLoader` 有以下参数：

- `file_path`：`.bib` bibtex 文件的路径

- 可选参数 `max_docs`：默认为 `None`，即不限制。使用它来限制检索到的文档数量。

- 可选参数 `max_content_chars`：默认为 4000。使用它来限制单个文档中的字符数。

- 可选参数 `load_extra_meta`：默认为 `False`。默认情况下，仅从 bibtex 条目中加载最重要的字段：`Published`（出版年份）、`Title`、`Authors`、`Summary`、`Journal`、`Keywords` 和 `URL`。如果为 `True`，它还将尝试加载返回 `entry_id`、`note`、`doi` 和 `links` 字段。

- 可选参数 `file_pattern`：默认为 `r'[^:]+\.pdf'`。用于在 `file` 条目中查找文件的正则表达式模式。默认模式支持 `Zotero` 风格的 bibtex 格式和裸文件路径。

```python
from langchain_community.document_loaders import BibtexLoader
```

```python
# 创建一个虚拟的 bibtex 文件并下载一个 pdf。
import urllib.request
urllib.request.urlretrieve(
    "https://www.fourmilab.ch/etexts/einstein/specrel/specrel.pdf", "einstein1905.pdf"
)
bibtex_text = """
    @article{einstein1915,
        title={Die Feldgleichungen der Gravitation},
        abstract={Die Grundgleichungen der Gravitation, die ich hier entwickeln werde, wurden von mir in einer Abhandlung: ,,Die formale Grundlage der allgemeinen Relativit{\"a}tstheorie`` in den Sitzungsberichten der Preu{\ss}ischen Akademie der Wissenschaften 1915 ver{\"o}ffentlicht.},
        author={Einstein, Albert},
        journal={Sitzungsberichte der K{\"o}niglich Preu{\ss}ischen Akademie der Wissenschaften},
        volume={1915},
        number={1},
        pages={844--847},
        year={1915},
        doi={10.1002/andp.19163540702},
        link={https://onlinelibrary.wiley.com/doi/abs/10.1002/andp.19163540702},
        file={einstein1905.pdf}
    }
    """
# 将 bibtex_text 保存到 biblio.bib 文件中
with open("./biblio.bib", "w") as file:
    file.write(bibtex_text)
```

```python
docs = BibtexLoader("./biblio.bib").load()
```

```python
docs[0].metadata
```

```output
{'id': 'einstein1915',
 'published_year': '1915',
 'title': 'Die Feldgleichungen der Gravitation',
 'publication': 'Sitzungsberichte der K{"o}niglich Preu{\\ss}ischen Akademie der Wissenschaften',
 'authors': 'Einstein, Albert',
 'abstract': 'Die Grundgleichungen der Gravitation, die ich hier entwickeln werde, wurden von mir in einer Abhandlung: ,,Die formale Grundlage der allgemeinen Relativit{"a}tstheorie`` in den Sitzungsberichten der Preu{\\ss}ischen Akademie der Wissenschaften 1915 ver{"o}ffentlicht.',
 'url': 'https://doi.org/10.1002/andp.19163540702'}
```

```python
print(docs[0].page_content[:400])  # pdf 内容的所有页面
```

```output
ON THE ELECTRODYNAMICS OF MOVING
BODIES
By A. EINSTEIN
June 30, 1905
It is known that Maxwell’s electrodynamics—as usually understood at the
present time—when applied to moving bodies, leads to asymmetries which do
not appear to be inherent in the phenomena. Take, for example, the recipro-
cal electrodynamic action of a magnet and a conductor. The observable phe-
nomenon here depends only on the r
```