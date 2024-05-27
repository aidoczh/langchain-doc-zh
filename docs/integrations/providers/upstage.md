# Upstage
[Upstage](https://upstage.ai) 是一家领先的人工智能（AI）公司，专门致力于提供高于人类水平的LLM组件。
## Solar LLM
**Solar Mini Chat** 是一个快速而强大的先进大型语言模型，专注于英语和韩语。它经过特别调整，用于多轮对话目的，显示出在多轮对话或需要理解长篇上下文（如RAG（检索增强生成））等各种自然语言处理任务中，与其他类似规模的模型相比表现出色。这种特别调整使其能够更有效地处理更长的对话，使其特别适用于交互式应用程序。
除了Solar之外，Upstage还提供了用于真实世界的RAG（检索增强生成）的功能，例如**Groundedness Check**和**Layout Analysis**。
## 安装和设置
安装 `langchain-upstage` 包：
```bash
pip install -qU langchain-core langchain-upstage
```
获取[API密钥](https://console.upstage.ai)并设置环境变量 `UPSTAGE_API_KEY`。
## Upstage LangChain 集成
| API | 描述 | 导入 | 示例用法 |
| --- | --- | --- | --- |
| Chat | 使用Solar Mini Chat构建助手 | `from langchain_upstage import ChatUpstage` | [前往](../../chat/upstage) |
| 文本嵌入 | 将字符串嵌入向量 | `from langchain_upstage import UpstageEmbeddings` | [前往](../../text_embedding/upstage) |
| Groundedness Check | 验证助手回应的基础性 | `from langchain_upstage import UpstageGroundednessCheck` | [前往](../../tools/upstage_groundedness_check) |
| Layout Analysis | 序列化带有表格和图表的文档 | `from langchain_upstage import UpstageLayoutAnalysisLoader` | [前往](../../document_loaders/upstage) |
更多功能的详细信息，请参阅[文档](https://developers.upstage.ai/)。
## 快速示例
### 环境设置
```python
import os
os.environ["UPSTAGE_API_KEY"] = "YOUR_API_KEY"
```
### 对话
```python
from langchain_upstage import ChatUpstage
chat = ChatUpstage()
response = chat.invoke("你好，你好吗？")
print(response)
```
### 文本嵌入
```python
from langchain_upstage import UpstageEmbeddings
embeddings = UpstageEmbeddings()
doc_result = embeddings.embed_documents(
    ["山姆是一名老师。", "这是另一个文档"]
)
print(doc_result)
query_result = embeddings.embed_query("山姆做什么？")
print(query_result)
```
### Groundedness Check
```python
from langchain_upstage import UpstageGroundednessCheck
groundedness_check = UpstageGroundednessCheck()
request_input = {
    "context": "毛纳凯山是夏威夷岛上的一座死火山。它的山顶海拔4207.3米，是夏威夷的最高点，也是地球上岛屿的第二高峰。",
    "answer": "毛纳凯山高5207.3米。",
}
response = groundedness_check.invoke(request_input)
print(response)
```
### 布局分析
```python
from langchain_upstage import UpstageLayoutAnalysisLoader
file_path = "/文件路径/到/你的/文件.pdf"
layzer = UpstageLayoutAnalysisLoader(file_path, split="page")
# 为了提高内存效率，考虑使用lazy_load方法逐页加载文档。
docs = layzer.load()  # or layzer.lazy_load()
for doc in docs[:3]:
    print(doc)
```