# Label Studio

[Label Studio](https://labelstud.io/guide/get_started) 是一个开源的数据标注平台，为 LangChain 提供了灵活性，可以用于标记数据以对大型语言模型（LLMs）进行微调。它还可以用于准备自定义训练数据，并通过人类反馈收集和评估响应。

在本指南中，您将学习如何将 LangChain 流水线连接到 `Label Studio`，以便：

- 在单个 `Label Studio` 项目中汇总所有输入提示、对话和响应。这样可以将所有数据整合到一个地方，以便更容易地进行标记和分析。

- 优化提示和响应，创建用于监督微调（SFT）和通过人类反馈进行强化学习（RLHF）场景的数据集。标记的数据可用于进一步训练 LLM 以提高其性能。

- 通过人类反馈评估模型响应。`Label Studio` 提供了一个界面，供人类审查并对模型响应提供反馈，从而进行评估和迭代。

## 安装和设置

首先安装 Label Studio 和 Label Studio API 客户端的最新版本：

```python
%pip install --upgrade --quiet langchain label-studio label-studio-sdk langchain-openai
```

接下来，在命令行上运行 `label-studio`，以在 `http://localhost:8080` 启动本地 LabelStudio 实例。有关更多选项，请参阅[Label Studio 安装指南](https://labelstud.io/guide/install)。

您将需要一个令牌来进行 API 调用。

在浏览器中打开您的 LabelStudio 实例，转到 `Account & Settings > Access Token` 并复制密钥。

使用您的 LabelStudio URL、API 密钥和 OpenAI API 密钥设置环境变量：

```python
import os
os.environ["LABEL_STUDIO_URL"] = "<YOUR-LABEL-STUDIO-URL>"  # 例如 http://localhost:8080
os.environ["LABEL_STUDIO_API_KEY"] = "<YOUR-LABEL-STUDIO-API-KEY>"
os.environ["OPENAI_API_KEY"] = "<YOUR-OPENAI-API-KEY>"
```

## 收集 LLM 提示和响应

用于标注的数据存储在 Label Studio 中的项目中。每个项目由一个 XML 配置标识，其中详细说明了输入和输出数据的规格。

创建一个项目，以文本格式接收人类输入，并在文本区域中输出可编辑的 LLM 响应：

```xml
<View>
<Style>
    .prompt-box {
        background-color: white;
        border-radius: 10px;
        box-shadow: 0px 4px 6px rgba(0, 0, 0, 0.1);
        padding: 20px;
    }
</Style>
<View className="root">
    <View className="prompt-box">
        <Text name="prompt" value="$prompt"/>
    </View>
    <TextArea name="response" toName="prompt"
              maxSubmissions="1" editable="true"
              required="true"/>
</View>
<Header value="Rate the response:"/>
<Rating name="rating" toName="prompt"/>
</View>
```

1. 在 Label Studio 中创建项目，单击“创建”按钮。

2. 在“项目名称”字段中输入项目名称，例如 `My Project`。

3. 转到 `Labeling Setup > Custom Template`，并粘贴上面提供的 XML 配置。

您可以通过 `LabelStudioCallbackHandler` 将输入的 LLM 提示和输出的响应收集到 LabelStudio 项目中：

```python
from langchain_community.callbacks.labelstudio_callback import (
    LabelStudioCallbackHandler,
)
```

```python
from langchain_openai import OpenAI
llm = OpenAI(
    temperature=0, callbacks=[LabelStudioCallbackHandler(project_name="My Project")]
)
print(llm.invoke("Tell me a joke"))
```

在 Label Studio 中打开 `My Project`，您将看到提示、响应和模型名称等元数据。

## 收集聊天模型对话

您还可以在 Label Studio 中跟踪和显示完整的聊天对话，有能力评分和修改最后的响应：

1. 打开 Label Studio 并单击“创建”按钮。

2. 在“项目名称”字段中输入项目名称，例如 `New Project with Chat`。

3. 转到 Labeling Setup > Custom Template，并粘贴以下 XML 配置：

```xml
<View>
<View className="root">
     <Paragraphs name="dialogue"
               value="$prompt"
               layout="dialogue"
               textKey="content"
               nameKey="role"
               granularity="sentence"/>
  <Header value="Final response:"/>
    <TextArea name="response" toName="dialogue"
              maxSubmissions="1" editable="true"
              required="true"/>
</View>
<Header value="Rate the response:"/>
<Rating name="rating" toName="dialogue"/>
</View>
```

```python
from langchain_core.messages import HumanMessage, SystemMessage
from langchain_openai import ChatOpenAI
chat_llm = ChatOpenAI(
    callbacks=[
        LabelStudioCallbackHandler(
            mode="chat",
            project_name="New Project with Chat",
        )
    ]
)
llm_results = chat_llm.invoke(
    [
        SystemMessage(content="Always use a lot of emojis"),
        HumanMessage(content="Tell me a joke"),
    ]
)
```

在 Label Studio 中，打开“使用聊天创建新项目”。点击已创建的任务，以查看对话历史并编辑/标注回复。

## 自定义标注配置

您可以在 Label Studio 中修改默认的标注配置，以添加更多目标标签，比如回复情感、相关性，以及[其他类型的注释反馈](https://labelstud.io/tags/)。

可以通过 UI 添加新的标注配置：转到 `设置 > 标注界面`，并设置一个自定义配置，添加额外的标签，比如用于情感的 `Choices` 或用于相关性的 `Rating`。请注意，任何配置中都应包含 [`TextArea` 标签](https://labelstud.io/tags/textarea)，以显示 LLM 的回复。

另外，您也可以在创建项目之前的初始调用中指定标注配置：

```python
ls = LabelStudioCallbackHandler(
    project_config="""
<View>
<Text name="prompt" value="$prompt"/>
<TextArea name="response" toName="prompt"/>
<TextArea name="user_feedback" toName="prompt"/>
<Rating name="rating" toName="prompt"/>
<Choices name="sentiment" toName="prompt">
    <Choice value="Positive"/>
    <Choice value="Negative"/>
</Choices>
</View>
"""
)
```

请注意，如果项目不存在，将会使用指定的标注配置创建项目。

## 其他参数

`LabelStudioCallbackHandler` 接受几个可选参数：

- **api_key** - Label Studio API 密钥。覆盖环境变量 `LABEL_STUDIO_API_KEY`。

- **url** - Label Studio URL。覆盖 `LABEL_STUDIO_URL`，默认为 `http://localhost:8080`。

- **project_id** - 现有的 Label Studio 项目 ID。覆盖 `LABEL_STUDIO_PROJECT_ID`。将数据存储在该项目中。

- **project_name** - 如果未指定项目 ID，则为项目名称。创建一个新项目。默认为当前日期的 `"LangChain-%Y-%m-%d"` 格式。

- **project_config** - [自定义标注配置](#自定义标注配置)

- **mode**：使用此快捷方式从头开始创建目标配置：

   - `"prompt"` - 单个提示，单个回复。默认。

   - `"chat"` - 多轮对话模式。