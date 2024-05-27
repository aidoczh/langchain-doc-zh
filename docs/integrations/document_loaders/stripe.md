# Stripe

[Stripe](https://stripe.com/en-ca) 是一家爱尔兰-美国的金融服务和软件即服务（SaaS）公司。它提供支付处理软件和应用程序编程接口，用于电子商务网站和移动应用程序。

本文介绍如何从 `Stripe REST API` 中加载数据，将其转换为可被 LangChain 吸收的格式，并提供了矢量化的示例用法。

```python
from langchain.indexes import VectorstoreIndexCreator
from langchain_community.document_loaders import StripeLoader
```

Stripe API 需要访问令牌，可以在 Stripe 仪表板中找到。

此文档加载器还需要一个 `resource` 选项，用于定义要加载的数据。

以下资源可用：

`balance_transations` [文档](https://stripe.com/docs/api/balance_transactions/list)

`charges` [文档](https://stripe.com/docs/api/charges/list)

`customers` [文档](https://stripe.com/docs/api/customers/list)

`events` [文档](https://stripe.com/docs/api/events/list)

`refunds` [文档](https://stripe.com/docs/api/refunds/list)

`disputes` [文档](https://stripe.com/docs/api/disputes/list)

```python
stripe_loader = StripeLoader("charges")
```

```python
# 从加载器创建一个 vectorstore 检索器
# 有关详细信息，请参阅 https://python.langchain.com/en/latest/modules/data_connection/getting_started.html
index = VectorstoreIndexCreator().from_loaders([stripe_loader])
stripe_doc_retriever = index.vectorstore.as_retriever()
```