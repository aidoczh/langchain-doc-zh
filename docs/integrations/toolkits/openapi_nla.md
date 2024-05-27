

# 自然语言 API

`自然语言 API` 工具包 (`NLAToolkits`) 允许 LangChain 代理有效地计划和组合跨端点的调用。

这个笔记本演示了 `Speak`、`Klarna` 和 `Spoonacluar` API 的示例组合。

### 首先，导入依赖项并加载 LLM

```python
from langchain.agents import AgentType, initialize_agent
from langchain_community.agent_toolkits import NLAToolkit
from langchain_community.utilities import Requests
from langchain_openai import OpenAI
```

```python
# 选择要使用的 LLM。这里我们使用 gpt-3.5-turbo-instruct
llm = OpenAI(
    temperature=0, max_tokens=700, model_name="gpt-3.5-turbo-instruct"
)  # 您可以在这里在不同的核心 LLM 之间切换。
```

### 接下来，加载自然语言 API 工具包

```python
speak_toolkit = NLAToolkit.from_llm_and_url(llm, "https://api.speak.com/openapi.yaml")
klarna_toolkit = NLAToolkit.from_llm_and_url(
    llm, "https://www.klarna.com/us/shopping/public/openai/v0/api-docs/"
)
```

```output
尝试加载 OpenAPI 3.0.1 规范。这可能会导致性能下降。将您的 OpenAPI 规范转换为 3.1.* 规范以获得更好的支持。
尝试加载 OpenAPI 3.0.1 规范。这可能会导致性能下降。将您的 OpenAPI 规范转换为 3.1.* 规范以获得更好的支持。
尝试加载 OpenAPI 3.0.1 规范。这可能会导致性能下降。将您的 OpenAPI 规范转换为 3.1.* 规范以获得更好的支持。
```

### 创建代理

```python
# 稍微调整默认代理的指令
openapi_format_instructions = """使用以下格式：
问题：您必须回答的输入问题
思考：您应该始终考虑要做什么
动作：要采取的行动，应该是[{tool_names}]中的一个
动作输入：指导 AI 代表进行的操作。
观察：代理的响应
... (这个思考/动作/动作输入/观察可以重复 N 次)
思考：我现在知道最终答案。用户看不到我的任何观察、API 响应、链接或工具。
最终答案：原始输入问题的最终答案，具有正确的详细程度
在回复最终答案时，请记住，您正在回复的人看不到您的任何思考/动作/动作输入/观察，因此如果那里有任何相关信息，您需要在回复中明确包含它。"""
```

```python
natural_language_tools = speak_toolkit.get_tools() + klarna_toolkit.get_tools()
mrkl = initialize_agent(
    natural_language_tools,
    llm,
    agent=AgentType.ZERO_SHOT_REACT_DESCRIPTION,
    verbose=True,
    agent_kwargs={"format_instructions": openapi_format_instructions},
)
```

```python
mrkl.run(
    "我在意大利课程的年终派对，需要为此购买一些意大利服装"
)
```

```output
> 进入新的 AgentExecutor 链...
 我需要找出有哪些意大利服装可用
动作：Open_AI_Klarna_product_Api.productsUsingGET
动作输入：意大利服装
观察：API 响应包含两件来自 Alé 品牌的意大利蓝色产品。第一件是 Alé Colour Block 短袖运动衫 男款 - 意大利蓝色，售价 $86.49，第二件是 Alé Dolid 闪光运动衫 男款 - 意大利蓝色，售价 $40.00。
思考：我现在知道有哪些意大利服装可用以及它们的价格。
最终答案：您可以为您的年终派对购买两件来自 Alé 品牌的意大利蓝色产品。Alé Colour Block 短袖运动衫 男款 - 意大利蓝色售价 $86.49，Alé Dolid 闪光运动衫 男款 - 意大利蓝色售价 $40.00。
> 链条完成。
```

```output
'您可以为您的年终派对购买两件来自 Alé 品牌的意大利蓝色产品。Alé Colour Block 短袖运动衫 男款 - 意大利蓝色售价 $86.49，Alé Dolid 闪光运动衫 男款 - 意大利蓝色售价 $40.00。'
```

### 使用身份验证并添加更多端点

某些端点可能需要用户通过访问令牌等进行身份验证。这里我们展示如何通过 `Requests` 包装对象传递身份验证信息。

由于每个 NLATool 都向其封装的 API 公开了简洁的自然语言接口，顶级对话代理更容易地整合每个端点以满足用户的请求。

**添加 Spoonacular 端点。**

