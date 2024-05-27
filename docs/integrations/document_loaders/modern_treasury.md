# 现代财务

>[现代财务](https://www.moderntreasury.com/)简化了复杂的支付操作。它是一个统一的平台，用于支持移动资金的产品和流程。

>- 连接银行和支付系统

>- 实时跟踪交易和余额

>- 自动化规模化的支付操作

本文介绍了如何从“现代财务 REST API”中加载数据，并将其转换为可以导入到LangChain中的格式，以及向量化的示例用法。

```python
from langchain.indexes import VectorstoreIndexCreator
from langchain_community.document_loaders import ModernTreasuryLoader
```

现代财务 API 需要组织 ID 和 API 密钥，可以在现代财务仪表板的开发者设置中找到。

此文档加载器还需要一个 `resource` 选项，用于定义要加载的数据。

以下资源可用：

`payment_orders` [文档](https://docs.moderntreasury.com/reference/payment-order-object)

`expected_payments` [文档](https://docs.moderntreasury.com/reference/expected-payment-object)

`returns` [文档](https://docs.moderntreasury.com/reference/return-object)

`incoming_payment_details` [文档](https://docs.moderntreasury.com/reference/incoming-payment-detail-object)

`counterparties` [文档](https://docs.moderntreasury.com/reference/counterparty-object)

`internal_accounts` [文档](https://docs.moderntreasury.com/reference/internal-account-object)

`external_accounts` [文档](https://docs.moderntreasury.com/reference/external-account-object)

`transactions` [文档](https://docs.moderntreasury.com/reference/transaction-object)

`ledgers` [文档](https://docs.moderntreasury.com/reference/ledger-object)

`ledger_accounts` [文档](https://docs.moderntreasury.com/reference/ledger-account-object)

`ledger_transactions` [文档](https://docs.moderntreasury.com/reference/ledger-transaction-object)

`events` [文档](https://docs.moderntreasury.com/reference/events)

`invoices` [文档](https://docs.moderntreasury.com/reference/invoices)

```python
modern_treasury_loader = ModernTreasuryLoader("payment_orders")
```

```python
# 从加载器创建一个向量存储检索器
# 更多详情请参见 https://python.langchain.com/en/latest/modules/data_connection/getting_started.html
index = VectorstoreIndexCreator().from_loaders([modern_treasury_loader])
modern_treasury_doc_retriever = index.vectorstore.as_retriever()
```