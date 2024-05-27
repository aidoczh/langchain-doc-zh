# Cube 语义层

本笔记本演示了检索 Cube 数据模型元数据的过程，以适合作为嵌入传递给 LLMs，从而增强上下文信息。

### 关于 Cube

[Cube](https://cube.dev/) 是构建数据应用程序的语义层。它帮助数据工程师和应用程序开发人员从现代数据存储中访问数据，将其组织成一致的定义，并将其传递给每个应用程序。

Cube 的数据模型提供了结构和定义，用作 LLM 理解数据并生成正确查询的上下文。LLM 不需要浏览复杂的连接和度量计算，因为 Cube 对其进行了抽象，并提供了一个简单的界面，该界面操作的是业务级术语，而不是 SQL 表和列名称。这种简化有助于使 LLM 更不容易出错并避免产生幻觉。

### 示例

**输入参数（必填）**

`Cube 语义加载器` 需要 2 个参数：

- `cube_api_url`：Cube 部署的 REST API 的 URL。请参阅 [Cube 文档](https://cube.dev/docs/http-api/rest#configuration-base-path) 以获取有关配置基本路径的更多信息。

- `cube_api_token`：基于 Cube API 密钥生成的身份验证令牌。请参阅 [Cube 文档](https://cube.dev/docs/security#generating-json-web-tokens-jwt) 以获取有关生成 JSON Web 令牌（JWT）的说明。

**输入参数（可选）**

- `load_dimension_values`：是否加载每个字符串维度的维度值。

- `dimension_values_limit`：要加载的维度值的最大数量。

- `dimension_values_max_retries`：加载维度值的最大重试次数。

- `dimension_values_retry_delay`：加载维度值的重试之间的延迟。

```python
import jwt
from langchain_community.document_loaders import CubeSemanticLoader
api_url = "https://api-example.gcp-us-central1.cubecloudapp.dev/cubejs-api/v1/meta"
cubejs_api_secret = "api-secret-here"
security_context = {}
# 有关安全上下文的更多信息，请阅读：https://cube.dev/docs/security
api_token = jwt.encode(security_context, cubejs_api_secret, algorithm="HS256")
loader = CubeSemanticLoader(api_url, api_token)
documents = loader.load()
```

返回具有以下属性的文档列表：

- `page_content`

- `metadata`

  - `table_name`

  - `column_name`

  - `column_data_type`

  - `column_title`

  - `column_description`

  - `column_values`

  - `cube_data_obj_type`

```python
# 包含页面内容的字符串
page_content = "Users View City, None"
# 包含元数据的字典
metadata = {
    "table_name": "users_view",
    "column_name": "users_view.city",
    "column_data_type": "string",
    "column_title": "Users View City",
    "column_description": "None",
    "column_member_type": "dimension",
    "column_values": [
        "Austin",
        "Chicago",
        "Los Angeles",
        "Mountain View",
        "New York",
        "Palo Alto",
        "San Francisco",
        "Seattle",
    ],
    "cube_data_obj_type": "view",
}
```