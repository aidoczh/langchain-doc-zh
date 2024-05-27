# 腾讯云对象存储 (COS) 目录

[腾讯云对象存储 (COS)](https://www.tencentcloud.com/products/cos) 是一项分布式存储服务，可以通过 HTTP/HTTPS 协议从任何地方存储任意数量的数据。`COS` 对数据结构或格式没有限制。它还没有存储桶大小限制和分区管理，适用于几乎任何用例，如数据传输、数据处理和数据湖。`COS` 提供基于 Web 的控制台、多语言 SDK 和 API、命令行工具以及图形工具。它与 Amazon S3 API 配合良好，可以快速访问社区工具和插件。

这里介绍了如何从 `腾讯云对象存储 (COS)` 目录加载文档对象。

```python
%pip install --upgrade --quiet  cos-python-sdk-v5
```

```python
from langchain_community.document_loaders import TencentCOSDirectoryLoader
from qcloud_cos import CosConfig
```

```python
conf = CosConfig(
    Region="your cos region",
    SecretId="your cos secret_id",
    SecretKey="your cos secret_key",
)
loader = TencentCOSDirectoryLoader(conf=conf, bucket="you_cos_bucket")
```

```python
loader.load()
```

## 指定前缀

您还可以指定前缀，以更精细地控制要加载的文件。

```python
loader = TencentCOSDirectoryLoader(conf=conf, bucket="you_cos_bucket", prefix="fake")
```

```python
loader.load()
```