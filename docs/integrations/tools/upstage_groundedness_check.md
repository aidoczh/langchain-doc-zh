---

sidebar_label: Upstage

---

# Upstage 立足性检查

本文档介绍了如何开始使用 Upstage 立足性检查模型。

## 安装  

安装 `langchain-upstage` 包。

```bash
pip install -U langchain-upstage
```

## 环境设置

确保设置以下环境变量：

- `UPSTAGE_API_KEY`: 从[Upstage开发者文档](https://developers.upstage.ai/docs/getting-started/quick-start)中获取的 Upstage API 密钥。

```python
import os
os.environ["UPSTAGE_API_KEY"] = "YOUR_API_KEY"
```

## 使用

初始化 `UpstageGroundednessCheck` 类。

```python
from langchain_upstage import UpstageGroundednessCheck
groundedness_check = UpstageGroundednessCheck()
```

使用 `run` 方法检查输入文本的立足性。

```python
request_input = {
    "context": "Mauna Kea is an inactive volcano on the island of Hawai'i. Its peak is 4,207.3 m above sea level, making it the highest point in Hawaii and second-highest peak of an island on Earth.",
    "answer": "Mauna Kea is 5,207.3 meters tall.",
}
response = groundedness_check.invoke(request_input)
print(response)
```
