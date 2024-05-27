# Airtable

[Airtable](https://en.wikipedia.org/wiki/Airtable) 是一项云协作服务。`Airtable` 是一种电子表格数据库混合体，具有数据库的特性，但应用于电子表格。Airtable 表中的字段类似于电子表格中的单元格，但具有诸如“复选框”、“电话号码”和“下拉列表”之类的类型，并且可以引用文件附件，如图像。

用户可以创建数据库，设置列类型，添加记录，将表格相互链接，进行协作，对记录进行排序，并将视图发布到外部网站。

## 安装和设置

```bash
pip install pyairtable
```

* 获取您的 [API 密钥](https://support.airtable.com/docs/creating-and-using-api-keys-and-access-tokens)。

* 获取您的[基地 ID](https://airtable.com/developers/web/api/introduction)。

* 从表格网址中获取 [表格 ID](https://www.highviewapps.com/kb/where-can-i-find-the-airtable-base-id-and-table-id/#:~:text=Both%20the%20Airtable%20Base%20ID,URL%20that%20begins%20with%20tbl)。

## 文档加载器

```python
from langchain_community.document_loaders import AirtableLoader
```

查看一个[示例](/docs/integrations/document_loaders/airtable)。