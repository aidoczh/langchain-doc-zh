# 语雀

>[语雀](https://www.yuque.com/) 是一个专业的基于云的知识库，用于团队协作文档。

本手册介绍了如何从 `语雀` 加载文档。

您可以通过点击[个人设置](https://www.yuque.com/settings/tokens)页面中的个人头像来获取个人访问令牌。

```python
from langchain_community.document_loaders import YuqueLoader
```

```python
loader = YuqueLoader(access_token="<your_personal_access_token>")
```

```python
docs = loader.load()
```