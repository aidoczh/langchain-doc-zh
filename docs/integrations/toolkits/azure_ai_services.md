# Azure AI 服务

这个工具包用于与“Azure AI 服务 API”进行交互，实现一些多模态功能。

目前，这个工具包中包含五个工具：

- **AzureAiServicesImageAnalysisTool**：用于从图像中提取标题、对象、标签和文本。

- **AzureAiServicesDocumentIntelligenceTool**：用于从文档中提取文本、表格和键值对。

- **AzureAiServicesSpeechToTextTool**：用于将语音转录为文本。

- **AzureAiServicesTextToSpeechTool**：用于将文本合成语音。

- **AzureAiServicesTextAnalyticsForHealthTool**：用于提取医疗实体。

首先，您需要设置一个 Azure 账户并创建一个 AI 服务资源。您可以按照[这里](https://learn.microsoft.com/en-us/azure/ai-services/multi-service-resource)的说明来创建一个资源。

然后，您需要获取资源的端点、密钥和区域，并将它们设置为环境变量。您可以在资源的“密钥和端点”页面找到它们。

```python
%pip install --upgrade --quiet azure-ai-formrecognizer > /dev/null
%pip install --upgrade --quiet azure-cognitiveservices-speech > /dev/null
%pip install --upgrade --quiet azure-ai-textanalytics > /dev/null
%pip install --upgrade --quiet azure-ai-vision-imageanalysis > /dev/null
```

```python
import os
os.environ["OPENAI_API_KEY"] = "sk-"
os.environ["AZURE_AI_SERVICES_KEY"] = ""
os.environ["AZURE_AI_SERVICES_ENDPOINT"] = ""
os.environ["AZURE_AI_SERVICES_REGION"] = ""
```

## 创建工具包

```python
from langchain_community.agent_toolkits import AzureAiServicesToolkit
toolkit = AzureAiServicesToolkit()
```

```python
[tool.name for tool in toolkit.get_tools()]
```

```output
['azure_ai_services_document_intelligence',
 'azure_ai_services_image_analysis',
 'azure_ai_services_speech_to_text',
 'azure_ai_services_text_to_speech',
 'azure_ai_services_text_analytics_for_health']
```

## 在代理中使用

```python
from langchain import hub
from langchain.agents import AgentExecutor, create_structured_chat_agent
from langchain_openai import OpenAI
```

```python
llm = OpenAI(temperature=0)
tools = toolkit.get_tools()
prompt = hub.pull("hwchase17/structured-chat-agent")
agent = create_structured_chat_agent(llm, tools, prompt)
agent_executor = AgentExecutor(
    agent=agent, tools=tools, verbose=True, handle_parsing_errors=True
)
```

```python
agent_executor.invoke(
    {
        "input": "What can I make with these ingredients? " + "https://images.openai.com/blob/9ad5a2ab-041f-475f-ad6a-b51899c50182/ingredients.png"
    }
)
```

```output
> 进入新的 AgentExecutor 链...
思考：我需要使用 azure_ai_services_image_analysis 工具来分析这些食材的图像。
操作：
```

{

  "action": "azure_ai_services_image_analysis",

  "action_input": "https://images.openai.com/blob/9ad5a2ab-041f-475f-ad6a-b51899c50182/ingredients.png"

}

```
标题：几个碗里的鸡蛋和面粉
对象：鸡蛋、鸡蛋、食物
标签：乳制品、成分、室内、增稠剂、食物、搅拌碗、粉末、面粉、鸡蛋、碗
操作：
```

{

  "action": "Final Answer",

  "action_input": "您可以用这些食材做蛋糕或其他烘焙食品。"

}

> 链结束。

```output
{'input': 'What can I make with these ingredients? https://images.openai.com/blob/9ad5a2ab-041f-475f-ad6a-b51899c50182/ingredients.png',
 'output': '您可以用这些食材做蛋糕或其他烘焙食品。'}
```

```python
tts_result = agent_executor.invoke({"input": "Tell me a joke and read it out for me."})
audio_file = tts_result.get("output")
```

```output
> 进入新的 AgentExecutor 链...
思考：我可以使用 Azure AI 服务的文本转语音 API 将文本转换为语音。
操作：
```

{

  "action": "azure_ai_services_text_to_speech",

  "action_input": "为什么科学家不相信原子？因为它们构成了一切。"

}

```
/tmp/tmpe48vamz0.wav
> 链结束。
```

```python
from IPython import display
audio = display.Audio(data=audio_file, autoplay=True, rate=22050)
display.display(audio)
```

```python
sample_input = """
患者是一位 54 岁的绅士，几个月来一直有进行性心绞痛的病史。
患者于今年 7 月进行了心脏导管检查，发现右冠状动脉完全闭塞，左主干病变 50% ，
家族中有冠状动脉疾病的强烈家族史，一位兄弟因心肌梗死在 52 岁时去世，
另一位兄弟做过冠状动脉旁路移植手术。患者于 2001 年 7 月做了一次压力超声心动图，
未显示心壁运动异常，但由于体形特殊，这是一项困难的研究。患者进行了六分钟的测试，
前侧壁导联有轻微 ST 下降，认为是由于疲劳和手腕疼痛引起的心绞痛等效症。由于患者症状加重，
家族史和左主干病变，右冠状动脉完全闭塞，被转诊接受开放性心脏手术血管重建手术。
列出所有诊断。
"""
agent_executor.invoke({"input": sample_input})
```

```output
> 进入新的 AgentExecutor 链...
思考：患者有进行性心绞痛的病史，家族中有冠状动脉疾病的强烈家族史，并且之前进行的心脏导管造影显示右冠状动脉完全闭塞，左主干病变50%。
行动：
```

{

  "action": "azure_ai_services_text_analytics_for_health",

  "action_input": "患者是一名54岁的绅士，过去几个月出现进行性心绞痛。患者于今年7月进行了心脏导管造影，结果显示右冠状动脉完全闭塞，左主干病变50%，家族中有强烈的冠状动脉疾病家族史，其中一名兄弟在52岁时死于心肌梗死，另一名兄弟则进行了冠状动脉搭桥手术。患者于2001年7月进行了负荷超声心动图检查，结果未显示心壁运动异常，但由于体形特殊，该检查较为困难。患者在进行检查时出现前侧壁导联ST段轻微压低，持续六分钟，可能由于疲劳和手腕疼痛引起，这是他的心绞痛等效症状。由于患者症状加重，家族史以及左主干病变和右冠状动脉完全闭塞的病史，被转诊进行开放性心脏手术血管重建治疗。"

该文本包含以下医疗实体：54岁是年龄类型的医疗实体，绅士是性别类型的医疗实体，进行性心绞痛是诊断类型的医疗实体，过去几个月是时间类型的医疗实体，心脏导管造影是检查名称类型的医疗实体，今年7月是时间类型的医疗实体，完全是状况限定词类型的医疗实体，闭塞是症状或体征类型的医疗实体，右冠状动脉是身体结构类型的医疗实体，50%是测量数值类型的医疗实体，左主干病变是诊断类型的医疗实体，家族是家庭关系类型的医疗实体，冠状动脉疾病是诊断类型的医疗实体，兄弟是家庭关系类型的医疗实体，死亡是诊断类型的医疗实体，52岁是年龄类型的医疗实体，心肌梗死是诊断类型的医疗实体，兄弟是家庭关系类型的医疗实体，冠状动脉搭桥手术是治疗名称类型的医疗实体，负荷超声心动图检查是检查名称类型的医疗实体，2001年7月是时间类型的医疗实体，心壁运动异常是症状或体征类型的医疗实体，体形特殊是症状或体征类型的医疗实体，六分钟是时间类型的医疗实体，轻微是状况限定词类型的医疗实体，前侧壁导联ST段轻微压低是症状或体征类型的医疗实体，疲劳是症状或体征类型的医疗实体，手腕疼痛是症状或体征类型的医疗实体，心绞痛是症状或体征类型的医疗实体，加重是病程类型的医疗实体，症状是症状或体征类型的医疗实体，家族是家庭关系类型的医疗实体，左主干病变是诊断类型的医疗实体，偶发是病程类型的医疗实体，右冠状动脉是身体结构类型的医疗实体，血管重建是治疗名称类型的医疗实体，开放性心脏手术是治疗名称类型的医疗实体"

行动：

```
{
  "action": "Final Answer",
  "action_input": "患者的诊断包括进行性心绞痛、右冠状动脉完全闭塞、左主干病变50%、冠状动脉疾病、心肌梗死，以及冠状动脉疾病家族史。"
}
> 链条完成。
```

**输出**：患者的诊断包括进行性心绞痛、右冠状动脉完全闭塞、左主干病变50%、冠状动脉疾病、心肌梗死以及家族史中有冠状动脉疾病。