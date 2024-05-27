# 亚马逊文本提取

[亚马逊文本提取](https://docs.aws.amazon.com/managedservices/latest/userguide/textract.html)是一项机器学习（ML）服务，可以自动从扫描文档中提取文本、手写内容和数据。

它不仅可以进行简单的光学字符识别（OCR），还可以识别、理解并从表格和表单中提取数据。如今，许多公司都需要手动从扫描文档（如PDF、图像、表格和表单）中提取数据，或者使用简单的OCR软件进行手动配置（当表单发生变化时通常需要更新）。为了克服这些手动且昂贵的流程，`Textract`利用机器学习来读取和处理任何类型的文档，准确提取文本、手写内容、表格和其他数据，无需人工干预。

这个示例演示了如何将`Amazon Textract`与LangChain结合使用作为DocumentLoader。

`Textract`支持`PDF`、`TIFF`、`PNG`和`JPEG`格式。

`Textract`支持这些[文档大小、语言和字符](https://docs.aws.amazon.com/textract/latest/dg/limits-document.html)。

```python
%pip install --upgrade --quiet  boto3 langchain-openai tiktoken python-dotenv
```

```python
%pip install --upgrade --quiet  "amazon-textract-caller>=0.2.0"
```

## 示例1

第一个示例使用本地文件，该文件将被发送到亚马逊文本提取同步API [DetectDocumentText](https://docs.aws.amazon.com/textract/latest/dg/API_DetectDocumentText.html)。

本地文件或HTTP://等URL端点仅限于单页文档的Textract。

多页文档必须存储在S3上。此示例文件是一个jpeg。

```python
from langchain_community.document_loaders import AmazonTextractPDFLoader
loader = AmazonTextractPDFLoader("example_data/alejandro_rosalez_sample-small.jpeg")
documents = loader.load()
```

文件的输出

```python
documents
```

```output
[Document(page_content='Patient Information First Name: ALEJANDRO Last Name: ROSALEZ Date of Birth: 10/10/1982 Sex: M Marital Status: MARRIED Email Address: Address: 123 ANY STREET City: ANYTOWN State: CA Zip Code: 12345 Phone: 646-555-0111 Emergency Contact 1: First Name: CARLOS Last Name: SALAZAR Phone: 212-555-0150 Relationship to Patient: BROTHER Emergency Contact 2: First Name: JANE Last Name: DOE Phone: 650-555-0123 Relationship FRIEND to Patient: Did you feel fever or feverish lately? Yes No Are you having shortness of breath? Yes No Do you have a cough? Yes No Did you experience loss of taste or smell? Yes No Where you in contact with any confirmed COVID-19 positive patients? Yes No Did you travel in the past 14 days to any regions affected by COVID-19? Yes No Patient Information First Name: ALEJANDRO Last Name: ROSALEZ Date of Birth: 10/10/1982 Sex: M Marital Status: MARRIED Email Address: Address: 123 ANY STREET City: ANYTOWN State: CA Zip Code: 12345 Phone: 646-555-0111 Emergency Contact 1: First Name: CARLOS Last Name: SALAZAR Phone: 212-555-0150 Relationship to Patient: BROTHER Emergency Contact 2: First Name: JANE Last Name: DOE Phone: 650-555-0123 Relationship FRIEND to Patient: Did you feel fever or feverish lately? Yes No Are you having shortness of breath? Yes No Do you have a cough? Yes No Did you experience loss of taste or smell? Yes No Where you in contact with any confirmed COVID-19 positive patients? Yes No Did you travel in the past 14 days to any regions affected by COVID-19? Yes No ', metadata={'source': 'example_data/alejandro_rosalez_sample-small.jpeg', 'page': 1})]
```

## 示例2

下一个示例从HTTPS端点加载文件。

它必须是单页的，因为亚马逊文本提取要求所有多页文档都存储在S3上。

```python
from langchain_community.document_loaders import AmazonTextractPDFLoader
loader = AmazonTextractPDFLoader(
    "https://amazon-textract-public-content.s3.us-east-2.amazonaws.com/langchain/alejandro_rosalez_sample_1.jpg"
)
documents = loader.load()
```

```python
documents
```

```output
[Document(page_content='Patient Information First Name: ALEJANDRO Last Name: ROSALEZ Date of Birth: 10/10/1982 Sex: M Marital Status: MARRIED Email Address: Address: 123 ANY STREET City: ANYTOWN State: CA Zip Code: 12345 Phone: 646-555-0111 Emergency Contact 1: First Name: CARLOS Last Name: SALAZAR Phone: 212-555-0150 Relationship to Patient: BROTHER Emergency Contact 2: First Name: JANE Last Name: DOE Phone: 650-555-0123 Relationship FRIEND to Patient: Did you feel fever or feverish lately? Yes No Are you having shortness of breath? Yes No Do you have a cough? Yes No Did you experience loss of taste or smell? Yes No Where you in contact with any confirmed COVID-19 positive patients? Yes No Did you travel in the past 14 days to any regions affected by COVID-19? Yes No Patient Information First Name: ALEJANDRO Last Name: ROSALEZ Date of Birth: 10/10/1982 Sex: M Marital Status: MARRIED Email Address: Address: 123 ANY STREET City: ANYTOWN State: CA Zip Code: 12345 Phone: 646-555-0111 Emergency Contact 1: First Name: CARLOS Last Name: SALAZAR Phone: 212-555-0150 Relationship to Patient: BROTHER Emergency Contact 2: First Name: JANE Last Name: DOE Phone: 650-555-0123 Relationship FRIEND to Patient: Did you feel fever or feverish lately? Yes No Are you having shortness of breath? Yes No Do you have a cough? Yes No Did you experience loss of taste or smell? Yes No Where you in contact with any confirmed COVID-19 positive patients? Yes No Did you travel in the past 14 days to any regions affected by COVID-19? Yes No ', metadata={'source': 'example_data/alejandro_rosalez_sample-small.jpeg', 'page': 1})]
```

## 示例 3

处理多页文档需要文档存储在 S3 上。示例文档位于 us-east-2 区域的一个存储桶中，Textract 需要在同一区域调用才能成功，因此我们在客户端上设置了 region_name，并将其传递给加载器，以确保从 us-east-2 调用 Textract。您也可以在 us-east-2 区域运行您的笔记本，将 AWS_DEFAULT_REGION 设置为 us-east-2，或者在运行在不同环境时，像下面的单元格中那样传递一个带有该区域名称的 boto3 Textract 客户端。

```python
import boto3
textract_client = boto3.client("textract", region_name="us-east-2")
file_path = "s3://amazon-textract-public-content/langchain/layout-parser-paper.pdf"
loader = AmazonTextractPDFLoader(file_path, client=textract_client)
documents = loader.load()
```

现在获取页面数以验证响应（打印完整响应会相当长...）。我们期望有 16 页。

```python
len(documents)
```

```output
16
```

## 示例 4

您可以选择向 AmazonTextractPDFLoader 传递一个名为 `linearization_config` 的额外参数，该参数将确定在 Textract 运行后解析器如何对文本输出进行线性化。

```python
from langchain_community.document_loaders import AmazonTextractPDFLoader
from textractor.data.text_linearization_config import TextLinearizationConfig
loader = AmazonTextractPDFLoader(
    "s3://amazon-textract-public-content/langchain/layout-parser-paper.pdf",
    linearization_config=TextLinearizationConfig(
        hide_header_layout=True,
        hide_footer_layout=True,
        hide_figure_layout=True,
    ),
)
documents = loader.load()
```

## 在 LangChain 链中使用 AmazonTextractPDFLoader（例如 OpenAI）

AmazonTextractPDFLoader 可以像其他加载器一样在链中使用。

Textract 本身具有 [Query 功能](https://docs.aws.amazon.com/textract/latest/dg/API_Query.html)，提供了与此示例中的 QA 链类似的功能，也值得一试。

```python
# 您也可以将 OPENAI_API_KEY 存储在 .env 文件中
# import os
# from dotenv import load_dotenv
# load_dotenv()
```

```python
# 或者直接在环境中设置 OpenAI 密钥
import os
os.environ["OPENAI_API_KEY"] = "your-OpenAI-API-key"
```

```python
from langchain.chains.question_answering import load_qa_chain
from langchain_openai import OpenAI
chain = load_qa_chain(llm=OpenAI(), chain_type="map_reduce")
query = ["Who are the autors?"]
chain.run(input_documents=documents, question=query)
```

```output
' The authors are Zejiang Shen, Ruochen Zhang, Melissa Dell, Benjamin Charles Germain Lee, Jacob Carlson, Weining Li, Gardner, M., Grus, J., Neumann, M., Tafjord, O., Dasigi, P., Liu, N., Peters, M., Schmitz, M., Zettlemoyer, L., Lukasz Garncarek, Powalski, R., Stanislawek, T., Topolski, B., Halama, P., Gralinski, F., Graves, A., Fernández, S., Gomez, F., Schmidhuber, J., Harley, A.W., Ufkes, A., Derpanis, K.G., He, K., Gkioxari, G., Dollár, P., Girshick, R., He, K., Zhang, X., Ren, S., Sun, J., Kay, A., Lamiroy, B., Lopresti, D., Mears, J., Jakeway, E., Ferriter, M., Adams, C., Yarasavage, N., Thomas, D., Zwaard, K., Li, M., Cui, L., Huang,'
```