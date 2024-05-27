# SceneXplain

[SceneXplain](https://scenex.jina.ai/) 是一项通过 SceneXplain 工具访问的图像描述服务。

要使用这个工具，您需要创建一个账户并从[网站](https://scenex.jina.ai/api)获取您的 API 令牌，然后您就可以实例化这个工具。

```python
import os
os.environ["SCENEX_API_KEY"] = "<YOUR_API_KEY>"
```

```python
from langchain.agents import load_tools
tools = load_tools(["sceneXplain"])
```

或者直接实例化这个工具。

```python
from langchain_community.tools import SceneXplainTool
tool = SceneXplainTool()
```

## 在代理中的使用

可以在任何 LangChain 代理中如下使用这个工具：

```python
from langchain.agents import initialize_agent
from langchain.memory import ConversationBufferMemory
from langchain_openai import OpenAI
llm = OpenAI(temperature=0)
memory = ConversationBufferMemory(memory_key="chat_history")
agent = initialize_agent(
    tools, llm, memory=memory, agent="conversational-react-description", verbose=True
)
output = agent.run(
    input=(
        "这张图片 https://storage.googleapis.com/causal-diffusion.appspot.com/imagePrompts%2F0rw369i5h9t%2Foriginal.png 中有什么？是电影还是游戏？如果是电影，电影的名字是什么？"
    )
)
print(output)
```

```output
> 进入新的 AgentExecutor 链...
思考：我需要使用一个工具吗？是
行动：图像解释器
行动输入：https://storage.googleapis.com/causal-diffusion.appspot.com/imagePrompts%2F0rw369i5h9t%2Foriginal.png
观察：在一个迷人而奇幻的场景中，一个年轻女孩与她可爱的龙猫毛绒伴侣一同冒雨而行。两者站在熙熙攘攘的街角，被一把明黄色的伞遮挡雨水。女孩身穿一件开心的黄色连衣裙，双手握住伞柄，目光注视着龙猫，表情中充满了惊奇和喜悦。
龙猫高大而自豪地站在他的小伙伴身旁，举起自己的伞，保护他们免受倾盆大雨。他毛茸茸的身体呈现出丰富的灰色和白色，他那双大耳朵和大眼睛赋予了他迷人的魅力。
在场景的背景中，可以看到一块街道标志从人行道上伸出，雨滴纷飞。标志上有着中文字符，增添了文化多样性和神秘感。尽管天气阴沉，但这幅温馨的画面中却充满了快乐和友情。
思考：我需要使用一个工具吗？不需要
AI：这张图片似乎是来自1988年的日本动画奇幻电影《龙猫》。电影讲述了两个年轻女孩，小月和小梅，在探索乡村并与神奇的森林精灵交友，其中包括主角龙猫。
> 链结束。
这张图片似乎是来自1988年的日本动画奇幻电影《龙猫》。电影讲述了两个年轻女孩，小月和小梅，在探索乡村并与神奇的森林精灵交友，其中包括主角龙猫。
```