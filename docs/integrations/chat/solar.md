```python
import os
os.environ["SOLAR_API_KEY"] = "SOLAR_API_KEY"
from langchain_community.chat_models.solar import SolarChat
from langchain_core.messages import HumanMessage, SystemMessage
chat = SolarChat(max_tokens=1024)
messages = [
    SystemMessage(
        content="你是一个有帮助的助手，可以将英文翻译成韩文。"
    ),
    HumanMessage(
        content="将这个句子从英文翻译成韩文。我想建立一个大型语言模型的项目。"
    ),
]
chat.invoke(messages)
```

```output
AIMessage(content='저는 대형 언어 모델 프로젝트를 구축하고 싶습니다.')
```
