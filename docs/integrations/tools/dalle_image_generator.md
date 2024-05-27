# Dall-E 图像生成器

>[OpenAI Dall-E](https://openai.com/dall-e-3) 是由 `OpenAI` 开发的文本到图像模型，使用深度学习方法从自然语言描述生成数字图像，称为 "提示"。

这个笔记本展示了如何使用 OpenAI LLM 合成的提示生成图像。这些图像是使用 `Dall-E` 生成的，它使用与 LLM 相同的 OpenAI API 密钥。

```python
# 如果您想在笔记本中显示图像，则需要执行此操作
%pip install --upgrade --quiet  opencv-python scikit-image
```

```python
import os
from langchain_openai import OpenAI
os.environ["OPENAI_API_KEY"] = "<your-key-here>"
```

## 作为链运行

```python
from langchain.chains import LLMChain
from langchain_community.utilities.dalle_image_generator import DallEAPIWrapper
from langchain_core.prompts import PromptTemplate
from langchain_openai import OpenAI
llm = OpenAI(temperature=0.9)
prompt = PromptTemplate(
    input_variables=["image_desc"],
    template="生成一个详细的提示，以根据以下描述生成图像：{image_desc}",
)
chain = LLMChain(llm=llm, prompt=prompt)
```

```python
image_url = DallEAPIWrapper().run(chain.run("在一个闹鬼的博物馆里过万圣节之夜"))
```

```python
image_url
```

```output
'https://oaidalleapiprodscus.blob.core.windows.net/private/org-i0zjYONU3PemzJ222esBaAzZ/user-f6uEIOFxoiUZivy567cDSWni/img-i7Z2ZxvJ4IbbdAiO6OXJgS3v.png?st=2023-08-11T14%3A03%3A14Z&se=2023-08-11T16%3A03%3A14Z&sp=r&sv=2021-08-06&sr=b&rscd=inline&rsct=image/png&skoid=6aaadede-4fb3-4698-a8f6-684d7786b067&sktid=a48cca56-e6da-484e-a814-9c849652bcb3&skt=2023-08-10T20%3A58%3A32Z&ske=2023-08-11T20%3A58%3A32Z&sks=b&skv=2021-08-06&sig=/sECe7C0EAq37ssgBm7g7JkVIM/Q1W3xOstd0Go6slA%3D'
```

```python
# 您可以点击上面的链接显示图像
# 或者您可以尝试以下选项在此笔记本中内联显示图像
try:
    import google.colab
    IN_COLAB = True
except ImportError:
    IN_COLAB = False
if IN_COLAB:
    from google.colab.patches import cv2_imshow  # 用于图像显示
    from skimage import io
    image = io.imread(image_url)
    cv2_imshow(image)
else:
    import cv2
    from skimage import io
    image = io.imread(image_url)
    cv2.imshow("image", image)
    cv2.waitKey(0)  # 等待键盘输入
    cv2.destroyAllWindows()
```

## 作为带有代理的工具运行

```python
from langchain.agents import initialize_agent, load_tools
tools = load_tools(["dalle-image-generator"])
agent = initialize_agent(tools, llm, agent="zero-shot-react-description", verbose=True)
output = agent.run("创建一个在闹鬼博物馆里过万圣节之夜的图像")
```

```output
> 进入新的 AgentExecutor 链...
如何将此描述转换为图像的最佳方法？
操作：Dall-E 图像生成器
操作输入：一个在闹鬼的博物馆里过万圣节之夜的恐怖场景https://oaidalleapiprodscus.blob.core.windows.net/private/org-rocrupyvzgcl4yf25rqq6d1v/user-WsxrbKyP2c8rfhCKWDyMfe8N/img-ogKfqxxOS5KWVSj4gYySR6FY.png?st=2023-01-31T07%3A38%3A25Z&se=2023-01-31T09%3A38%3A25Z&sp=r&sv=2021-08-06&sr=b&rscd=inline&rsct=image/png&skoid=6aaadede-4fb3-4698-a8f6-684d7786b067&sktid=a48cca56-e6da-484e-a814-9c849652bcb3&skt=2023-01-30T22%3A19%3A36Z&ske=2023-01-31T22%3A19%3A36Z&sks=b&skv=2021-08-06&sig=XsomxxBfu2CP78SzR9lrWUlbask4wBNnaMsHamy4VvU%3D
观察：https://oaidalleapiprodscus.blob.core.windows.net/private/org-rocrupyvzgcl4yf25rqq6d1v/user-WsxrbKyP2c8rfhCKWDyMfe8N/img-ogKfqxxOS5KWVSj4gYySR6FY.png?st=2023-01-31T07%3A38%3A25Z&se=2023-01-31T09%3A38%3A25Z&sp=r&sv=2021-08-06&sr=b&rscd=inline&rsct=image/png&skoid=6aaadede-4fb3-4698-a8f6-684d7786b067&sktid=a48cca56-e6da-484e-a814-9c849652bcb3&skt=2023-01-30T22%3A19%3A36Z&ske=2023-01-31T22%3A19%3A36Z&sks=b&skv=2021-08-06&sig=XsomxxBfu2CP78SzR9lrWUlbask4wBNnaMsHamy4VvU%3D
思考：有了生成的图像，我现在可以给出最终答案。
最终答案：一个在闹鬼博物馆里过万圣节之夜的图像可以在这里看到：https://oaidalleapiprodscus.blob.core.windows.net/private/org-rocrupyvzgcl4yf25rqq6d1v/user-WsxrbKyP2c8rfhCKWDyMfe8N/img-ogKfqxxOS5KWVSj4gYySR6FY.png?st=2023-01-31T07%3A38%3A25Z&se=2023-01-31T09%3A38%3A25Z&sp=r&sv=2021-08-06&sr=b&rscd=inline&rsct=image/png&skoid=6aaadede-4fb3-4698-a8f6-684d7786b067&sktid=a48cca56-e6da-484e-a814-9c849652bcb3&skt=2023-01-30T22
> 链结束。
```