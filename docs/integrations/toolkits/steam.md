# Steam 游戏推荐与游戏详情

[Steam (Wikipedia)](https://en.wikipedia.org/wiki/Steam_(service)) 是由 `Valve Corporation` 开发的视频游戏数字发行服务和商店。它为 Valve 的游戏提供自动更新，并扩展到了第三方游戏的发行。`Steam` 提供各种功能，如与 Valve 反作弊措施的游戏服务器匹配、社交网络和游戏流媒体服务。

[Steam](https://store.steampowered.com/about/) 是玩游戏、讨论游戏和创作游戏的终极目的地。

Steam 工具包包括两个工具：

- `游戏详情`

- `推荐游戏`

本笔记本提供了使用 Steam API 与 LangChain 的演示，以根据您当前的 Steam 游戏清单获取 Steam 游戏推荐，或者收集您提供的某些 Steam 游戏的信息。

## 设置

我们需要安装两个 Python 库。

## 导入

```python
%pip install --upgrade --quiet  python-steam-api python-decouple
```

## 分配环境变量

要使用此工具包，请准备好您的 OpenAI API 密钥、Steam API 密钥（从[这里](https://steamcommunity.com/dev/apikey)获取）和您自己的 SteamID。一旦您收到了 Steam API 密钥，您可以在下面将其输入为环境变量。

工具包将读取 "STEAM_KEY" API 密钥作为环境变量来进行身份验证，请在此处设置。您还需要设置您的 "OPENAI_API_KEY" 和 "STEAM_ID"。

```python
import os
os.environ["STEAM_KEY"] = "xyz"
os.environ["STEAM_ID"] = "123"
os.environ["OPENAI_API_KEY"] = "abc"
```

## 初始化

初始化 LLM、SteamWebAPIWrapper、SteamToolkit，最重要的是 langchain 代理来处理您的查询！

## 示例

```python
from langchain.agents import AgentType, initialize_agent
from langchain_community.agent_toolkits.steam.toolkit import SteamToolkit
from langchain_community.utilities.steam import SteamWebAPIWrapper
from langchain_openai import OpenAI
```

```python
llm = OpenAI(temperature=0)
Steam = SteamWebAPIWrapper()
toolkit = SteamToolkit.from_steam_api_wrapper(Steam)
agent = initialize_agent(
    toolkit.get_tools(), llm, agent=AgentType.ZERO_SHOT_REACT_DESCRIPTION, verbose=True
)
```

```python
out = agent("can you give the information about the game Terraria")
print(out)
```

```output
> 进入新的 AgentExecutor 链...
我需要找到游戏详情
动作：获取游戏详情
动作输入：Terraria
观察结果：id 为：105600
链接为：https://store.steampowered.com/app/105600/Terraria/?snr=1_7_15__13
价格为：$9.99
游戏摘要为：“挖掘、战斗、探索、建造：整个世界都掌握在你的手中，当你为生存、财富和荣耀而战。你会深入洞穴广袤的地下，寻找宝藏和原材料，用它们打造不断进化的装备、机械和美学吗？也许你会选择寻找更强大的敌人，来测试你在战斗中的能力？也许你会决定建造自己的城市，来容纳你在旅途中可能遇到的神秘盟友？在《泰拉瑞亚》的世界中，选择权在你手中！《泰拉瑞亚》融合了经典动作游戏的元素和沙盒式创意自由，是一种独特的游戏体验，玩家完全掌控着旅程和目的地。《泰拉瑞亚》的冒险真正如同玩家本身一样独特！你准备好探索、创造和保卫自己的世界了吗？主要特点：沙盒游戏、随机生成的世界、免费内容更新”
游戏支持的语言有：英语、法语、意大利语、德语、西班牙语 - 西班牙、波兰语、葡萄牙语 - 巴西、俄语、简体中文
思考：我现在知道最终答案
最终答案：Terraria 是一款 id 为 105600 的游戏，链接为 https://store.steampowered.com/app/105600/Terraria/?snr=1_7_15__13，价格为 $9.99，摘要为“挖掘、战斗、探索、建造：整个世界都掌握在你的手中，当你为生存、财富和荣耀而战。你会深入洞穴广袤的地下，寻找宝藏和原材料，用它们打造不断进化的装备、机械和美学吗？也许你会选择寻找更强大的敌人，来测试你在战斗中的能力？也许你会决定建造自己的城市，来容纳你在旅途中可能遇到的神秘盟友？在《泰拉瑞亚》的世界中，选择权在你手中！《泰拉瑞亚》融合了经典动作游戏的元素和沙盒式创意自由，是一种独特的游戏体验，玩家完全掌控着旅程和目的地。《泰拉瑞亚》的冒险真正如同玩家本身一样独特！你准备好探索、创造和保卫自己的世界了吗？”
> 完成链。
{'input': 'can you give the information about the game Terraria', 'output': 'Terraria 是一款 id 为 105600 的游戏，链接为 https://store.steampowered.com/app/105600/Terraria/?snr=1_7_15__13，价格为 $9.99，摘要为“挖掘、战斗、探索、建造：整个世界都掌握在你的手中，当你为生存、财富和荣耀而战。你会深入洞穴广袤的地下，寻找宝藏和原材料，用它们打造不断进化的装备、机械和美学吗？也许你会选择寻找更强大的敌人，来测试你在战斗中的能力？也许你会决定建造自己的城市，来容纳你在旅途中可能遇到的神秘盟友？在《泰拉瑞亚》的世界中，选择权在你手中！《泰拉瑞亚》融合了经典动作游戏的元素和沙盒式创意自由，是一种独特的游戏体验，玩家完全掌控着旅程和目的地。《泰拉瑞亚》的冒险真正如同玩家本身一样独特！你准备好探索、创造和保卫自己的世界了吗？'}
```