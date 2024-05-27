# Google Cloud Document AI

Google Cloud 的 Document AI 是一个文档理解平台，可以将文档中的非结构化数据转化为结构化数据，使其更易于理解、分析和使用。

了解更多：

- [Document AI 概述](https://cloud.google.com/document-ai/docs/overview)

- [Document AI 视频和实验室](https://cloud.google.com/document-ai/docs/videos)

- [试用一下！](https://cloud.google.com/document-ai/docs/drag-and-drop)

该模块包含了一个基于 Google Cloud 的 DocAI 的 `PDF` 解析器。

您需要安装两个库来使用此解析器：

```python
%pip install --upgrade --quiet  langchain-google-community[docai]
```

首先，您需要按照这里的说明设置一个 Google Cloud Storage (GCS) 存储桶，并创建自己的光学字符识别 (OCR) 处理器：https://cloud.google.com/document-ai/docs/create-processor

`GCS_OUTPUT_PATH` 应该是 GCS 上的一个文件夹路径（以 `gs://` 开头），`PROCESSOR_NAME` 应该类似于 `projects/PROJECT_NUMBER/locations/LOCATION/processors/PROCESSOR_ID` 或 `projects/PROJECT_NUMBER/locations/LOCATION/processors/PROCESSOR_ID/processorVersions/PROCESSOR_VERSION_ID`。您可以通过编程方式获取它，或者从 Google Cloud 控制台的 `Processor details` 标签页的 `Prediction endpoint` 部分复制获取。

```python
GCS_OUTPUT_PATH = "gs://BUCKET_NAME/FOLDER_PATH"
PROCESSOR_NAME = "projects/PROJECT_NUMBER/locations/LOCATION/processors/PROCESSOR_ID"
```

```python
from langchain_core.document_loaders.blob_loaders import Blob
from langchain_google_community import DocAIParser
```

现在，创建一个 `DocAIParser`。

```python
parser = DocAIParser(
    location="us", processor_name=PROCESSOR_NAME, gcs_output_path=GCS_OUTPUT_PATH
)
```

对于这个示例，您可以使用上传到公共 GCS 存储桶的 Alphabet 收益报告。

[2022Q1_alphabet_earnings_release.pdf](https://storage.googleapis.com/cloud-samples-data/gen-app-builder/search/alphabet-investor-pdfs/2022Q1_alphabet_earnings_release.pdf)

将文档传递给 `lazy_parse()` 方法进行解析。

```python
blob = Blob(
    path="gs://cloud-samples-data/gen-app-builder/search/alphabet-investor-pdfs/2022Q1_alphabet_earnings_release.pdf"
)
```

我们将得到每页一个文档，总共有 11 个：

```python
docs = list(parser.lazy_parse(blob))
print(len(docs))
```

```output
11
```

您可以逐个运行一个 blob 的端到端解析。如果您有很多文档，将它们一起批处理可能是一个更好的方法，甚至可以将解析与处理解析结果分离。

```python
operations = parser.docai_parse([blob])
print([op.operation.name for op in operations])
```

```output
['projects/543079149601/locations/us/operations/16447136779727347991']
```

您可以检查操作是否已完成：

```python
parser.is_running(operations)
```

```output
True
```

当操作完成时，您可以解析结果：

```python
parser.is_running(operations)
```

```output
False
```

```python
results = parser.get_results(operations)
print(results[0])
```

```output
DocAIParsingResults(source_path='gs://vertex-pgt/examples/goog-exhibit-99-1-q1-2023-19.pdf', parsed_path='gs://vertex-pgt/test/run1/16447136779727347991/0')
```

现在，我们终于可以从解析结果生成文档了：

```python
docs = list(parser.parse_from_results(results))
```

```python
print(len(docs))
```

```output
11
```