# 腾讯云对象存储（COS）

[腾讯云对象存储（COS）](https://www.tencentcloud.com/products/cos)是一项分布式存储服务，可以通过HTTP/HTTPS协议从任何地方存储任意数量的数据。`COS`对数据结构或格式没有限制。它还没有存储桶大小限制和分区管理，适用于几乎任何用例，如数据传输、数据处理和数据湖。`COS`提供基于Web的控制台、多语言SDK和API、命令行工具和图形工具。它与Amazon S3 API配合良好，允许您快速访问社区工具和插件。

这里介绍了如何从`腾讯COS文件`中加载文档对象。

```python
%pip install --upgrade --quiet  cos-python-sdk-v5
```

```python
from langchain_community.document_loaders import TencentCOSFileLoader
from qcloud_cos import CosConfig
```

```python
conf = CosConfig(
    Region="your cos region",
    SecretId="your cos secret_id",
    SecretKey="your cos secret_key",
)
loader = TencentCOSFileLoader(conf=conf, bucket="you_cos_bucket", key="fake.docx")
```

```python
loader.load()
```