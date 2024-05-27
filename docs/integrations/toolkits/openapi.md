# OpenAPI

我们可以构建代理来消费任意的 API，包括符合 `OpenAPI`/`Swagger` 规范的 API。

```python
# 注意：在这个例子中，我们必须设置 `allow_dangerous_request=True` 来启用 OpenAPI 代理自动使用请求工具。
# 这可能会对调用不需要的请求造成危险。请确保您的自定义 OpenAPI 规范（yaml）是安全的。
ALLOW_DANGEROUS_REQUEST = True
```

## 第一个例子：分层规划代理

在这个例子中，我们将考虑一种称为分层规划的方法，这在机器人学中很常见，并且出现在最近关于 LLMs X 机器人学的作品中。我们将看到这是一种可行的方法，可以开始处理庞大的 API 规范，并且可以帮助用户查询需要对 API 进行多个步骤的操作。

这个想法很简单：为了在长序列行为中获得连贯的代理行为并节省令牌，我们将分开处理：一个“规划者”将负责调用哪些端点，而一个“控制器”将负责如何调用它们。

在最初的实现中，规划者是一个 LLM 链，它具有上下文中每个端点的名称和简短描述。控制器是一个只针对特定计划的端点实例化的 LLM 代理，其文档仅包含端点的文档。还有很多工作要做，才能使其非常稳健地工作 :)

---

### 首先，让我们收集一些 OpenAPI 规范。

```python
import os
import yaml
```

