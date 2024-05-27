# JSON

这篇笔记展示了一个与大型 `JSON/dict` 对象交互的代理程序。当您想要回答关于一个 JSON 数据块的问题，而该数据块太大而无法适应 LLM 上下文窗口时，这将非常有用。该代理程序能够迭代地探索数据块，找到答案用户问题所需的信息。

在下面的示例中，我们使用了 OpenAI API 的 OpenAPI 规范，您可以在[这里](https://github.com/openai/openai-openapi/blob/master/openapi.yaml)找到。

我们将使用 JSON 代理程序来回答一些关于 API 规范的问题。

## 初始化

```python
import yaml
from langchain_community.agent_toolkits import JsonToolkit, create_json_agent
from langchain_community.tools.json.tool import JsonSpec
from langchain_openai import OpenAI
```

```python
with open("openai_openapi.yml") as f:
    data = yaml.load(f, Loader=yaml.FullLoader)
json_spec = JsonSpec(dict_=data, max_value_length=4000)
json_toolkit = JsonToolkit(spec=json_spec)
json_agent_executor = create_json_agent(
    llm=OpenAI(temperature=0), toolkit=json_toolkit, verbose=True
)
```

## 示例：获取请求中到达 /completions 端点所需的 POST 参数

```python
json_agent_executor.run(
    "What are the required parameters in the request body to the /completions endpoint?"
)
```

```output
> 进入新的代理执行链...
动作: json_spec_list_keys
动作输入: data
观察: ['openapi', 'info', 'servers', 'tags', 'paths', 'components', 'x-oaiMeta']
思考: 我应该查看 paths 键，看看存在哪些端点
动作: json_spec_list_keys
动作输入: data["paths"]
观察: ['/engines', '/engines/{engine_id}', '/completions', '/edits', '/images/generations', '/images/edits', '/images/variations', '/embeddings', '/engines/{engine_id}/search', '/files', '/files/{file_id}', '/files/{file_id}/content', '/answers', '/classifications', '/fine-tunes', '/fine-tunes/{fine_tune_id}', '/fine-tunes/{fine_tune_id}/cancel', '/fine-tunes/{fine_tune_id}/events', '/models', '/models/{model}', '/moderations']
思考: 我应该查看 /completions 端点，看看需要哪些参数
动作: json_spec_list_keys
动作输入: data["paths"]["/completions"]
观察: ['post']
思考: 我应该查看 post 键，看看需要哪些参数
动作: json_spec_list_keys
动作输入: data["paths"]["/completions"]["post"]
观察: ['operationId', 'tags', 'summary', 'requestBody', 'responses', 'x-oaiMeta']
思考: 我应该查看 requestBody 键，看看需要哪些参数
动作: json_spec_list_keys
动作输入: data["paths"]["/completions"]["post"]["requestBody"]
观察: ['required', 'content']
思考: 我应该查看 required 键，看看需要哪些参数
动作: json_spec_get_value
动作输入: data["paths"]["/completions"]["post"]["requestBody"]["required"]
观察: True
思考: 我应该查看 content 键，看看需要哪些参数
动作: json_spec_list_keys
动作输入: data["paths"]["/completions"]["post"]["requestBody"]["content"]
观察: ['application/json']
思考: 我应该查看 application/json 键，看看需要哪些参数
动作: json_spec_list_keys
动作输入: data["paths"]["/completions"]["post"]["requestBody"]["content"]["application/json"]
观察: ['schema']
思考: 我应该查看 schema 键，看看需要哪些参数
动作: json_spec_list_keys
动作输入: data["paths"]["/completions"]["post"]["requestBody"]["content"]["application/json"]["schema"]
观察: ['$ref']
思考: 我应该查看 $ref 键，看看需要哪些参数
动作: json_spec_get_value
动作输入: data["paths"]["/completions"]["post"]["requestBody"]["content"]["application/json"]["schema"]["$ref"]
观察: #/components/schemas/CreateCompletionRequest
思考: 我应该查看 CreateCompletionRequest 模式，看看需要哪些参数
动作: json_spec_list_keys
动作输入: data["components"]["schemas"]["CreateCompletionRequest"]
观察: ['type', 'properties', 'required']
思考: 我应该查看 required 键，看看需要哪些参数
动作: json_spec_get_value
动作输入: data["components"]["schemas"]["CreateCompletionRequest"]["required"]
观察: ['model']
思考: 我现在知道最终答案
最终答案: 到达 /completions 端点时请求体中所需的参数是 'model'。
> 链结束。
```

```output
"到达 /completions 端点时请求体中所需的参数是 'model'。"
```