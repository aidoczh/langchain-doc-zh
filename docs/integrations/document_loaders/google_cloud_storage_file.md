# 谷歌云存储文件

[谷歌云存储](https://en.wikipedia.org/wiki/Google_Cloud_Storage) 是一个用于存储非结构化数据的托管服务。

这里介绍了如何从谷歌云存储（GCS）文件对象（blob）中加载文档对象。

```python
%pip install --upgrade --quiet  langchain-google-community[gcs]
```

```python
from langchain_google_community import GCSFileLoader
```

```python
loader = GCSFileLoader(project_name="aist", bucket="testing-hwc", blob="fake.docx")
```

```python
loader.load()
```

```output
/Users/harrisonchase/workplace/langchain/.venv/lib/python3.10/site-packages/google/auth/_default.py:83: UserWarning: Your application has authenticated using end user credentials from Google Cloud SDK without a quota project. You might receive a "quota exceeded" or "API not enabled" error. We recommend you rerun `gcloud auth application-default login` and make sure a quota project is added. Or you can use service accounts instead. For more information about service accounts, see https://cloud.google.com/docs/authentication/
  warnings.warn(_CLOUD_SDK_CREDENTIALS_WARNING)
```

```output
[Document(page_content='Lorem ipsum dolor sit amet.', lookup_str='', metadata={'source': '/var/folders/y6/8_bzdg295ld6s1_97_12m4lr0000gn/T/tmp3srlf8n8/fake.docx'}, lookup_index=0)]
```

如果你想使用另一种加载器，你可以提供一个自定义函数，例如：

```python
from langchain_community.document_loaders import PyPDFLoader
def load_pdf(file_path):
    return PyPDFLoader(file_path)
loader = GCSFileLoader(
    project_name="aist", bucket="testing-hwc", blob="fake.pdf", loader_func=load_pdf
)
```