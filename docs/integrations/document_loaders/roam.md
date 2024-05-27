# Roam

>[ROAM](https://roamresearch.com/) 是一个用于网络化思维的笔记工具，旨在创建个人知识库。

本笔记本介绍了如何从 Roam 数据库加载文档。这在很大程度上受到了这个示例存储库 [here](https://github.com/JimmyLv/roam-qa) 的启发。

## 🧑 自定义数据集加载说明

从 Roam Research 导出您的数据集。您可以通过单击右上角的三个点，然后单击 `Export` 来执行此操作。

在导出时，请确保选择 `Markdown & CSV` 格式选项。

这将在您的下载文件夹中生成一个 `.zip` 文件。将 `.zip` 文件移动到此存储库中。

运行以下命令以解压缩 zip 文件（根据需要将 `Export...` 替换为您自己的文件名）。

```shell
unzip Roam-Export-1675782732639.zip -d Roam_DB
```

```python
from langchain_community.document_loaders import RoamLoader
```

```python
loader = RoamLoader("Roam_DB")
```

```python
docs = loader.load()
```