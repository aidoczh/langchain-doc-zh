---

关键词: [双子座, 谷歌生成式人工智能, 双子座-专业版]

---

# 谷歌人工智能

使用[谷歌生成式人工智能](https://developers.generativeai.google/)模型与 Langchain 的指南。注意：这与谷歌云 Vertex AI 的[集成](/docs/integrations/llms/google_vertex_ai_palm)是分开的。

## 设置

要使用谷歌生成式人工智能，您必须安装 `langchain-google-genai` Python 包并生成 API 密钥。[阅读更多详情](https://developers.generativeai.google/)。

```python
%pip install --upgrade --quiet  langchain-google-genai
```

```python
from langchain_google_genai import GoogleGenerativeAI
```

```python
from getpass import getpass
api_key = getpass()
```

```python
llm = GoogleGenerativeAI(model="models/text-bison-001", google_api_key=api_key)
print(
    llm.invoke(
        "Python 作为一种编程语言有哪些优缺点？"
    )
)
```

```output
**Python 的优点：**
* **易学性：** Python 是一种非常易学的编程语言，即使对初学者来说也是如此。它的语法简单直观，而且有很多资源可帮助您入门。
* **多功能性：** Python 可用于各种任务，包括 Web 开发、数据科学和机器学习。对初学者来说也是一个不错的选择，因为它可用于各种项目，您可以先学习基础知识，然后转向更复杂的任务。
* **高级别：** Python 是一种高级编程语言，这意味着它比其他编程语言更接近人类语言。这使得阅读和理解变得更容易，对初学者来说是一个很大的优势。
* **开源：** Python 是一种开源编程语言，这意味着可以免费使用，并且有很多资源可帮助您学习它。
* **社区：** Python 有一个庞大活跃的开发者社区，这意味着有很多人可以在您遇到问题时为您提供帮助。
**Python 的缺点：**
* **速度慢：** Python 是一种相对较慢的编程语言，与某些其他语言（如 C++）相比。如果您正在处理计算密集型任务，则这可能是一个缺点。
* **性能不佳：** Python 的性能不如某些其他编程语言（如 C++ 或 Java）。如果您正在处理需要高性能的项目，则这可能是一个缺点。
* **动态类型：** Python 是一种动态类型的编程语言，这意味着变量的类型可以在运行时更改。如果您需要确保代码是类型安全的，则这可能是一个缺点。
* **非托管内存：** Python 使用垃圾回收系统来管理内存。如果您需要更多地控制内存管理，则这可能是一个缺点。
总的来说，Python 对于初学者来说是一种非常好的编程语言。易学、多功能，并且有一个庞大的开发者社区。然而，重要的是要意识到它的局限性，比如性能较慢和缺乏性能。
```

```python
llm = GoogleGenerativeAI(model="gemini-pro", google_api_key=api_key)
print(
    llm.invoke(
        "Python 作为一种编程语言有哪些优缺点？"
    )
)
```

```output
**优点：**
* **简单易读：** Python 以其简单易读的语法而闻名，这使得它对初学者易于理解，并减少了出错的可能性。它使用缩进来定义代码块，使代码结构清晰且视觉上吸引。
* **多功能性：** Python 是一种通用语言，意味着它可用于各种任务，包括 Web 开发、数据科学、机器学习和桌面应用程序。这种多功能性使其成为各种项目和行业的热门选择。
* **庞大社区：** Python 拥有庞大而活跃的开发者社区，这促进了其增长和流行。这个社区提供了广泛的文档、教程和开源库，使得 Python 开发者可以轻松找到支持和资源。
* **丰富的库：** Python 提供了丰富的库和框架，用于各种任务，如数据分析（NumPy、Pandas）、Web 开发（Django、Flask）、机器学习（Scikit-learn、TensorFlow）等。这些库提供了预构建的函数和模块，使开发人员能够快速高效地解决常见问题。
* **跨平台支持：** Python 是跨平台的，可以在各种操作系统上运行，包括 Windows、macOS 和 Linux。这使开发人员能够编写可轻松共享和在不同平台上使用的代码。
**缺点：**
* **速度和性能：** 由于 Python 是解释性语言，通常比像 C++ 或 Java 这样的编译语言慢，这可能对性能密集型任务（如实时系统或大量数值计算）构成不利。
* **内存使用：** 与编译语言相比，Python 程序往往会消耗更多内存。这是因为 Python 使用动态内存分配系统，这可能导致内存碎片化和更高的内存使用。
* **缺乏静态类型：** Python 是一种动态类型语言，这意味着变量的数据类型不是明确定义的。这可能使在开发过程中检测类型错误变得具有挑战性，这可能导致运行时出现意外行为或错误。
* **GIL（全局解释器锁）：** Python 使用全局解释器锁（GIL）来确保一次只有一个线程可以执行 Python 字节码。这可能限制 Python 程序的可伸缩性和并行性，特别是在多线程或多进程场景中。
* **包管理：** 虽然 Python 有庞大的库和包生态系统，但管理依赖关系和包版本可能具有挑战性。Python 包索引（PyPI）是 Python 包的官方存储库，但确保兼容性并避免不同版本包之间的冲突可能会很困难。
```

## 在链式调用中

```python
from langchain_core.prompts import PromptTemplate
```

```python
template = """Question: {question}
Answer: Let's think step by step."""
prompt = PromptTemplate.from_template(template)
chain = prompt | llm
question = "How much is 2+2?"
print(chain.invoke({"question": question}))
```

```output
4
```

## 流式调用

```python
import sys
for chunk in llm.stream("Tell me a short poem about snow"):
    sys.stdout.write(chunk)
    sys.stdout.flush()
```

```output
在冬季的拥抱中，一场寂静的芭蕾，
雪花飘落，一场天上的表演。
轻声细语，它们轻轻飘落，
一片白色的毯子，覆盖一切。
以温柔的优雅，它们描绘着大地，
将世界变成冬季仙境。
树木身披冰雪的华丽，
闪闪发光的壮观景象，令人叹为观止。
雪花旋转，像舞台上的舞者，
创造出交响乐，一幅冬季的画卷。
它们的悄悄细语，一曲甜美的小夜曲，
当它们舞动旋转，雪花瀑布。
在黎明的寂静中，一个寒冷的清晨，
雪花闪闪发光，像重生的钻石。
每片雪花独一无二，各具设计，
是神的杰作。
让我们陶醉在这冬日的幸福中，
当雪花飘落，带着温柔的吻。
因为在它们的拥抱中，我们找到深深的平静，
一个冰冻的世界，到处都是魔法。
```

### 安全设置

Gemini 模型具有默认的安全设置，可以被覆盖。如果您的模型收到大量“安全警告”，您可以尝试调整模型的 `safety_settings` 属性。例如，要关闭危险内容的安全阻止，您可以构建您的 LLM 如下：

```python
from langchain_google_genai import GoogleGenerativeAI, HarmBlockThreshold, HarmCategory
llm = GoogleGenerativeAI(
    model="gemini-pro",
    google_api_key=api_key,
    safety_settings={
        HarmCategory.HARM_CATEGORY_DANGEROUS_CONTENT: HarmBlockThreshold.BLOCK_NONE,
    },
)
```

要获取可用的类别和阈值的枚举，请参阅 Google 的 [安全设置类型](https://ai.google.dev/api/python/google/generativeai/types/SafetySettingDict)。