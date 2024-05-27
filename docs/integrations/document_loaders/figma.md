# Figma

[Figma](https://www.figma.com/) 是一个用于界面设计的协作式 Web 应用程序。

本笔记涵盖了如何将数据从 `Figma` REST API 加载到可被 LangChain 吸收的格式中，以及代码生成的示例用法。

```python
import os
from langchain.indexes import VectorstoreIndexCreator
from langchain_community.document_loaders.figma import FigmaFileLoader
from langchain_core.prompts.chat import (
    ChatPromptTemplate,
    HumanMessagePromptTemplate,
    SystemMessagePromptTemplate,
)
from langchain_openai import ChatOpenAI
```

Figma API 需要访问令牌（access token）、节点 ID 和文件密钥。

文件密钥可以从 URL 中提取。https://www.figma.com/file/{filekey}/sampleFilename

节点 ID 也可以在 URL 中找到。单击任何内容，查找 '?node-id={node_id}' 参数。

访问令牌的说明请参阅 Figma 帮助中心文章：https://help.figma.com/hc/en-us/articles/8085703771159-Manage-personal-access-tokens

```python
figma_loader = FigmaFileLoader(
    os.environ.get("ACCESS_TOKEN"),
    os.environ.get("NODE_IDS"),
    os.environ.get("FILE_KEY"),
)
```

```python
# 有关更多详细信息，请参阅 https://python.langchain.com/en/latest/modules/data_connection/getting_started.html
index = VectorstoreIndexCreator().from_loaders([figma_loader])
figma_doc_retriever = index.vectorstore.as_retriever()
```

```python
def generate_code(human_input):
    # 我不知道 Jon Carmack 的事情是否会让代码更好。结果可能有所不同。
    # 有关聊天信息，请参阅 https://python.langchain.com/en/latest/modules/models/chat/getting_started.html
    system_prompt_template = """You are expert coder Jon Carmack. Use the provided design context to create idiomatic HTML/CSS code as possible based on the user request.
    Everything must be inline in one file and your response must be directly renderable by the browser.
    Figma file nodes and metadata: {context}"""
    human_prompt_template = "Code the {text}. Ensure it's mobile responsive"
    system_message_prompt = SystemMessagePromptTemplate.from_template(
        system_prompt_template
    )
    human_message_prompt = HumanMessagePromptTemplate.from_template(
        human_prompt_template
    )
    # 删除 gpt-4 model_name 以使用默认的 gpt-3.5 turbo 以获得更快的结果
    gpt_4 = ChatOpenAI(temperature=0.02, model_name="gpt-4")
    # 如果需要过滤较长的文档，请使用 retriever 的 'get_relevant_documents' 方法
    relevant_nodes = figma_doc_retriever.invoke(human_input)
    conversation = [system_message_prompt, human_message_prompt]
    chat_prompt = ChatPromptTemplate.from_messages(conversation)
    response = gpt_4(
        chat_prompt.format_prompt(
            context=relevant_nodes, text=human_input
        ).to_messages()
    )
    return response
```

```python
response = generate_code("page top header")
```

`response.content` 返回以下内容：

```
<!DOCTYPE html>\n<html lang="en">\n<head>\n    <meta charset="UTF-8">\n    <meta name="viewport" content="width=device-width, initial-scale=1.0">\n    <style>\n        @import url(\'https://fonts.googleapis.com/css2?family=DM+Sans:wght@500;700&family=Inter:wght@600&display=swap\');\n\n        body {\n            margin: 0;\n            font-family: \'DM Sans\', sans-serif;\n        }\n\n        .header {\n            display: flex;\n            justify-content: space-between;\n            align-items: center;\n            padding: 20px;\n            background-color: #fff;\n            box-shadow: 0 2px 4px rgba(0, 0, 0, 0.1);\n        }\n\n        .header h1 {\n            font-size: 16px;\n            font-weight: 700;\n            margin: 0;\n        }\n\n        .header nav {\n            display: flex;\n            align-items: center;\n        }\n\n        .header nav a {\n            font-size: 14px;\n            font-weight: 500;\n            text-decoration: none;\n            color: #000;\n            margin-left: 20px;\n        }\n\n        @media (max-width: 768px) {\n            .header nav {\n                display: none;\n            }\n        }\n    </style>\n</head>\n<body>\n    <header class="header">\n        <h1>Company Contact</h1>\n        <nav>\n            <a href="#">Lorem Ipsum</a>\n            <a href="#">Lorem Ipsum</a>\n            <a href="#">Lorem Ipsum</a>\n        </nav>\n    </header>\n</body>\n</html>
```