# ChatGPT 插件

这个例子展示了如何在 LangChain 抽象中使用 ChatGPT 插件。

注意1：目前这仅适用于没有身份验证的插件。

注意2：肯定还有其他方法可以做到这一点，这只是第一次尝试。如果你有更好的想法，请提交一个 PR！

```python
from langchain_community.tools import AIPluginTool
```

```python
from langchain.agents import AgentType, initialize_agent, load_tools
from langchain_openai import ChatOpenAI
```

```python
tool = AIPluginTool.from_plugin_url("https://www.klarna.com/.well-known/ai-plugin.json")
```

```python
llm = ChatOpenAI(temperature=0)
tools = load_tools(["requests_all"])
tools += [tool]
agent_chain = initialize_agent(
    tools, llm, agent=AgentType.ZERO_SHOT_REACT_DESCRIPTION, verbose=True
)
agent_chain.run("what t shirts are available in klarna?")
```

```output
> 进入新的 AgentExecutor 链...
我需要检查 Klarna 购物 API，看看它是否有关于可用 T 恤的信息。
动作：KlarnaProducts
动作输入：无
观察：使用指南：使用 Klarna 插件获取任何购物或研究目的的相关产品建议。发送的查询不应包括冠词、介词和限定词。当搜索与产品相关的词语时，如它们的名称、品牌、型号或类别时，API 的效果最佳。链接将始终返回，并应显示给用户。
OpenAPI 规范：{'openapi': '3.0.1', 'info': {'version': 'v0', 'title': 'Open AI Klarna product Api'}, 'servers': [{'url': 'https://www.klarna.com/us/shopping'}], 'tags': [{'name': 'open-ai-product-endpoint', 'description': 'Open AI Product Endpoint. Query for products.'}], 'paths': {'/public/openai/v0/products': {'get': {'tags': ['open-ai-product-endpoint'], 'summary': 'API for fetching Klarna product information', 'operationId': 'productsUsingGET', 'parameters': [{'name': 'q', 'in': 'query', 'description': 'query, must be between 2 and 100 characters', 'required': True, 'schema': {'type': 'string'}}, {'name': 'size', 'in': 'query', 'description': 'number of products returned', 'required': False, 'schema': {'type': 'integer'}}, {'name': 'budget', 'in': 'query', 'description': 'maximum price of the matching product in local currency, filters results', 'required': False, 'schema': {'type': 'integer'}}], 'responses': {'200': {'description': 'Products found', 'content': {'application/json': {'schema': {'$ref': '#/components/schemas/ProductResponse'}}}}, '503': {'description': 'one or more services are unavailable'}}, 'deprecated': False}}}, 'components': {'schemas': {'Product': {'type': 'object', 'properties': {'attributes': {'type': 'array', 'items': {'type': 'string'}}, 'name': {'type': 'string'}, 'price': {'type': 'string'}, 'url': {'type': 'string'}}, 'title': 'Product'}, 'ProductResponse': {'type': 'object', 'properties': {'products': {'type': 'array', 'items': {'$ref': '#/components/schemas/Product'}}}, 'title': 'ProductResponse'}}}}
思考：我需要使用 Klarna 购物 API 来搜索 T 恤。
动作：requests_get
动作输入：https://www.klarna.com/us/shopping/public/openai/v0/products?q=t%20shirts
观察：{"products":[{"name":"Lacoste Men's Pack of Plain T-Shirts","url":"https://www.klarna.com/us/shopping/pl/cl10001/3202043025/Clothing/Lacoste-Men-s-Pack-of-Plain-T-Shirts/?utm_source=openai","price":"$26.60","attributes":["Material:Cotton","Target Group:Man","Color:White,Black"]},{"name":"Hanes Men's Ultimate 6pk. Crewneck T-Shirts","url":"https://www.klarna.com/us/shopping/pl/cl10001/3201808270/Clothing/Hanes-Men-s-Ultimate-6pk.-Crewneck-T-Shirts/?utm_source=openai","price":"$13.82","attributes":["Material:Cotton","Target Group:Man","Color:White"]},{"name":"Nike Boy's Jordan Stretch T-shirts","url":"https://www.klarna.com/us/shopping/pl/cl359/3201863202/Children-s-Clothing/Nike-Boy-s-Jordan-Stretch-T-shirts/?utm_source=openai","price":"$14.99","attributes":["Material:Cotton","Color:White,Green","Model:Boy","Size (Small-Large):S,XL,L,M"]},{"name":"Polo Classic Fit Cotton V-Neck T-Shirts 3-Pack","url":"https://www.klarna.com/us/shopping/pl/cl10001/3203028500/Clothing/Polo-Classic-Fit-Cotton-V-Neck-T-Shirts-3-Pack/?utm_source=openai","price":"$29.95","attributes":["Material:Cotton","Target Group:Man","Color:White,Blue,Black"]},{"name":"adidas Comfort T-shirts Men's 3-pack","url":"https://www.klarna.com/us/shopping/pl/cl10001/3202640533/Clothing/adidas-Comfort-T-shirts-Men-s-3-pack/?utm_source=openai","price":"$14.99","attributes":["Material:Cotton","Target Group:Man","Color:White,Black","Neckline:Round"]}]}
思考：Klarna 上有以下可用的 T 恤：Lacoste 男士纯色 T 恤包、Hanes 男士终极 6 包圆领 T 恤、Nike 男孩乔丹弹力 T 恤、Polo 经典合身棉质 V 领 T 恤 3 包和 adidas 舒适 T 恤男士 3 包。
最终答案：Klarna 上有以下可用的 T 恤：Lacoste 男士纯色 T 恤包、Hanes 男士终极 6 包圆领 T 恤、Nike 男孩乔丹弹力 T 恤、Polo 经典合身棉质 V 领 T 恤 3 包和 adidas 舒适 T 恤男士 3 包。
> 链结束。
```

"Klarna 上有各种款式的 T 恤，包括 Lacoste 男士纯色 T 恤套装、Hanes 男士经典圆领 T 恤 6 包、Nike 男童乔丹弹力 T 恤、Polo 经典修身棉质 V 领 T 恤 3 包，以及 adidas 男士舒适款 T 恤 3 包。"