1. 转到 [Spoonacular API 控制台](https://spoonacular.com/food-api/console#Profile) 并创建一个免费帐户。

2. 点击 `Profile` 并在下面复制您的 API 密钥。

```python
spoonacular_api_key = ""  # 从 API 控制台复制
```

```python
requests = Requests(headers={"x-api-key": spoonacular_api_key})
spoonacular_toolkit = NLAToolkit.from_llm_and_url(
    llm,
    "https://spoonacular.com/application/frontend/downloads/spoonacular-openapi-3.json",
    requests=requests,
    max_text_length=1800,  # 如果您想要截断响应文本
)
```

```output
尝试加载 OpenAPI 3.0.0 规范。这可能会导致性能下降。将您的 OpenAPI 规范转换为 3.1.* 规范以获得更好的支持。
不支持参数 Content-Type 的 APIPropertyLocation "header"。有效值为 ['path', 'query']。忽略可选参数
不支持参数 Accept 的 APIPropertyLocation "header"。有效值为 ['path', 'query']。忽略可选参数
不支持参数 Content-Type 的 APIPropertyLocation "header"。有效值为 ['path', 'query']。忽略可选参数
不支持参数 Accept 的 APIPropertyLocation "header"。有效值为 ['path', 'query']。忽略可选参数
不支持参数 Content-Type 的 APIPropertyLocation "header"。有效值为 ['path', 'query']。忽略可选参数
不支持参数 Accept 的 APIPropertyLocation "header"。有效值为 ['path', 'query']。忽略可选参数
不支持参数 Content-Type 的 APIPropertyLocation "header"。有效值为 ['path', 'query']。忽略可选参数
不支持参数 Accept 的 APIPropertyLocation "header"。有效值为 ['path', 'query']。忽略可选参数
不支持参数 Content-Type 的 APIPropertyLocation "header"。有效值为 ['path', 'query']。忽略可选参数
不支持参数 Content-Type 的 APIPropertyLocation "header"。有效值为 ['path', 'query']。忽略可选参数
不支持参数 Content-Type 的 APIPropertyLocation "header"。有效值为 ['path', 'query']。忽略可选参数
不支持参数 Content-Type 的 APIPropertyLocation "header"。有效值为 ['path', 'query']。忽略可选参数
不支持参数 Accept 的 APIPropertyLocation "header"。有效值为 ['path', 'query']。忽略可选参数
不支持参数 Content-Type 的 APIPropertyLocation "header"。有效值为 ['path', 'query']。忽略可选参数
不支持参数 Accept 的 APIPropertyLocation "header"。有效值为 ['path', 'query']。忽略可选参数
不支持参数 Accept 的 APIPropertyLocation "header"。有效值为 ['path', 'query']。忽略可选参数
不支持参数 Accept 的 APIPropertyLocation "header"。有效值为 ['path', 'query']。忽略可选参数
不支持参数 Content-Type 的 APIPropertyLocation "header"。有效值为 ['path', 'query']。忽略可选参数
```

```python
natural_language_api_tools = (
    speak_toolkit.get_tools()
    + klarna_toolkit.get_tools()
    + spoonacular_toolkit.get_tools()[:30]
)
print(f"{len(natural_language_api_tools)} tools loaded.")
```

```output
34 tools loaded.
```

```python
# 使用新工具创建一个代理
mrkl = initialize_agent(
    natural_language_api_tools,
    llm,
    agent=AgentType.ZERO_SHOT_REACT_DESCRIPTION,
    verbose=True,
    agent_kwargs={"format_instructions": openapi_format_instructions},
)
```

```python
# 使查询更加复杂！
user_input = (
    "I'm learning Italian, and my language class is having an end of year party... "
    " Could you help me find an Italian outfit to wear and"
    " an appropriate recipe to prepare so I can present for the class in Italian?"
)
```

```python
mrkl.run(user_input)
```

```output
> 进入新的 AgentExecutor 链...
我需要找到一个意大利风格的食谱和服装。
动作：spoonacular_API.searchRecipes
动作输入：Italian
观察：API 响应包含 10 个意大利食谱，包括土耳其番茄奶酪比萨、西兰花藜麦菜肴、意式烤面包和猪肉意面、三文鱼藜麦烩饭、意大利金枪鱼意面、大蒜烤花椰菜、柠檬芦笋烩饭、意大利蒸朝鲜蓟、脆皮意大利花菜开胃菜和番茄面包汤。
想法：我需要找到一个意大利风格的服装。
动作：Open_AI_Klarna_product_Api.productsUsingGET
动作输入：Italian
观察：在 API 响应中找到了 10 个与“意大利”相关的产品。这些产品包括意大利金闪闪完美项链 - 金色、意大利设计迈阿密古巴链接链项链 - 金色、意大利金迈阿密古巴链接链项链 - 金色、意大利金鱼骨链项链 - 金色、意大利金克拉达戒指 - 金色、意大利金鱼骨链项链 - 金色、佳明 QuickFit 22mm 意大利 Vacchetta 皮带、梅西意大利号角吊坠 - 金色、Dolce & Gabbana 浅蓝色意大利之爱 Pour Homme EdT 1.7 盎司。
想法：我现在知道最终答案。
最终答案：为了在意大利语课上展示，你可以穿意大利金闪闪完美项链 - 金色、意大利设计迈阿密古巴链接链项链 - 金色或意大利金迈阿密古巴链接链项链 - 金色。至于食谱，你可以做土耳其番茄奶酪比萨、西兰花藜麦菜肴、意式烤面包和猪肉意面、三文鱼藜麦烩饭、意大利金枪鱼意面、大蒜烤花椰菜、柠檬芦笋烩饭、意大利蒸朝鲜蓟、脆皮意大利花菜开胃菜或番茄面包汤。
> 链结束。
```

```output
'为了在意大利语课上展示，你可以穿意大利金闪闪完美项链 - 金色、意大利设计迈阿密古巴链接链项链 - 金色或意大利金迈阿密古巴链接链项链 - 金色。至于食谱，你可以做土耳其番茄奶酪比萨、西兰花藜麦菜肴、意式烤面包和猪肉意面、三文鱼藜麦烩饭、意大利金枪鱼意面、大蒜烤花椰菜、柠檬芦笋烩饭、意大利蒸朝鲜蓟、脆皮意大利花菜开胃菜或番茄面包汤。'
```

```python
natural_language_api_tools[1].run(
    "Tell the LangChain audience to 'enjoy the meal' in Italian, please!"
)
```

```output
"在意大利语中，你可以对别人说 'Buon appetito' 来祝愿他们享用美食。这个短语在意大利很常用，当有人要开始用餐时经常使用。这类似于法语中的 'Bon appétit' 或德语中的 'Guten Appetit'。"
```