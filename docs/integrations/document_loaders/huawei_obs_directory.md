# 华为 OBS 目录

以下代码演示了如何从华为 OBS（对象存储服务）中加载对象作为文档。

```python
# 安装所需的包
# pip install esdk-obs-python
```

```python
from langchain_community.document_loaders import OBSDirectoryLoader
```

```python
endpoint = "your-endpoint"
```

```python
# 配置访问凭证
config = {"ak": "your-access-key", "sk": "your-secret-key"}
loader = OBSDirectoryLoader("your-bucket-name", endpoint=endpoint, config=config)
```

```python
loader.load()
```

## 指定加载的前缀

如果您想从存储桶中加载具有特定前缀的对象，可以使用以下代码：

```python
loader = OBSDirectoryLoader(
    "your-bucket-name", endpoint=endpoint, config=config, prefix="test_prefix"
)
```

```python
loader.load()
```

## 从 ECS 获取认证信息

如果您的 langchain 部署在华为云 ECS 上，并且[已设置代理](https://support.huaweicloud.com/intl/en-us/usermanual-ecs/ecs_03_0166.html#section7)，加载程序可以直接从 ECS 获取安全令牌，而无需访问密钥和秘密密钥。

```python
config = {"get_token_from_ecs": True}
loader = OBSDirectoryLoader("your-bucket-name", endpoint=endpoint, config=config)
```

```python
loader.load()
```

## 使用公共存储桶

如果您的存储桶的存储桶策略允许匿名访问（匿名用户具有`listBucket`和`GetObject`权限），您可以直接加载对象，而无需配置`config`参数。

```python
loader = OBSDirectoryLoader("your-bucket-name", endpoint=endpoint)
```

```python
loader.load()
```