您可以从这里获取 OpenAPI 规范：[APIs-guru/openapi-directory](https://github.com/APIs-guru/openapi-directory)

```python
!wget https://raw.githubusercontent.com/openai/openai-openapi/master/openapi.yaml -O openai_openapi.yaml
!wget https://www.klarna.com/us/shopping/public/openai/v0/api-docs -O klarna_openapi.yaml
!wget https://raw.githubusercontent.com/APIs-guru/openapi-directory/main/APIs/spotify.com/1.0.0/openapi.yaml -O spotify_openapi.yaml
```
```output
--2023-03-31 15:45:56--  https://raw.githubusercontent.com/openai/openai-openapi/master/openapi.yaml
Resolving raw.githubusercontent.com (raw.githubusercontent.com)... 185.199.110.133, 185.199.109.133, 185.199.111.133, ...
Connecting to raw.githubusercontent.com (raw.githubusercontent.com)|185.199.110.133|:443... connected.
HTTP request sent, awaiting response... 200 OK
Length: 122995 (120K) [text/plain]
Saving to: ‘openapi.yaml’
openapi.yaml        100%[===================>] 120.11K  --.-KB/s    in 0.01s   
2023-03-31 15:45:56 (10.4 MB/s) - ‘openapi.yaml’ saved [122995/122995]
--2023-03-31 15:45:57--  https://www.klarna.com/us/shopping/public/openai/v0/api-docs
Resolving www.klarna.com (www.klarna.com)... 52.84.150.34, 52.84.150.46, 52.84.150.61, ...
Connecting to www.klarna.com (www.klarna.com)|52.84.150.34|:443... connected.
HTTP request sent, awaiting response... 200 OK
Length: unspecified [application/json]
Saving to: ‘api-docs’
api-docs                [ <=>                ]   1.87K  --.-KB/s    in 0s      
2023-03-31 15:45:57 (261 MB/s) - ‘api-docs’ saved [1916]
--2023-03-31 15:45:57--  https://raw.githubusercontent.com/APIs-guru/openapi-directory/main/APIs/spotify.com/1.0.0/openapi.yaml
Resolving raw.githubusercontent.com (raw.githubusercontent.com)... 185.199.110.133, 185.199.109.133, 185.199.111.133, ...
Connecting to raw.githubusercontent.com (raw.githubusercontent.com)|185.199.110.133|:443... connected.
HTTP request sent, awaiting response... 200 OK
Length: 286747 (280K) [text/plain]
Saving to: ‘openapi.yaml’
openapi.yaml        100%[===================>] 280.03K  --.-KB/s    in 0.02s   
2023-03-31 15:45:58 (13.3 MB/s) - ‘openapi.yaml’ saved [286747/286747]
```
```python
from langchain_community.agent_toolkits.openapi.spec import reduce_openapi_spec
```
```python
with open("openai_openapi.yaml") as f:
    raw_openai_api_spec = yaml.load(f, Loader=yaml.Loader)
openai_api_spec = reduce_openapi_spec(raw_openai_api_spec)
with open("klarna_openapi.yaml") as f:
    raw_klarna_api_spec = yaml.load(f, Loader=yaml.Loader)
klarna_api_spec = reduce_openapi_spec(raw_klarna_api_spec)
with open("spotify_openapi.yaml") as f:
    raw_spotify_api_spec = yaml.load(f, Loader=yaml.Loader)
spotify_api_spec = reduce_openapi_spec(raw_spotify_api_spec)
```

---

我们将使用 Spotify API 作为一个稍微复杂的 API 的例子。如果您想要复制这个操作，需要进行一些与授权相关的设置。

- 您将需要在 Spotify 开发者控制台设置一个应用程序，文档在[这里](https://developer.spotify.com/documentation/general/guides/authorization/)，以获取凭证：`CLIENT_ID`、`CLIENT_SECRET` 和 `REDIRECT_URI`。

- 要获取访问令牌（并保持其更新），您可以实现 oauth 流程，或者您可以使用 `spotipy`。如果您已将 Spotify 凭据设置为环境变量 `SPOTIPY_CLIENT_ID`、`SPOTIPY_CLIENT_SECRET` 和 `SPOTIPY_REDIRECT_URI`，您可以使用下面的辅助函数：

```python
import spotipy.util as util
from langchain.requests import RequestsWrapper
def construct_spotify_auth_headers(raw_spec: dict):
    scopes = list(
        raw_spec["components"]["securitySchemes"]["oauth_2_0"]["flows"][
            "authorizationCode"
        ]["scopes"].keys()
    )
    access_token = util.prompt_for_user_token(scope=",".join(scopes))
    return {"Authorization": f"Bearer {access_token}"}
# 获取 API 凭证。
headers = construct_spotify_auth_headers(raw_spotify_api_spec)
requests_wrapper = RequestsWrapper(headers=headers)
```
```python
> Entering new AgentExecutor chain...
Action: api_planner
Action Input: I need to find the right API calls to give me a song that I would like and make it bluesy
Observation: 1. GET /recommendations to get a recommended song
Thought: I have the plan, now I need to execute the API call.
Action: api_controller
Action Input: 1. GET /recommendations to get a recommended song
> Entering new AgentExecutor chain...
Action: requests_get
Action Input: {"url": "https://api.spotify.com/v1/recommendations?seed_genres=blues&limit=1", "output_instructions": "Extract the name of the recommended song"}
Observation: "Stormy Monday"
Thought: I am finished executing the plan.
Final Answer: The recommended song is "Stormy Monday".
> Finished chain.
```
```output
'The recommended song is "Stormy Monday".'
```

> 进入新的 AgentExecutor 链...

动作：api_planner

动作输入：我需要找到合适的 API 调用来为用户获取一首蓝调歌曲推荐

观察：1. 使用 GET /me 获取当前用户的信息

2. 使用 GET /recommendations/available-genre-seeds 检索可用流派的列表

3. 使用带有 seed_genre 参数设置为 "blues" 的 GET /recommendations 来为用户获取一首蓝调歌曲推荐

想法：我已经有计划了，现在需要执行 API 调用。

动作：api_controller

动作输入：1. 使用 GET /me 获取当前用户的信息

2. 使用 GET /recommendations/available-genre-seeds 检索可用流派的列表

3. 使用带有 seed_genre 参数设置为 "blues" 的 GET /recommendations 来为用户获取一首蓝调歌曲推荐

> 进入新的 AgentExecutor 链...

动作：requests_get

动作输入：{"url": "https://api.spotify.com/v1/me", "output_instructions": "提取用户的 id 和用户名"}

观察：ID：22rhrz4m4kvpxlsb5hezokzwi，用户名：Jeremy Welborn

想法：动作：requests_get

动作输入：{"url": "https://api.spotify.com/v1/recommendations/available-genre-seeds", "output_instructions": "提取可用流派的列表"}

观察：acoustic, afrobeat, alt-rock, alternative, ambient, anime, black-metal, bluegrass, blues, bossanova, brazil, breakbeat, british, cantopop, chicago-house, children, chill, classical, club, comedy, country, dance, dancehall, death-metal, deep-house, detroit-techno, disco, disney, drum-and-bass, dub, dubstep, edm, electro, electronic, emo, folk, forro, french, funk, garage, german, gospel, goth, grindcore, groove, grunge, guitar, happy, hard-rock, hardcore, hardstyle, heavy-metal, hip-hop, holidays, honky-tonk, house, idm, indian, indie, indie-pop, industrial, iranian, j-dance, j-idol, j-pop, j-rock, jazz, k-pop, kids, latin, latino, malay, mandopop, metal, metal-misc, metalcore, minimal-techno, movies, mpb, new-age, new-release, opera, pagode, party, philippines-

想法：

``````output

在 4.0 秒后重试 langchain.llms.openai.completion_with_retry.<locals>._completion_with_retry，因为它引发了 RateLimitError：该模型当前由其他请求过载。如果错误持续，请通过我们的帮助中心 help.openai.com 联系我们。 （请在您的消息中包含请求 ID 2167437a0072228238f3c0c5b3882764）。

``````output

动作：requests_get

动作输入：{"url": "https://api.spotify.com/v1/recommendations?seed_genres=blues", "output_instructions": "提取推荐曲目的列表及其 id 和名称"}

观察：[

  {

    id: '03lXHmokj9qsXspNsPoirR',

    name: 'Get Away Jordan'

  }

]

想法：我已经完成执行计划。

最终答案：为用户 Jeremy Welborn（ID：22rhrz4m4kvpxlsb5hezokzwi）推荐的蓝调歌曲是 "Get Away Jordan"，曲目 ID 为：03lXHmokj9qsXspNsPoirR。

> 完成链。

观察：为用户 Jeremy Welborn（ID：22rhrz4m4kvpxlsb5hezokzwi）推荐的蓝调歌曲是 "Get Away Jordan"，曲目 ID 为：03lXHmokj9qsXspNsPoirR。

想法：我已经完成执行计划并获得了用户要求的信息。

最终答案：为您推荐的蓝调歌曲是 "Get Away Jordan"，曲目 ID 为：03lXHmokj9qsXspNsPoirR。

> 完成链。

```
```output

'为您推荐的蓝调歌曲是 "Get Away Jordan"，曲目 ID 为：03lXHmokj9qsXspNsPoirR。'

```
#### 尝试另一个 API。
```python
headers = {"Authorization": f"Bearer {os.getenv('OPENAI_API_KEY')}"}
openai_requests_wrapper = RequestsWrapper(headers=headers)
```
```python
# Meta!
llm = ChatOpenAI(model_name="gpt-4", temperature=0.25)
openai_agent = planner.create_openapi_agent(
    openai_api_spec, openai_requests_wrapper, llm
)
user_query = "生成一条简短的建议"
openai_agent.invoke(user_query)
```
```output

> 进入新的 AgentExecutor 链...

动作：api_planner

动作输入：我需要找到合适的 API 调用来生成一条简短的建议

观察：1. 使用 GET /engines 检索可用引擎的列表

2. 使用选定的引擎和提示进行 POST /completions 以生成一条简短的建议

想法：我已经有计划了，现在需要执行 API 调用。

动作：api_controller

动作输入：1. 使用 GET /engines 检索可用引擎的列表

2. 使用选定的引擎和提示进行 POST /completions 以生成一条简短的建议

> 进入新的 AgentExecutor 链...

动作：requests_get

动作输入：{"url": "https://api.openai.com/v1/engines", "output_instructions": "提取引擎的 id"}

观察：babbage, davinci, text-davinci-edit-001, babbage-code-search-code, text-similarity-babbage-001, code-davinci-edit-001, text-davinci-001, ada, babbage-code-search-text, babbage-similarity, whisper-1, code-search-babbage-text-001, text-curie-001, code-search-babbage-code-001, text-ada-001, text-embedding-ada-002, text-similarity-ada-001, curie-instruct-beta, ada-code-search-code, ada-similarity, text-davinci-003, code-search-ada-text-001, text-search-ada-query-001, davinci-search-document, ada-code-search-text, text-search-ada-doc-001, davinci-instruct-beta, text-similarity-curie-001, code-search-ada-code-001

思考：我将使用"davinci"引擎生成一条关于如何提高工作效率的简短建议。

行动：请求发送

行动输入：{"url": "https://api.openai.com/v1/completions", "data": {"engine": "davinci", "prompt": "Give me a short piece of advice on how to be more productive."}, "output_instructions": "Extract the text from the first choice"}

观察："you must provide a model parameter"

思考：!! 无法从"I cannot finish executing the plan without knowing how to provide the model parameter correctly." 中提取工具和输入_get_next_action

我无法在不知道如何正确提供模型参数的情况下完成执行计划。

> 完成链。

观察：我需要更多关于如何在POST请求中正确提供模型参数以生成一条简短建议的信息。

思考：我需要调整我的计划，以在POST请求中包括模型参数。

行动：api_planner

行动输入：我需要找到正确的API调用来生成一条简短建议，包括在POST请求中包括模型参数

观察：1. GET /models 获取可用模型列表

2. 从列表中选择合适的模型

3. POST /completions 使用所选模型作为参数生成一条简短建议

思考：我有了更新的计划，现在需要执行API调用。

行动：api_controller

行动输入：1. GET /models 获取可用模型列表

2. 从列表中选择合适的模型

3. POST /completions 使用所选模型作为参数生成一条简短建议

> 进入新的AgentExecutor链...

行动：请求获取

行动输入：{"url": "https://api.openai.com/v1/models", "output_instructions": "Extract the ids of the available models"}

观察：babbage, davinci, text-davinci-edit-001, babbage-code-search-code, text-similarity-babbage-001, code-davinci-edit-001, text-davinci-edit-001, ada

思考：行动：请求发送

行动输入：{"url": "https://api.openai.com/v1/completions", "data": {"model": "davinci", "prompt": "Give me a short piece of advice on how to improve communication skills."}, "output_instructions": "Extract the text from the first choice"}

观察："I'd like to broaden my horizon.\n\nI was trying to"

思考：我无法在不知道其他信息的情况下完成执行计划。

最终答案：生成的文本不是关于提高沟通技巧的建议。我需要使用不同的提示或模型重试API调用，以获得更相关的响应。

> 完成链。

观察：生成的文本不是关于提高沟通技巧的建议。我需要使用不同的提示或模型重试API调用，以获得更相关的响应。

思考：我需要调整我的计划，以包括更具体的提示，以生成一条关于提高沟通技巧的简短建议。

行动：api_planner

行动输入：我需要找到正确的API调用来生成一条关于提高沟通技巧的简短建议，包括在POST请求中包括模型参数

观察：1. GET /models 获取可用模型列表

2. 从列表中选择适合生成文本的模型（例如，text-davinci-002）

3. POST /completions 使用所选模型和与提高沟通技巧相关的提示生成一条简短建议

思考：我有了更新的计划，现在需要执行API调用。

行动：api_controller

行动输入：1. GET /models 获取可用模型列表

2. 从列表中选择适合生成文本的模型（例如，text-davinci-002）

3. POST /completions 使用所选模型和与提高沟通技巧相关的提示生成一条简短建议

> 进入新的AgentExecutor链...

行动：请求获取

行动输入：{"url": "https://api.openai.com/v1/models", "output_instructions": "Extract the names of the models"}

观察：babbage, davinci, text-davinci-edit-001, babbage-code-search-code, text-similarity-babbage-001, code-davinci-edit-001, text-davinci-edit-001, ada

思考：行动：请求发送

行动输入：{"url": "https://api.openai.com/v1/completions", "data": {"model": "text-davinci-002", "prompt": "Give a short piece of advice on how to improve communication skills"}, "output_instructions": "Extract the text from the first choice"}

观察：一些改善沟通技巧的基本建议是确保倾听。

思考：我已经执行完计划。

最终答案：一些改善沟通技巧的基本建议是确保倾听。

> 完成链。

观察：一些改善沟通技巧的基本建议是确保倾听。

思考：我已经执行完计划并获得了用户要求的信息。

最终答案：改善沟通技巧的一个简短建议是确保倾听。

> 完成链。

需要一段时间才能到达目标！

## 第二个例子："json explorer" 代理

这是一个不太实用但很有趣的代理！该代理可以访问两个工具包。一个包含与 JSON 交互的工具：一个用于列出 JSON 对象的键的工具，另一个用于获取给定键的值的工具。另一个工具包包括 `requests` 的包装器，用于发送 GET 和 POST 请求。这个代理消耗了大量语言模型的调用，但表现出人意外的不错。

```python
from langchain_community.agent_toolkits import OpenAPIToolkit, create_openapi_agent
from langchain_community.tools.json.tool import JsonSpec
from langchain_openai import OpenAI
```
```python
with open("openai_openapi.yaml") as f:
    data = yaml.load(f, Loader=yaml.FullLoader)
json_spec = JsonSpec(dict_=data, max_value_length=4000)
openapi_toolkit = OpenAPIToolkit.from_llm(
    OpenAI(temperature=0), json_spec, openai_requests_wrapper, verbose=True
)
openapi_agent_executor = create_openapi_agent(
    llm=OpenAI(temperature=0),
    toolkit=openapi_toolkit,
    allow_dangerous_requests=ALLOW_DANGEROUS_REQUEST,
    verbose=True,
)
```
```python
openapi_agent_executor.run(
    "Make a post request to openai /completions. The prompt should be 'tell me a joke.'"
)
```

> 进入新的 AgentExecutor 链...

动作：json_explorer

动作输入：API 的基本 URL 是什么？

> 进入新的 AgentExecutor 链...

动作：json_spec_list_keys

动作输入：data

观察：['openapi', 'info', 'servers', 'tags', 'paths', 'components', 'x-oaiMeta']

思考：我应该查看 servers 键，看看基本 URL 是什么

动作：json_spec_list_keys

动作输入：data["servers"][0]

观察：ValueError('Value at path `data["servers"][0]` is not a dict, get the value directly.')

思考：我应该获取 servers 键的值

动作：json_spec_get_value

动作输入：data["servers"][0]

观察：{'url': 'https://api.openai.com/v1'}

思考：我现在知道 API 的基本 URL

最终答案：API 的基本 URL 是 https://api.openai.com/v1

> 完成链。

观察：API 的基本 URL 是 https://api.openai.com/v1

思考：我应该找到 /completions 端点的路径。

动作：json_explorer

动作输入：/completions 端点的路径是什么？

> 进入新的 AgentExecutor 链...

动作：json_spec_list_keys

动作输入：data

观察：['openapi', 'info', 'servers', 'tags', 'paths', 'components', 'x-oaiMeta']

思考：我应该查看 paths 键，看看存在哪些端点

动作：json_spec_list_keys

动作输入：data["paths"]

观察：['/engines', '/engines/{engine_id}', '/completions', '/chat/completions', '/edits', '/images/generations', '/images/edits', '/images/variations', '/embeddings', '/audio/transcriptions', '/audio/translations', '/engines/{engine_id}/search', '/files', '/files/{file_id}', '/files/{file_id}/content', '/answers', '/classifications', '/fine-tunes', '/fine-tunes/{fine_tune_id}', '/fine-tunes/{fine_tune_id}/cancel', '/fine-tunes/{fine_tune_id}/events', '/models', '/models/{model}', '/moderations']

思考：我现在知道 /completions 端点的路径

最终答案：/completions 端点的路径是 data["paths"][2]

> 完成链。

观察：/completions 端点的路径是 data["paths"][2]

思考：我应该找到向 /completions 端点发送 POST 请求所需的参数。

动作：json_explorer

动作输入：向 /completions 端点发送 POST 请求所需的参数是什么？

> 进入新的 AgentExecutor 链...

动作：json_spec_list_keys

动作输入：data

观察：['openapi', 'info', 'servers', 'tags', 'paths', 'components', 'x-oaiMeta']

思考：我应该查看 paths 键，看看存在哪些端点

动作：json_spec_list_keys

动作输入：data["paths"]

观察：['/engines', '/engines/{engine_id}', '/completions', '/chat/completions', '/edits', '/images/generations', '/images/edits', '/images/variations', '/embeddings', '/audio/transcriptions', '/audio/translations', '/engines/{engine_id}/search', '/files', '/files/{file_id}', '/files/{file_id}/content', '/answers', '/classifications', '/fine-tunes', '/fine-tunes/{fine_tune_id}', '/fine-tunes/{fine_tune_id}/cancel', '/fine-tunes/{fine_tune_id}/events', '/models', '/models/{model}', '/moderations']

想法：我应该查看 /completions 端点，看看需要哪些参数

行动：json_spec_list_keys

行动输入：data["paths"]["/completions"]

观察：['post']

想法：我应该查看 post 键，看看需要哪些参数

行动：json_spec_list_keys

行动输入：data["paths"]["/completions"]["post"]

观察：['operationId', 'tags', 'summary', 'requestBody', 'responses', 'x-oaiMeta']

想法：我应该查看 requestBody 键，看看需要哪些参数

行动：json_spec_list_keys

行动输入：data["paths"]["/completions"]["post"]["requestBody"]

观察：['required', 'content']

想法：我应该查看 content 键，看看需要哪些参数

行动：json_spec_list_keys

行动输入：data["paths"]["/completions"]["post"]["requestBody"]["content"]

观察：['application/json']

想法：我应该查看 application/json 键，看看需要哪些参数

行动：json_spec_list_keys

行动输入：data["paths"]["/completions"]["post"]["requestBody"]["content"]["application/json"]

观察：['schema']

想法：我应该查看 schema 键，看看需要哪些参数

行动：json_spec_list_keys

行动输入：data["paths"]["/completions"]["post"]["requestBody"]["content"]["application/json"]["schema"]

观察：['$ref']

想法：我应该查看 $ref 键，看看需要哪些参数

行动：json_spec_list_keys

行动输入：data["paths"]["/completions"]["post"]["requestBody"]["content"]["application/json"]["schema"]["$ref"]

观察：ValueError('Value at path `data["paths"]["/completions"]["post"]["requestBody"]["content"]["application/json"]["schema"]["$ref"]` is not a dict, get the value directly.')

想法：我应该直接获取 $ref 键的值

行动：json_spec_get_value

行动输入：data["paths"]["/completions"]["post"]["requestBody"]["content"]["application/json"]["schema"]["$ref"]

观察：#/components/schemas/CreateCompletionRequest

想法：我应该查看 CreateCompletionRequest 模式，看看需要哪些参数

行动：json_spec_list_keys

行动输入：data["components"]["schemas"]["CreateCompletionRequest"]

观察：['type', 'properties', 'required']

想法：我应该查看 required 键，看看需要哪些参数

行动：json_spec_get_value

行动输入：data["components"]["schemas"]["CreateCompletionRequest"]["required"]

观察：['model']

想法：我现在知道最终答案

最终答案：对于对 /completions 端点的 POST 请求，需要的参数是 'model'。

> 链结束。

观察：对于对 /completions 端点的 POST 请求，需要的参数是 'model'。

想法：我现在知道需要进行请求的参数。

行动：requests_post

行动输入：{ "url": "https://api.openai.com/v1/completions", "data": { "model": "davinci", "prompt": "tell me a joke" } }

观察：{"id":"cmpl-70Ivzip3dazrIXU8DSVJGzFJj2rdv","object":"text_completion","created":1680307139,"model":"davinci","choices":[{"text":" with mummy not there”\n\nYou dig deep and come up with,","index":0,"logprobs":null,"finish_reason":"length"}],"usage":{"prompt_tokens":4,"completion_tokens":16,"total_tokens":20}}

想法：我现在知道最终答案。

最终答案：对于 POST 请求的响应是 {"id":"cmpl-70Ivzip3dazrIXU8DSVJGzFJj2rdv","object":"text_completion","created":1680307139,"model":"davinci","choices":[{"text":" with mummy not there”\n\nYou dig deep and come up with,","index":0,"logprobs":null,"finish_reason":"length"}],"usage":{"prompt_tokens":4,"completion_tokens":16,"total_tokens":20}}

> 链结束。

```output
'The response of the POST request is {"id":"cmpl-70Ivzip3dazrIXU8DSVJGzFJj2rdv","object":"text_completion","created":1680307139,"model":"davinci","choices":[{"text":" with mummy not there”\\n\\nYou dig deep and come up with,","index":0,"logprobs":null,"finish_reason":"length"}],"usage":{"prompt_tokens":4,"completion_tokens":16,"total_tokens":20}}'
```