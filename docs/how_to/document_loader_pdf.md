# 如何加载PDF文件

[便携式文档格式（PDF）](https://zh.wikipedia.org/wiki/PDF)是由Adobe于1992年开发的一种文件格式，标准化为ISO 32000。它以一种与应用软件、硬件和操作系统无关的方式呈现文档，包括文本格式和图像。

本指南介绍了如何将PDF文档加载到我们在下游使用的LangChain [Document](https://api.python.langchain.com/en/latest/documents/langchain_core.documents.base.Document.html#langchain_core.documents.base.Document)格式中。

LangChain集成了许多PDF解析器。有些解析器简单且相对低级，而其他解析器支持OCR和图像处理，或进行高级文档布局分析。选择合适的解析器将取决于您的应用程序。下面我们列举了一些可能的选择。

## 使用PyPDF

这里我们使用`pypdf`将PDF加载为文档数组，其中每个文档包含页面内容和带有`page`编号的元数据。

```python
%pip install pypdf
```
```python
from langchain_community.document_loaders import PyPDFLoader
file_path = ("../../../docs/integrations/document_loaders/example_data/layout-parser-paper.pdf")
loader = PyPDFLoader(file_path)
pages = loader.load_and_split()
pages[0]
```
```output
Document(page_content='LayoutParser : A Uniﬁed Toolkit for Deep\nLearning Based Document Image Analysis\nZejiang Shen1( \x00), Ruochen Zhang2, Melissa Dell3, Benjamin Charles Germain\nLee4, Jacob Carlson3, and Weining Li5\n1Allen Institute for AI\nshannons@allenai.org\n2Brown University\nruochen zhang@brown.edu\n3Harvard University\n{melissadell,jacob carlson }@fas.harvard.edu\n4University of Washington\nbcgl@cs.washington.edu\n5University of Waterloo\nw422li@uwaterloo.ca\nAbstract. Recent advances in document image analysis (DIA) have been\nprimarily driven by the application of neural networks. Ideally, research\noutcomes could be easily deployed in production and extended for further\ninvestigation. However, various factors like loosely organized codebases\nand sophisticated model conﬁgurations complicate the easy reuse of im-\nportant innovations by a wide audience. Though there have been on-going\neﬀorts to improve reusability and simplify deep learning (DL) model\ndevelopment in disciplines like natural language processing and computer\nvision, none of them are optimized for challenges in the domain of DIA.\nThis represents a major gap in the existing toolkit, as DIA is central to\nacademic research across a wide range of disciplines in the social sciences\nand humanities. This paper introduces LayoutParser , an open-source\nlibrary for streamlining the usage of DL in DIA research and applica-\ntions. The core LayoutParser library comes with a set of simple and\nintuitive interfaces for applying and customizing DL models for layout de-\ntection, character recognition, and many other document processing tasks.\nTo promote extensibility, LayoutParser also incorporates a community\nplatform for sharing both pre-trained models and full document digiti-\nzation pipelines. We demonstrate that LayoutParser is helpful for both\nlightweight and large-scale digitization pipelines in real-word use cases.\nThe library is publicly available at https://layout-parser.github.io .\nKeywords: Document Image Analysis ·Deep Learning ·Layout Analysis\n·Character Recognition ·Open Source library ·Toolkit.\n1 Introduction\nDeep Learning(DL)-based approaches are the state-of-the-art for a wide range of\ndocument image analysis (DIA) tasks including document image classiﬁcation [ 11,arXiv:2103.15348v2  [cs.CV]  21 Jun 2021', metadata={'source': '../../../docs/integrations/document_loaders/example_data/layout-parser-paper.pdf', 'page': 0})
```

这种方法的优点是可以通过页码检索文档。

### 对PDF进行向量搜索

一旦我们将PDF加载到LangChain的`Document`对象中，我们可以像通常一样对它们进行索引（例如，RAG应用程序）。

```python
import getpass
import os
os.environ["OPENAI_API_KEY"] = getpass.getpass("OpenAI API Key:")
```
```python
from langchain_community.vectorstores import FAISS
from langchain_openai import OpenAIEmbeddings
faiss_index = FAISS.from_documents(pages, OpenAIEmbeddings())
docs = faiss_index.similarity_search("What is LayoutParser?", k=2)
for doc in docs:
    print(str(doc.metadata["page"]) + ":", doc.page_content[:300])
```
```output
13: 14 Z. Shen et al.
6 Conclusion
LayoutParser provides a comprehensive toolkit for deep learning-based document
image analysis. The oﬀ-the-shelf library is easy to install, and can be used to
build ﬂexible and accurate pipelines for processing documents with complicated
structures. It also supports hi
0: LayoutParser : A Uniﬁed Toolkit for Deep
Learning Based Document Image Analysis
Zejiang Shen1(), Ruochen Zhang2, Melissa Dell3, Benjamin Charles Germain
Lee4, Jacob Carlson3, and Weining Li5
1Allen Institute for AI
shannons@allenai.org
2Brown University
ruochen zhang@brown.edu
3Harvard University
### 从图像中提取文本
一些 PDF 包含文本图像，例如扫描文档或图表。使用 `rapidocr-onnxruntime` 软件包，我们也可以将图像提取为文本：
```python
%pip install rapidocr-onnxruntime
```
```python
loader = PyPDFLoader("https://arxiv.org/pdf/2103.15348.pdf", extract_images=True)
pages = loader.load()
pages[4].page_content
```
```output

'LayoutParser : A Uniﬁed Toolkit for DL-Based DIA 5\nTable 1: Current layout detection models in the LayoutParser model zoo\nDataset Base Model1Large Model Notes\nPubLayNet [38] F / M M Layouts of modern scientiﬁc documents\nPRImA [3] M - Layouts of scanned modern magazines and scientiﬁc reports\nNewspaper [17] F - Layouts of scanned US newspapers from the 20th century\nTableBank [18] F F Table region on modern scientiﬁc and business document\nHJDataset [31] F / M - Layouts of history Japanese documents\n1For each dataset, we train several models of diﬀerent sizes for diﬀerent needs (the trade-oﬀ between accuracy\nvs. computational cost). For “base model” and “large model”, we refer to using the ResNet 50 or ResNet 101\nbackbones [ 13], respectively. One can train models of diﬀerent architectures, like Faster R-CNN [ 28] (F) and Mask\nR-CNN [ 12] (M). For example, an F in the Large Model column indicates it has a Faster R-CNN model trained\nusing the ResNet 101 backbone. The platform is maintained and a number of additions will be made to the model\nzoo in coming months.\nlayout data structures , which are optimized for eﬃciency and versatility. 3) When\nnecessary, users can employ existing or customized OCR models via the uniﬁed\nAPI provided in the OCR module . 4)LayoutParser comes with a set of utility\nfunctions for the visualization and storage of the layout data. 5) LayoutParser\nis also highly customizable, via its integration with functions for layout data\nannotation and model training . We now provide detailed descriptions for each\ncomponent.\n3.1 Layout Detection Models\nInLayoutParser , a layout model takes a document image as an input and\ngenerates a list of rectangular boxes for the target content regions. Diﬀerent\nfrom traditional methods, it relies on deep convolutional neural networks rather\nthan manually curated rules to identify content regions. It is formulated as an\nobject detection problem and state-of-the-art models like Faster R-CNN [ 28] and\nMask R-CNN [ 12] are used. This yields prediction results of high accuracy and\nmakes it possible to build a concise, generalized interface for layout detection.\nLayoutParser , built upon Detectron2 [ 35], provides a minimal API that can\nperform layout detection with only four lines of code in Python:\n1import layoutparser as lp\n2image = cv2. imread (" image_file ") # load images\n3model = lp. Detectron2LayoutModel (\n4 "lp :// PubLayNet / faster_rcnn_R_50_FPN_3x / config ")\n5layout = model . detect ( image )\nLayoutParser provides a wealth of pre-trained model weights using various\ndatasets covering diﬀerent languages, time periods, and document types. Due to\ndomain shift [ 7], the prediction performance can notably drop when models are ap-\nplied to target samples that are signiﬁcantly diﬀerent from the training dataset. As\ndocument structures and layouts vary greatly in diﬀerent domains, it is important\nto select models trained on a dataset similar to the test samples. A semantic syntax\nis used for initializing the model weights in LayoutParser , using both the dataset\nname and model name lp://<dataset-name>/<model-architecture-name> .'

```
## 使用 PyMuPDF
这是 PDF 解析选项中最快的选项，包含有关 PDF 及其页面的详细元数据，并且每页返回一个文档：
```python
from langchain_community.document_loaders import PyMuPDFLoader
loader = PyMuPDFLoader("example_data/layout-parser-paper.pdf")
data = loader.load()
data[0]
```
另外，您可以在 `load` 调用中作为关键字参数传递 [PyMuPDF 文档](https://pymupdf.readthedocs.io/en/latest/app1.html#plain-text/) 中的任何选项，并将其传递给 `get_text()` 调用。
## 使用 MathPix
受 Daniel Gross 的 [https://gist.github.com/danielgross/3ab4104e14faccc12b49200843adab21](https://gist.github.com/danielgross/3ab4104e14faccc12b49200843adab21) 的启发：
```python
from langchain_community.document_loaders import MathpixPDFLoader
file_path = (
    "../../../docs/integrations/document_loaders/example_data/layout-parser-paper.pdf"
)
loader = MathpixPDFLoader(file_path)
data = loader.load()
```
## 使用 Unstructured
[Unstructured](https://unstructured-io.github.io/unstructured/) 支持一个通用接口，用于处理非结构化或半结构化文件格式，例如 Markdown 或 PDF。LangChain 的 [UnstructuredPDFLoader](https://api.python.langchain.com/en/latest/document_loaders/langchain_community.document_loaders.pdf.UnstructuredPDFLoader.html) 与 Unstructured 集成，将 PDF 文档解析为 LangChain [Document](https://api.python.langchain.com/en/latest/documents/langchain_core.documents.base.Document.html) 对象。
```python
from langchain_community.document_loaders import UnstructuredPDFLoader
file_path = (
    "../../../docs/integrations/document_loaders/example_data/layout-parser-paper.pdf"
)
loader = UnstructuredPDFLoader(file_path)
data = loader.load()
```
### 保留元素
在幕后，Unstructured 为不同的文本块创建不同的 "元素"。默认情况下，我们将它们合并在一起，但您可以通过指定 `mode="elements"` 轻松保持分离。
```python
file_path = (
    "../../../docs/integrations/document_loaders/example_data/layout-parser-paper.pdf"
)
loader = UnstructuredPDFLoader(file_path, mode="elements")
data = loader.load()
data[0]
```
```output

Document(page_content='1 2 0 2', metadata={'source': '../../../docs/integrations/document_loaders/example_data/layout-parser-paper.pdf', 'coordinates': {'points': ((16.34, 213.36), (16.34, 253.36), (36.34, 253.36), (36.34, 213.36)), 'system': 'PixelSpace', 'layout_width': 612, 'layout_height': 792}, 'file_directory': '../../../docs/integrations/document_loaders/example_data', 'filename': 'layout-parser-paper.pdf', 'languages': ['eng'], 'last_modified': '2024-03-18T13:22:22', 'page_number': 1, 'filetype': 'application/pdf', 'category': 'UncategorizedText'})

```
查看此特定文档的完整元素类型集合：
```python
set(doc.metadata["category"] for doc in data)
```
```output

{'ListItem', 'NarrativeText', 'Title', 'UncategorizedText'}

```
### 使用 Unstructured 加载远程 PDF
这涵盖了如何将在线 PDF 加载到我们可以在下游使用的文档格式中。这可用于各种在线 PDF 站点，如 https://open.umn.edu/opentextbooks/textbooks/ 和 https://arxiv.org/archive/
注意：所有其他 PDF 加载器也可以用于获取远程 PDF，但 `OnlinePDFLoader` 是一个旧函数，专门与 `UnstructuredPDFLoader` 配合使用。
```python
from langchain_community.document_loaders import OnlinePDFLoader
loader = OnlinePDFLoader("https://arxiv.org/pdf/2302.03803.pdf")
data = loader.load()
```
## 使用 PyPDFium2
```python
from langchain_community.document_loaders import PyPDFium2Loader
file_path = (
    "../../../docs/integrations/document_loaders/example_data/layout-parser-paper.pdf"
)
loader = PyPDFium2Loader(file_path)
data = loader.load()
```
## 使用 PDFMiner
```python
from langchain_community.document_loaders import PDFMinerLoader
file_path = (
    "../../../docs/integrations/document_loaders/example_data/layout-parser-paper.pdf"
)
loader = PDFMinerLoader(file_path)
data = loader.load()
```
### 使用 PDFMiner 生成 HTML 文本
这对于将文本在语义上分块很有帮助，因为输出的 HTML 内容可以通过 `BeautifulSoup` 进行解析，以获取有关字体大小、页码、PDF 页眉/页脚等更结构化和丰富的信息。
```python
from langchain_community.document_loaders import PDFMinerPDFasHTMLLoader
file_path = (
    "../../../docs/integrations/document_loaders/example_data/layout-parser-paper.pdf"
)
loader = PDFMinerPDFasHTMLLoader(file_path)
data = loader.load()[0]
```
```python
from bs4 import BeautifulSoup
soup = BeautifulSoup(data.page_content, "html.parser")
content = soup.find_all("div")
```
```python
import re
cur_fs = None
cur_text = ""
snippets = []  # 首先收集所有具有相同字体大小的片段
for c in content:
    sp = c.find("span")
    if not sp:
        continue
    st = sp.get("style")
    if not st:
        continue
    fs = re.findall("font-size:(\d+)px", st)
    if not fs:
        continue
    fs = int(fs[0])
    if not cur_fs:
        cur_fs = fs
    if fs == cur_fs:
        cur_text += c.text
    else:
        snippets.append((cur_text, cur_fs))
        cur_fs = fs
        cur_text = c.text
snippets.append((cur_text, cur_fs))
# 注意：上述逻辑非常直接。人们还可以添加更多策略，例如删除重复片段（因为 PDF 中的页眉/页脚会出现在多个页面上，因此如果我们发现重复内容，可以安全地假定它是多余信息）
```
```python
from langchain_core.documents import Document
cur_idx = -1
semantic_snippets = []
# 假设：标题的字体大小高于其相应内容
for s in snippets:
    # 如果当前片段的字体大小 > 前一个部分的标题 => 这是一个新标题
    if (
        not semantic_snippets
        or s[1] > semantic_snippets[cur_idx].metadata["heading_font"]
    ):
        metadata = {"heading": s[0], "content_font": 0, "heading_font": s[1]}
        metadata.update(data.metadata)
        semantic_snippets.append(Document(page_content="", metadata=metadata))
        cur_idx += 1
        continue
    # 如果当前片段的字体大小 <= 前一个部分的内容 => 内容属于同一部分（如果需要，人们还可以为子部分创建类似树状结构，但这可能需要更多思考，可能是特定于数据的）
    if (
        not semantic_snippets[cur_idx].metadata["content_font"]
        or s[1] <= semantic_snippets[cur_idx].metadata["content_font"]
    ):
        semantic_snippets[cur_idx].page_content += s[0]
        semantic_snippets[cur_idx].metadata["content_font"] = max(
            s[1], semantic_snippets[cur_idx].metadata["content_font"]
        )
        continue
    # 如果当前片段的字体大小 > 前一个部分的内容但小于前一个部分的标题，则也创建一个新部分
    metadata = {"heading": s[0], "content_font": 0, "heading_font": s[1]}
    metadata.update(data.metadata)
    semantic_snippets.append(Document(page_content="", metadata=metadata))
    cur_idx += 1
```
最近，针对布局分析任务，已经开发出了各种深度学习模型和数据集。dhSegment [22] 使用完全卷积网络 [20] 对历史文档进行分割任务。基于目标检测的方法，如 Faster R-CNN [28] 和 Mask R-CNN [12]，用于识别文档元素 [38] 和检测表格 [30, 26]。最近，图神经网络 [29] 也被用于表格检测 [27]。然而，这些模型通常是单独实现的，没有统一的框架来加载和使用这些模型。
近年来，创建开源工具来处理文档图像的兴趣激增：在 Github 上搜索文档图像分析可以找到 500 万个相关代码片段 [6]；然而，其中大多数依赖于传统的基于规则的方法或提供有限的功能。与我们的工作最接近的先前研究是 OCR-D 项目 [7]，它也试图构建一个完整的 DIA 工具包。然而，与 Neudecker 等人开发的平台 [21] 类似，它是为分析历史文档而设计的，并且不支持最近的深度学习模型。DocumentLayoutAnalysis 项目 [8] 专注于通过分析存储的 PDF 数据处理出生数字 PDF 文档。DeepLayout [9] 和 Detectron2-PubLayNet [10] 等存储库是在布局分析数据集上训练的独立深度学习模型，不支持完整的 DIA 流程。Document Analysis and Exploitation (DAE) 平台 [15] 和 DeepDIVA 项目 [2] 旨在提高 DIA 方法（或深度学习模型）的可重复性，但它们没有得到积极的维护。OCR 引擎如 Tesseract [14]、easyOCR [11] 和 paddleOCR [12] 通常不具备其他 DIA 任务（如布局分析）的全面功能。
近年来，为促进深度学习领域的可重复性和可重用性，进行了许多努力。像 Dectectron2 [35]、AllenNLP [8] 和 transformers [34] 这样的库为开发和部署通用计算机视觉和自然语言处理问题的模型提供了完整的深度学习支持。而 LayoutParser 则专门针对 DIA 任务进行了优化。LayoutParser 还配备了一个社区平台，受到 Torch Hub [23] 和 TensorFlow Hub [1] 等已建立的模型中心的启发。它可以共享预训练模型以及专门用于 DIA 任务的完整文档处理流程。
为了促进深度学习模型的开发，已经创建了各种文档数据集。一些例子包括 PRImA [3]（杂志布局）、PubLayNet [38]（学术论文布局）、Table Bank [18]（学术论文中的表格）、Newspaper Navigator Dataset [16, 17]（报纸图表布局）和 HJDataset [31]（历史日本文档布局）。在 LayoutParser 模型库中，训练在这些数据集上的各种模型可支持不同的用例。
AmazonTextractPDFLoader 调用 [Amazon Textract 服务](https://aws.amazon.com/textract/) 将 PDF 转换为 Document 结构。该加载器目前仅进行纯 OCR，根据需求计划增加更多的布局支持等功能。支持单页和多页文档，最多支持 3000 页和 512 MB 的大小。
为了成功调用，需要一个 AWS 账户，类似于 [AWS CLI](https://docs.aws.amazon.com/cli/latest/userguide/cli-chap-configure.html) 的要求。
除了 AWS 配置外，它与其他 PDF 加载器非常相似，同时还支持 JPEG、PNG 和 TIFF 等非原生 PDF 格式。
```python
from langchain_community.document_loaders import AmazonTextractPDFLoader
loader = AmazonTextractPDFLoader("example_data/alejandro_rosalez_sample-small.jpeg")
documents = loader.load()
```
## 使用 AzureAIDocumentIntelligenceLoader
[Azure AI Document Intelligence](https://aka.ms/doc-intelligence)（以前称为 `Azure Form Recognizer`）是一种基于机器学习的服务，可以从数字或扫描的 PDF、图像、Office 和 HTML 文件中提取文本（包括手写文字）、表格、文档结构（例如标题、章节标题等）和键值对。Document Intelligence 支持 `PDF`、`JPEG/JPG`、`PNG`、`BMP`、`TIFF`、`HEIF`、`DOCX`、`XLSX`、`PPTX` 和 `HTML`。
这个[当前实现](https://aka.ms/di-langchain)的加载器使用 `Document Intelligence` 可以将内容逐页合并并转换为 LangChain 文档。默认的输出格式是 markdown，可以很容易地与 `MarkdownHeaderTextSplitter` 链接在一起，以进行语义化的文档分块。您还可以使用 `mode="single"` 或 `mode="page"` 来返回单个页面或按页面拆分的纯文本文档。
### 先决条件
在以下 3 个预览区域之一拥有 Azure AI Document Intelligence 资源：**East US**、**West US2**、**West Europe** - 如果没有，请按照[此文档](https://learn.microsoft.com/azure/ai-services/document-intelligence/create-document-intelligence-resource?view=doc-intel-4.0.0)创建一个。您将把 `<endpoint>` 和 `<key>` 作为参数传递给加载器。
```python
%pip install --upgrade --quiet  langchain langchain-community azure-ai-documentintelligence
```
```python
from langchain_community.document_loaders import AzureAIDocumentIntelligenceLoader
file_path = "<filepath>"
endpoint = "<endpoint>"
key = "<key>"
loader = AzureAIDocumentIntelligenceLoader(
    api_endpoint=endpoint, api_key=key, file_path=file_path, api_model="prebuilt-layout"
)
documents = loader.load()
```
