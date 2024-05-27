# 如何检查可运行对象
:::info 先决条件
本指南假设您熟悉以下概念：
- [LangChain 表达式语言（LCEL）](/docs/concepts/#langchain-expression-language)
- [链接可运行对象](/docs/how_to/sequence/)
:::
一旦您使用[LangChain 表达式语言](/docs/concepts/#langchain-expression-language)创建了一个可运行对象，您可能经常想要检查它以更好地了解正在发生的情况。本文介绍了一些方法。
本指南展示了一些以编程方式内省链的内部步骤的方法。如果您对调试链中的问题感兴趣，请参阅[此部分](/docs/how_to/debugging)。
首先，让我们创建一个示例链。我们将创建一个用于检索的链：
```python
%pip install -qU langchain langchain-openai faiss-cpu tiktoken
```
```python
from langchain_community.vectorstores import FAISS
from langchain_core.output_parsers import StrOutputParser
from langchain_core.prompts import ChatPromptTemplate
from langchain_core.runnables import RunnablePassthrough
from langchain_openai import ChatOpenAI, OpenAIEmbeddings
vectorstore = FAISS.from_texts(
    ["harrison worked at kensho"], embedding=OpenAIEmbeddings()
)
retriever = vectorstore.as_retriever()
template = """Answer the question based only on the following context:
{context}
Question: {question}
"""
prompt = ChatPromptTemplate.from_template(template)
model = ChatOpenAI()
chain = (
    {"context": retriever, "question": RunnablePassthrough()}
    | prompt
    | model
    | StrOutputParser()
)
```
## 获取图形
您可以使用`get_graph()`方法获取可运行对象的图形表示：
```python
chain.get_graph()
```
## 打印图形
虽然这不是非常易读，但您可以使用`print_ascii()`方法以更容易理解的方式显示该图形：
```python
chain.get_graph().print_ascii()
```
```output
           +---------------------------------+         
| Parallel<context,question>Input |
           +---------------------------------+         
                    **               **                
                 ***                   ***             
               **                         **           
+----------------------+              +-------------+  
| VectorStoreRetriever |              | Passthrough |
+----------------------+              +-------------+  
                    **               **                
                      ***         ***                  
                         **     **                     
           +----------------------------------+        
| Parallel<context,question>Output |
           +----------------------------------+        
                             *                         
                             *                         
                             *                         
                  +--------------------+               
| ChatPromptTemplate |
                  +--------------------+               
                             *                         
                             *                         
                             *                         
                      +------------+                   
| ChatOpenAI |
                      +------------+                   
                             *                         
                             *                         
                             *                         
                   +-----------------+                 
| StrOutputParser |
                   +-----------------+                 
                             *                         
                             *                         
                             *                         
                +-----------------------+              
| StrOutputParserOutput |
                +-----------------------+
```
## 获取提示
您可能希望使用`get_prompts()`方法查看链中使用的提示：
```python
chain.get_prompts()
```
```output
[ChatPromptTemplate(input_variables=['context', 'question'], messages=[HumanMessagePromptTemplate(prompt=PromptTemplate(input_variables=['context', 'question'], template='Answer the question based only on the following context:\n{context}\n\nQuestion: {question}\n'))])]
```
## 下一步
您现在已经学会了如何内省您组合的 LCEL 链。
接下来，请查看本节中有关可运行对象的其他操作指南，或者查看有关[调试链](/docs/how_to/debugging)的相关操作指南。