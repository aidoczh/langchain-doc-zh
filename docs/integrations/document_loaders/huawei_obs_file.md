# 华为 OBS 文件

以下代码演示了如何从华为 OBS（对象存储服务）加载对象作为文档。

```python
# 安装所需的包
# pip install esdk-obs-python
```

```python
from langchain_community.document_loaders.obs_file import OBSFileLoader
```

```python
endpoint = "your-endpoint"
```

```python
from obs import ObsClient
obs_client = ObsClient(
    access_key_id="your-access-key",
    secret_access_key="your-secret-key",
    server=endpoint,
)
loader = OBSFileLoader("your-bucket-name", "your-object-key", client=obs_client)
```

```python
loader.load()
```

## 每个加载器都有单独的身份验证信息

如果您不需要在不同的加载器之间重用 OBS 连接，可以直接配置 `config`。加载器将使用配置信息初始化自己的 OBS 客户端。

```python
# 配置您的访问凭证
config = {"ak": "your-access-key", "sk": "your-secret-key"}
loader = OBSFileLoader(
    "your-bucket-name", "your-object-key", endpoint=endpoint, config=config
)
```

```python
loader.load()
```

## 从 ECS 获取身份验证信息

如果您的 langchain 部署在华为云 ECS 上，并且已经设置了 [代理](https://support.huaweicloud.com/intl/en-us/usermanual-ecs/ecs_03_0166.html#section7)，加载器可以直接从 ECS 获取安全令牌，而无需访问密钥和秘密密钥。

```python
config = {"get_token_from_ecs": True}
loader = OBSFileLoader(
    "your-bucket-name", "your-object-key", endpoint=endpoint, config=config
)
```

```python
loader.load()
```

## 访问公开可访问的对象

如果您要访问的对象允许匿名用户访问（匿名用户具有 `GetObject` 权限），则可以直接加载对象，而无需配置 `config` 参数。

```python
loader = OBSFileLoader("your-bucket-name", "your-object-key", endpoint=endpoint)
```

```python
loader.load()
```