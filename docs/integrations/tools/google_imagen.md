# 谷歌图像

[Vertex AI 上的 Imagen](https://cloud.google.com/vertex-ai/generative-ai/docs/image/overview) 将谷歌最先进的图像生成人工智能能力带给应用开发者。使用 Imagen on Vertex AI，应用开发者可以构建下一代人工智能产品，通过 AI 生成在几秒钟内将用户的想象力转化为高质量的视觉资产。

使用 Langchain 上的 Imagen，您可以执行以下任务：

- [VertexAIImageGeneratorChat](#image-generation)：仅使用文本提示生成新颖图像（文本到图像的 AI 生成）。

- [VertexAIImageEditorChat](#image-editing)：使用文本提示编辑整个上传或生成的图像。

- [VertexAIImageCaptioning](#image-captioning)：通过视觉说明获取图像的文本描述。

- [VertexAIVisualQnAChat](#visual-question-answering-vqa)：通过视觉问答（VQA）获取关于图像的问题的答案。

    * 注意：目前我们仅支持视觉问答（VQA）的单轮对话。

## 图像生成

仅使用文本提示生成新颖图像（文本到图像的 AI 生成）

```python
from langchain_core.messages import AIMessage, HumanMessage
from langchain_google_vertexai.vision_models import VertexAIImageGeneratorChat
```

```python
# 创建图像生成模型对象
generator = VertexAIImageGeneratorChat()
```

```python
messages = [HumanMessage(content=["a cat at the beach"])]
response = generator.invoke(messages)
```

```python
# 查看生成的图像
generated_image = response.content[0]
```

```python
import base64
import io
from PIL import Image
# 解析响应对象以获取图像的 base64 字符串
img_base64 = generated_image["image_url"]["url"].split(",")[-1]
# 将 base64 字符串转换为图像
img = Image.open(io.BytesIO(base64.decodebytes(bytes(img_base64, "utf-8"))))
# 查看图像
img
```

![](/img/google_imagen_1.png)

## 图像编辑

使用文本提示编辑整个上传或生成的图像。

### 编辑生成的图像

```python
from langchain_core.messages import AIMessage, HumanMessage
from langchain_google_vertexai.vision_models import (
    VertexAIImageEditorChat,
    VertexAIImageGeneratorChat,
)
```

```python
# 创建图像生成模型对象
generator = VertexAIImageGeneratorChat()
# 为图像提供文本输入
messages = [HumanMessage(content=["a cat at the beach"])]
# 调用模型生成图像
response = generator.invoke(messages)
# 从响应中读取图像对象
generated_image = response.content[0]
```

```python
# 创建图像编辑模型对象
editor = VertexAIImageEditorChat()
```

```python
# 为编辑编写提示，并传递“生成的图像”
messages = [HumanMessage(content=[generated_image, "a dog at the beach "])]
# 调用模型编辑图像
editor_response = editor.invoke(messages)
```

```python
import base64
import io
from PIL import Image
# 解析响应对象以获取图像的 base64 字符串
edited_img_base64 = editor_response.content[0]["image_url"]["url"].split(",")[-1]
# 将 base64 字符串转换为图像
edited_img = Image.open(
    io.BytesIO(base64.decodebytes(bytes(edited_img_base64, "utf-8")))
)
# 查看图像
edited_img
```

![](/img/google_imagen_2.png)

## 图像说明

```python
from langchain_google_vertexai import VertexAIImageCaptioning
# 初始化图像说明对象
model = VertexAIImageCaptioning()
```

注意：我们在[图像生成部分](#image-generation)使用了生成的图像

```python
# 使用在图像生成部分生成的图像
img_base64 = generated_image["image_url"]["url"]
response = model.invoke(img_base64)
print(f"生成的说明：{response}")
# 将 base64 字符串转换为图像
img = Image.open(
    io.BytesIO(base64.decodebytes(bytes(img_base64.split(",")[-1], "utf-8")))
)
# 显示图像
img
```

```output
生成的说明：a cat sitting on the beach looking at the camera
```

![](/img/google_imagen_3.png)

## 视觉问答（VQA）

```python
from langchain_google_vertexai import VertexAIVisualQnAChat
model = VertexAIVisualQnAChat()
```

注意：我们在[图像生成部分](#image-generation)使用了生成的图像

```python
question = "图像中显示了什么动物？"
response = model.invoke(
    input=[
        HumanMessage(
            content=[
                {"type": "image_url", "image_url": {"url": img_base64}},
                question,
            ]
        )
    ]
)
print(f"问题：{question}\n答案：{response.content}")
# 将 base64 字符串转换为图像
img = Image.open(
    io.BytesIO(base64.decodebytes(bytes(img_base64.split(",")[-1], "utf-8")))
)
# 显示图像
img
```

```output
问题：图像中显示了什么动物？
答案：猫
```

![](/img/google_imagen_3.png)