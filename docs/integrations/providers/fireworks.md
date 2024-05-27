# 烟花

本页面介绍如何在 Langchain 中使用 [Fireworks](https://fireworks.ai/) 模型。

## 安装和设置

- 安装 Fireworks 集成包。

  ```
  pip install langchain-fireworks
  ```

- 在 [fireworks.ai](https://fireworks.ai) 注册并获取 Fireworks API 密钥。

- 通过设置 FIREWORKS_API_KEY 环境变量进行身份验证。

## 身份验证

有两种使用 Fireworks API 密钥进行身份验证的方式：

1. 设置 `FIREWORKS_API_KEY` 环境变量。

    ```python
    os.environ["FIREWORKS_API_KEY"] = "<KEY>"
    ```

2. 在 Fireworks LLM 模块中设置 `api_key` 字段。

    ```python
    llm = Fireworks(api_key="<KEY>")
    ```

## 使用 Fireworks LLM 模块

Fireworks 通过 LLM 模块与 Langchain 进行集成。在此示例中，我们将使用 mixtral-8x7b-instruct 模型。

```python
from langchain_fireworks import Fireworks 
llm = Fireworks(
    api_key="<KEY>",
    model="accounts/fireworks/models/mixtral-8x7b-instruct",
    max_tokens=256)
llm("Name 3 sports.")
```

有关更详细的操作步骤，请参阅[此处](/docs/integrations/llms/Fireworks)。