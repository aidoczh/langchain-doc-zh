# Azure 认知服务

这个工具包用于与 `Azure 认知服务 API` 进行交互，实现一些多模态功能。

目前，这个工具包中包含四个工具：

- AzureCogsImageAnalysisTool：用于从图像中提取标题、对象、标签和文本。（注意：由于依赖于目前仅支持 Windows 和 Linux 的 `azure-ai-vision` 软件包，此工具目前尚不支持 Mac OS。）

- AzureCogsFormRecognizerTool：用于从文档中提取文本、表格和键值对。

- AzureCogsSpeech2TextTool：用于将语音转录为文本。

- AzureCogsText2SpeechTool：用于将文本合成语音。

- AzureCogsTextAnalyticsHealthTool：用于提取医疗保健实体。

首先，您需要设置一个 Azure 账户并创建一个认知服务资源。您可以按照[这里](https://docs.microsoft.com/en-us/azure/cognitive-services/cognitive-services-apis-create-account?tabs=multiservice%2Cwindows)的说明来创建一个资源。

然后，您需要获取资源的端点、密钥和区域，并将它们设置为环境变量。您可以在资源的“Keys and Endpoint”页面找到它们。

```python
%pip install --upgrade --quiet  azure-ai-formrecognizer > /dev/null
%pip install --upgrade --quiet  azure-cognitiveservices-speech > /dev/null
%pip install --upgrade --quiet  azure-ai-textanalytics > /dev/null
# 对于 Windows/Linux
%pip install --upgrade --quiet  azure-ai-vision > /dev/null
```

```python
import os
os.environ["OPENAI_API_KEY"] = "sk-"
os.environ["AZURE_COGS_KEY"] = ""
os.environ["AZURE_COGS_ENDPOINT"] = ""
os.environ["AZURE_COGS_REGION"] = ""
```

## 创建工具包

```python
from langchain_community.agent_toolkits import AzureCognitiveServicesToolkit
toolkit = AzureCognitiveServicesToolkit()
```

```python
[tool.name for tool in toolkit.get_tools()]
```

```output
['Azure Cognitive Services Image Analysis',
 'Azure Cognitive Services Form Recognizer',
 'Azure Cognitive Services Speech2Text',
 'Azure Cognitive Services Text2Speech']
```

## 在代理中使用

```python
from langchain.agents import AgentType, initialize_agent
from langchain_openai import OpenAI
```

```python
llm = OpenAI(temperature=0)
agent = initialize_agent(
    tools=toolkit.get_tools(),
    llm=llm,
    agent=AgentType.STRUCTURED_CHAT_ZERO_SHOT_REACT_DESCRIPTION,
    verbose=True,
)
```

```python
agent.run(
    "What can I make with these ingredients?"
    "https://images.openai.com/blob/9ad5a2ab-041f-475f-ad6a-b51899c50182/ingredients.png"
)
```

```output
> 进入新的 AgentExecutor 链...
操作:
```

{

  "action": "Azure Cognitive Services Image Analysis",

  "action_input": "https://images.openai.com/blob/9ad5a2ab-041f-475f-ad6a-b51899c50182/ingredients.png"

}

```
观察：标题：几个碗里的鸡蛋和面粉
对象：鸡蛋、鸡蛋、食物
标签：乳制品、成分、室内、增稠剂、食物、搅拌碗、粉末、面粉、鸡蛋、碗
想法：我可以利用对象和标签来推荐食谱
操作：
```

{

  "action": "Final Answer",

  "action_input": "您可以用这些成分做煎饼、煎蛋或奶酪蛋饼！"

}

> 链结束。

```output
'您可以用这些成分做煎饼、煎蛋或奶酪蛋饼！'
```

```python
audio_file = agent.run("Tell me a joke and read it out for me.")
```

```output
> 进入新的 AgentExecutor 链...
操作:
```

{

  "action": "Azure Cognitive Services Text2Speech",

  "action_input": "为什么小鸡过操场？为了到达另一边的滑梯！"

}

```
观察：/tmp/tmpa3uu_j6b.wav
想法：我有这个笑话的音频文件
操作：
```

{

  "action": "Final Answer",

  "action_input": "/tmp/tmpa3uu_j6b.wav"

}

> 链结束。

```output
'/tmp/tmpa3uu_j6b.wav'
```

```python
from IPython import display
audio = display.Audio(audio_file)
display.display(audio)
```

```python
agent.run(
    """患者是一名 54 岁的绅士，几个月来一直有进行性心绞痛的病史。
患者于今年 7 月进行了心脏导管造影，显示右冠状动脉完全闭塞，左主干病变 50% ，有冠心病家族史，其中一位兄弟在 52 岁时死于心肌梗死，
另一位兄弟做过冠状动脉旁路移植术。患者于 2001 年 7 月做了一次负荷超声心动图检查，未见壁运动异常，但由于体形特殊，这是一次困难的检查。
患者在前侧壁导联出现轻微 ST 下降，认为是由于疲劳和手腕疼痛引起的心绞痛等效症。由于患者症状加重、家族史和左主干病变合并右冠状动脉完全闭塞，
被转诊接受开放性心脏手术血管重建。
列出所有诊断。
"""
)
```

```output
> 进入新的 AgentExecutor 链...
操作：
```

{

  "action": "azure_cognitive_services_text_analyics_health",

  "action_input": "The patient is a 54-year-old gentleman with a history of progressive angina over the past several months. The patient had a cardiac catheterization in July of this year revealing total occlusion of the RCA and 50% left main disease, with a strong family history of coronary artery disease with a brother dying at the age of 52 from a myocardial infarction and another brother who is status post coronary artery bypass grafting. The patient had a stress echocardiogram done on July, 2001, which showed no wall motion abnormalities, but this was a difficult study due to body habitus. The patient went for six minutes with minimal ST depressions in the anterior lateral leads, thought due to fatigue and wrist pain, his anginal equivalent. Due to the patient's increased symptoms and family history and history left main disease with total occasional of his RCA was referred for revascularization with open heart surgery."

}

```
观察：文本包含以下医疗实体：54 岁是年龄类型的医疗实体，绅士是性别类型的医疗实体，进行性心绞痛是诊断类型的医疗实体，过去几个月是时间类型的医疗实体，心脏导管造影是检查名称类型的医疗实体，今年七月是时间类型的医疗实体，完全是状况描述类型的医疗实体，闭塞是症状或体征类型的医疗实体，右冠状动脉是身体结构类型的医疗实体，50% 是测量数值类型的医疗实体，左主是身体结构类型的医疗实体，疾病是诊断类型的医疗实体，家族是家庭关系类型的医疗实体，冠状动脉疾病是诊断类型的医疗实体，兄弟是家庭关系类型的医疗实体，死亡是诊断类型的医疗实体，52 是年龄类型的医疗实体，心肌梗死是诊断类型的医疗实体，兄弟是家庭关系类型的医疗实体，冠状动脉旁路移植术是治疗名称类型的医疗实体，应激超声心动图是检查名称类型的医疗实体，2001 年七月是时间类型的医疗实体，心壁运动异常是症状或体征类型的医疗实体，体形是症状或体征类型的医疗实体，六分钟是时间类型的医疗实体，最小是状况描述类型的医疗实体，前侧壁导联 ST 段压低是症状或体征类型的医疗实体，疲劳是症状或体征类型的医疗实体，手腕疼痛是症状或体征类型的医疗实体，心绞痛等效是症状或体征类型的医疗实体，加重是病程类型的医疗实体，症状是症状或体征类型的医疗实体，家族是家庭关系类型的医疗实体，左是方向类型的医疗实体，主要是身体结构类型的医疗实体，疾病是诊断类型的医疗实体，偶发是病程类型的医疗实体，右冠状动脉是身体结构类型的医疗实体，再血管化是治疗名称类型的医疗实体，开放性心脏手术是治疗名称类型的医疗实体
想法：我知道该如何回应
操作：
```

{

  "action": "Final Answer",

  "action_input": "The text contains the following diagnoses: progressive angina, coronary artery disease, myocardial infarction, and coronary artery bypass grafting."

}

```
> 链结束。
```

```output
'文本包含以下诊断：进行性心绞痛，冠状动脉疾病，心肌梗死和冠状动脉旁路移植术。'
```