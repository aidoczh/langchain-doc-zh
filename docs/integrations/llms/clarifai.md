# Clarifai

[Clarifai](https://www.clarifai.com/) 是一个提供完整 AI 生命周期的 AI 平台，涵盖数据探索、数据标记、模型训练、评估和推理等方面。

以下示例介绍如何使用 LangChain 与 `Clarifai` [模型](https://clarifai.com/explore/models) 进行交互。

要使用 Clarifai，您必须拥有一个帐户和个人访问令牌（PAT）密钥。请在[此处检查](https://clarifai.com/settings/security)获取或创建 PAT。

# 依赖关系

```python
# 安装所需依赖项
%pip install --upgrade --quiet clarifai
```

```python
# 将 Clarifai pat 令牌声明为环境变量，或者您可以将其作为参数传递给 clarifai 类。
import os
os.environ["CLARIFAI_PAT"] = "CLARIFAI_PAT_TOKEN"
```

# 导入

在这里，我们将设置个人访问令牌。您可以在 Clarifai 帐户的[设置/安全性](https://clarifai.com/settings/security)下找到您的 PAT。

```python
# 请登录并从 https://clarifai.com/settings/security 获取您的 API 密钥
from getpass import getpass
CLARIFAI_PAT = getpass()
```

```python
# 导入所需模块
from langchain.chains import LLMChain
from langchain_community.llms import Clarifai
from langchain_core.prompts import PromptTemplate
```

# 输入

创建一个用于 LLM Chain 的提示模板：

```python
template = """Question: {question}
Answer: Let's think step by step."""
prompt = PromptTemplate.from_template(template)
```

# 设置

设置模型所在的用户 ID 和应用程序 ID。您可以在 https://clarifai.com/explore/models 上找到公共模型列表。

您还需要初始化模型 ID，如果需要，还可以提供模型版本 ID。一些模型有多个版本，您可以选择适合您任务的版本。

或者，您可以使用模型 URL（例如："https://clarifai.com/anthropic/completion/models/claude-v2"）进行初始化。

```python
USER_ID = "openai"
APP_ID = "chat-completion"
MODEL_ID = "GPT-3_5-turbo"
# 您可以将特定模型版本提供为 model_version_id 参数。
# MODEL_VERSION_ID = "MODEL_VERSION_ID"
# 或
MODEL_URL = "https://clarifai.com/openai/chat-completion/models/GPT-4"
```

```python
# 初始化 Clarifai LLM
clarifai_llm = Clarifai(user_id=USER_ID, app_id=APP_ID, model_id=MODEL_ID)
# 或
# 通过模型 URL 进行初始化
clarifai_llm = Clarifai(model_url=MODEL_URL)
```

```python
# 创建 LLM chain
llm_chain = LLMChain(prompt=prompt, llm=clarifai_llm)
```

# 运行 Chain

```python
question = "贾斯汀·比伯出生年份的超级碗冠军是哪支 NFL 球队？"
llm_chain.run(question)
```

```output
'好的，以下是找出答案的步骤：
1. 贾斯汀·比伯出生于1994年3月1日。
2. 他出生年份的超级碗是第二十八届超级碗。
3. 第二十八届超级碗于1994年1月30日举行。
4. 参加第二十八届超级碗的两支球队分别是达拉斯牛仔队和水牛城比尔斯队。
5. 达拉斯牛仔队以30-13战胜水牛城比尔斯队，赢得了第二十八届超级碗。
因此，贾斯汀·比伯出生年份的超级碗冠军是达拉斯牛仔队。'
```

## 使用推理参数对 GPT 进行模型预测

您还可以使用带有推理参数（如温度、max_tokens 等）的 GPT 模型。

```python
# 将参数初始化为字典。
params = dict(temperature=str(0.3), max_tokens=100)
```

```python
clarifai_llm = Clarifai(user_id=USER_ID, app_id=APP_ID, model_id=MODEL_ID)
llm_chain = LLMChain(
    prompt=prompt, llm=clarifai_llm, llm_kwargs={"inference_params": params}
)
```

```python
question = "如果一个数字是5，那么接下来的数字必须是7，那么你可以组成多少个三位数的偶数？"
llm_chain.run(question)
```

```output
'步骤1：第一个数字可以是1到9之间的任何偶数，但不能是5。因此，第一个数字有4个选择。
步骤2：如果第一个数字不是5，则第二个数字必须是7。因此，第二个数字只有1个选择。
步骤3：第三个数字可以是0到9之间的任何偶数，但不能是5和7。因此，有'
```

为提示列表生成响应

```python
# 我们可以使用 _generate 来为提示列表生成响应。
clarifai_llm._generate(
    [
        "帮我用 5 句话总结美国革命的事件",
        "用幽默的方式解释火箭科学",
        "为大学运动会的欢迎致辞创建一个剧本",
    ],
    inference_params=params,
)
```

```output
LLMResult(generations=[[Generation(text='以下是美国革命关键事件的 5 句话总结：\n\n美国革命始于美国殖民地与英国政府之间关于无代表性征税问题的紧张局势。1775年，在列克星敦和康科德，英军与美国民兵爆发冲突，拉开了革命战争的序幕。大陆会议任命乔治·华盛顿为大陆军总司令，大陆军随后在对英国的关键战役中取得胜利。1776年，通过《独立宣言》，正式宣布 13 个美国殖民地摆脱英国统治。经过多年的战斗，革命战争于1781年约克镇英军失败并承认美国独立结束。')], [Generation(text='以下是有趣的方式解释火箭科学：\n\n火箭科学如此简单，几乎就是小孩的游戏！只需将装满爆炸液体的大金属管绑在你的屁股上，点燃导火线。会发生什么呢？发射！呼啸声，你将在瞬间飞往月球。只需记得戴上头盔，否则当你离开大气层时，你的头可能会像挤痘痘一样爆炸。\n\n制作火箭也很简单。只需混合一些辛辣的香料、大蒜粉、辣椒粉、一点火药，然后大功告成——火箭燃料！如果想要额外的火力，可以加一点小苏打和醋。摇匀倒入 DIY 苏打瓶火箭中。退后一点，看着那个宝贝飞翔！\n\n引导火箭对全家都很有趣。只需系好安全带，按几个随机按钮，看看你最终会到哪里。这就像终极惊喜假期！你永远不知道自己会在金星上，在火星上坠毁，还是快速穿过土星的环。如果出了什么问题，别担心。火箭科学轻而易举。只需用一些胶带和疯狂胶水即可即时解决问题，你很快就能重新找到正确的方向。当你拥有这些时，谁还需要任务控制呢？')], [Generation(text='以下是大学运动会欢迎致辞的草稿：\n\n大家早上好，欢迎来到我们学院的年度运动会！很高兴看到这么多学生、教职员工、校友和来宾今天聚集在这里，共同庆祝我们学院的体育精神和运动成就。\n\n让我们首先感谢所有在幕后辛勤工作的组织者、志愿者、教练和工作人员，没有你们的奉献和承诺，我们的运动会将无法举行。\n\n我还要感谢今天与我们在一起的所有学生运动员。你们的才华、精神和决心让我们感到鼓舞。体育具有独特的力量，能够团结和激励我们的社区。通过个人和团体运动，你们展示了专注、合作、毅力和韧性——这些品质将在赛场内外为你们服务。\n\n竞争精神和公平竞赛是任何体育赛事的核心价值观。我鼓励大家今天积极参与比赛。发挥出最好的能力并享受比赛。无论结果如何，都要为你的同行运动员的努力和体育精神鼓掌。无论输赢，这个运动会都是我们建立友谊、创造终身回忆的日子。让我们把它变成一个为所有人带来健康和友谊的日子。祝愿大家度过愉快的一天！')]], llm_output=None, run=None)
```

```
