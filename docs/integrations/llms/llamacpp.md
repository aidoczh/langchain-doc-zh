

# Llama.cpp

[llama-cpp-python](https://github.com/abetlen/llama-cpp-python) 是 [llama.cpp](https://github.com/ggerganov/llama.cpp) 的 Python 绑定。

它支持对 [许多 LLMs 模型](https://github.com/ggerganov/llama.cpp#description) 进行推理，这些模型可以在 [Hugging Face](https://huggingface.co/TheBloke) 上访问。

本笔记介绍了如何在 LangChain 中运行 `llama-cpp-python`。

**注意：新版本的 `llama-cpp-python` 使用 GGUF 模型文件（参见[这里](https://github.com/abetlen/llama-cpp-python/pull/633)）。**

这是一个重大变更。

要将现有的 GGML 模型转换为 GGUF，您可以在 [llama.cpp](https://github.com/ggerganov/llama.cpp) 中运行以下命令：

```python
python ./convert-llama-ggmlv3-to-gguf.py --eps 1e-5 --input models/openorca-platypus2-13b.ggmlv3.q4_0.bin --output models/openorca-platypus2-13b.gguf.q4_0.bin
```

## 安装

有不同的选项可供安装 llama-cpp 包：

- 仅使用 CPU

- CPU + GPU（使用多个 BLAS 后端之一）

- Metal GPU（搭载 Apple Silicon 芯片的 MacOS）

### 仅 CPU 安装

```python
%pip install --upgrade --quiet llama-cpp-python
```

### 使用 OpenBLAS / cuBLAS / CLBlast 安装

`llama.cpp` 支持多个 BLAS 后端以加快处理速度。使用 `FORCE_CMAKE=1` 环境变量来强制使用 cmake 并为所需的 BLAS 后端安装 pip 包（[来源](https://github.com/abetlen/llama-cpp-python#installation-with-openblas--cublas--clblast)）。

使用 cuBLAS 后端的示例安装：

```python
!CMAKE_ARGS="-DLLAMA_CUBLAS=on" FORCE_CMAKE=1 pip install llama-cpp-python
```

**重要提示**：如果您已经安装了仅 CPU 版本的包，您需要从头开始重新安装。考虑以下命令：

```python
!CMAKE_ARGS="-DLLAMA_CUBLAS=on" FORCE_CMAKE=1 pip install --upgrade --force-reinstall llama-cpp-python --no-cache-dir
```

### 使用 Metal 安装

`llama.cpp` 支持 Apple Silicon 优化 - 通过 ARM NEON、Accelerate 和 Metal 框架进行优化。使用 `FORCE_CMAKE=1` 环境变量来强制使用 cmake 并为 Metal 支持安装 pip 包（[来源](https://github.com/abetlen/llama-cpp-python/blob/main/docs/install/macos.md)）。

带有 Metal 支持的示例安装：

```python
!CMAKE_ARGS="-DLLAMA_METAL=on" FORCE_CMAKE=1 pip install llama-cpp-python
```

**重要提示**：如果您已经安装了仅 CPU 版本的包，您需要从头开始重新安装。考虑以下命令：

```python
!CMAKE_ARGS="-DLLAMA_METAL=on" FORCE_CMAKE=1 pip install --upgrade --force-reinstall llama-cpp-python --no-cache-dir
```

### 使用 Windows 安装

通过从源代码编译来安装 `llama-cpp-python` 库是稳定的。您可以遵循存储库本身中的大部分说明，但也有一些可能有用的特定于 Windows 的说明。

安装 `llama-cpp-python` 的要求：

- git

- python

- cmake

- Visual Studio Community（确保您按照以下设置安装）

    - 使用 C++ 的桌面开发

    - Python 开发

    - 使用 C++ 的嵌入式 Linux 开发

1. 递归克隆 git 子模块以获取 `llama.cpp` 子模块

```python
git clone --recursive -j8 https://github.com/abetlen/llama-cpp-python.git
```

2. 打开命令提示符并设置以下环境变量。

```python
set FORCE_CMAKE=1
set CMAKE_ARGS=-DLLAMA_CUBLAS=OFF
```

如果您有 NVIDIA GPU，请确保 `DLLAMA_CUBLAS` 设置为 `ON`

#### 编译和安装

现在您可以进入 `llama-cpp-python` 目录并安装包

```python
python -m pip install -e .
```

**重要提示**：如果您已经安装了仅 CPU 版本的包，您需要从头开始重新安装。考虑以下命令：

```python
!python -m pip install -e . --force-reinstall --no-cache-dir
```

## 使用

确保您遵循所有说明来[安装所有必要的模型文件](https://github.com/ggerganov/llama.cpp)。

您不需要 `API_TOKEN`，因为您将在本地运行 LLM。

值得了解的是，哪些模型适合在所需的机器上使用。

[Hugging Face](https://huggingface.co/TheBloke) 的模型有一个`提供的文件`部分，展示了运行不同量化大小和方法的模型所需的 RAM（例如：[Llama2-7B-Chat-GGUF](https://huggingface.co/TheBloke/Llama-2-7b-Chat-GGUF#provided-files)）。

这个[github 问题](https://github.com/facebookresearch/llama/issues/425)也是找到适合您机器的正确模型的相关内容。

```python
from langchain_community.llms import LlamaCpp
from langchain_core.callbacks import CallbackManager, StreamingStdOutCallbackHandler
from langchain_core.prompts import PromptTemplate
```

**考虑使用适合您模型的模板！查看 Hugging Face 等网站上的模型页面，以获取正确的提示模板。**

```python
template = """问题：{question}
回答：让我们逐步解决这个问题，确保我们得到正确的答案。"""
prompt = PromptTemplate.from_template(template)
```

```python
# 回调支持逐标记流式处理
callback_manager = CallbackManager([StreamingStdOutCallbackHandler()])
```

### CPU

使用 LLaMA 2 7B 模型的示例

```python
# 确保模型路径对您的系统是正确的！
llm = LlamaCpp(
    model_path="/Users/rlm/Desktop/Code/llama.cpp/models/openorca-platypus2-13b.gguf.q4_0.bin",
    temperature=0.75,
    max_tokens=2000,
    top_p=1,
    callback_manager=callback_manager,
    verbose=True,  # 需要传递给回调管理器的详细信息
)
```

```python
question = """
问题：Stephen Colbert 和 John Oliver 之间的说唱大战
"""
llm.invoke(question)
```

```output
Stephen Colbert:
Yo, John, I heard you've been talkin' smack about me on your show.
Let me tell you somethin', pal, I'm the king of late-night TV
My satire is sharp as a razor, it cuts deeper than a knife
While you're just a british bloke tryin' to be funny with your accent and your wit.
John Oliver:
Oh Stephen, don't be ridiculous, you may have the ratings but I got the real talk.
My show is the one that people actually watch and listen to, not just for the laughs but for the facts.
While you're busy talkin' trash, I'm out here bringing the truth to light.
Stephen Colbert:
Truth? Ha! You think your show is about truth? Please, it's all just a joke to you.
You're just a fancy-pants british guy tryin' to be funny with your news and your jokes.
While I'm the one who's really makin' a difference, with my sat
``````output
llama_print_timings:        load time =   358.60 ms
llama_print_timings:      sample time =   172.55 ms /   256 runs   (    0.67 ms per token,  1483.59 tokens per second)
llama_print_timings: prompt eval time =   613.36 ms /    16 tokens (   38.33 ms per token,    26.09 tokens per second)
llama_print_timings:        eval time = 10151.17 ms /   255 runs   (   39.81 ms per token,    25.12 tokens per second)
llama_print_timings:       total time = 11332.41 ms
```

```output
"\nStephen Colbert:\nYo, John, I heard you've been talkin' smack about me on your show.\nLet me tell you somethin', pal, I'm the king of late-night TV\nMy satire is sharp as a razor, it cuts deeper than a knife\nWhile you're just a british bloke tryin' to be funny with your accent and your wit.\nJohn Oliver:\nOh Stephen, don't be ridiculous, you may have the ratings but I got the real talk.\nMy show is the one that people actually watch and listen to, not just for the laughs but for the facts.\nWhile you're busy talkin' trash, I'm out here bringing the truth to light.\nStephen Colbert:\nTruth? Ha! You think your show is about truth? Please, it's all just a joke to you.\nYou're just a fancy-pants british guy tryin' to be funny with your news and your jokes.\nWhile I'm the one who's really makin' a difference, with my sat"
```

使用 LLaMA v1 模型的示例

```python
# 确保模型路径对您的系统是正确的！
llm = LlamaCpp(
    model_path="./ggml-model-q4_0.bin", callback_manager=callback_manager, verbose=True
)
```

```python
llm_chain = prompt | llm
```

```python
question = "贾斯汀·比伯出生年份的超级碗冠军是哪支 NFL 球队？"
llm_chain.invoke({"question": question})
```

```output
1. 首先，找出贾斯汀·比伯的出生年份。
2. 我们知道贾斯汀·比伯出生于1994年3月1日。
3. 接下来，我们需要查找当年超级碗是在哪天举行的。
4. 超级碗是在1995年1月28日举行的。
5. 最后，我们可以利用这些信息来回答问题。贾斯汀·比伯出生年份的超级碗冠军是旧金山 49ers。
``````output
llama_print_timings:        load time =   434.15 ms
llama_print_timings:      sample time =    41.81 ms /   121 runs   (    0.35 ms per token)
llama_print_timings: prompt eval time =  2523.78 ms /    48 tokens (   52.58 ms per token)
llama_print_timings:        eval time = 23971.57 ms /   121 runs   (  198.11 ms per token)
llama_print_timings:       total time = 28945.95 ms
```

```output
'\n\n1. 首先，找出贾斯汀·比伯的出生年份。\n2. 我们知道贾斯汀·比伯出生于1994年3月1日。\n3. 接下来，我们需要查找当年超级碗是在哪天举行的。\n4. 超级碗是在1995年1月28日举行的。\n5. 最后，我们可以利用这些信息来回答问题。贾斯汀·比伯出生年份的超级碗冠军是旧金山 49ers。'
```

### GPU

如果使用 BLAS 后端进行安装正确，则在模型属性中会看到 `BLAS = 1` 指示器。

在使用 GPU 时，两个最重要的参数是：

- `n_gpu_layers` - 确定模型中有多少层被转移到 GPU 上运行。

- `n_batch` - 指定并行处理的 token 数量。

正确设置这些参数将显著提高评估速度（详细信息请参见[包装代码](https://github.com/langchain-ai/langchain/blob/master/libs/community/langchain_community/llms/llamacpp.py)）。

```python
n_gpu_layers = -1  # 将要放在 GPU 上的层数。其余将在 CPU 上运行。如果不知道有多少层，可以使用 -1 将所有层都移到 GPU 上。
n_batch = 512  # 应该在 1 和 n_ctx 之间，考虑您的 GPU 的 VRAM 大小。
# 确保模型路径对您的系统是正确的！
llm = LlamaCpp(
    model_path="/Users/rlm/Desktop/Code/llama.cpp/models/openorca-platypus2-13b.gguf.q4_0.bin",
    n_gpu_layers=n_gpu_layers,
    n_batch=n_batch,
    callback_manager=callback_manager,
    verbose=True,  # 必须传递给回调管理器
)
```

```python
llm_chain = prompt | llm
question = "贾斯汀·比伯出生年份的超级碗冠军是哪支 NFL 球队？"
llm_chain.invoke({"question": question})
```

```output
1. 确定贾斯汀·比伯的出生日期：贾斯汀·比伯出生于1994年3月1日。
2. 找到那一年的超级碗冠军：1993年的 NFL 赛季，超级碗在1994年的1月举行。
3. 确定哪支球队赢得了比赛：达拉斯牛仔队在1993年1月31日的超级碗 XXVII 中对阵水牛城比尔斯队（由于错误标记年份，实际为1993年）。达拉斯牛仔队赢得了这场比赛。
因此，贾斯汀·比伯出生时，达拉斯牛仔队是 NFL 超级碗的卫冕冠军。
```

```output
llama_print_timings:        load time =   427.63 ms
llama_print_timings:      sample time =   115.85 ms /   164 runs   (    0.71 ms per token,  1415.67 tokens per second)
llama_print_timings: prompt eval time =   427.53 ms /    45 tokens (    9.50 ms per token,   105.26 tokens per second)
llama_print_timings:        eval time =  4526.53 ms /   163 runs   (   27.77 ms per token,    36.01 tokens per second)
llama_print_timings:       total time =  5293.77 ms
```

```output
"\n\n1. 确定贾斯汀·比伯的出生日期：贾斯汀·比伯出生于1994年3月1日。\n\n2. 找到那一年的超级碗冠军：1993年的 NFL 赛季，超级碗在1994年的1月举行。\n\n3. 确定哪支球队赢得了比赛：达拉斯牛仔队在1993年1月31日的超级碗 XXVII 中对阵水牛城比尔斯队（由于错误标记年份，实际为1993年）。达拉斯牛仔队赢得了这场比赛。\n\n因此，贾斯汀·比伯出生时，达拉斯牛仔队是 NFL 超级碗的卫冕冠军。"
```

### Metal

如果 Metal 安装正确，您将在模型属性中看到 `NEON = 1` 指示器。

两个最重要的 GPU 参数是：

- `n_gpu_layers` - 确定有多少层模型被转移到 Metal GPU 上运行。

- `n_batch` - 并行处理的 token 数量，默认为 8，可设置为更大的数字。

- `f16_kv` - 由于某些原因，Metal 仅支持 `True`，否则会出现错误，如 `Asserting on type 0 GGML_ASSERT: .../ggml-metal.m:706: false && "not implemented"`。

正确设置这些参数将显著提高评估速度（详细信息请参见[包装代码](https://github.com/langchain-ai/langchain/blob/master/libs/community/langchain_community/llms/llamacpp.py)）。

```python
n_gpu_layers = 1  # 将要放在 GPU 上的层数。其余将在 CPU 上运行。如果不知道有多少层，可以使用 -1 将所有层都移到 GPU 上。
n_batch = 512  # 应该在 1 和 n_ctx 之间，考虑您的 Apple Silicon 芯片的 RAM 大小。
# 确保模型路径对您的系统是正确的！
llm = LlamaCpp(
    model_path="/Users/rlm/Desktop/Code/llama.cpp/models/openorca-platypus2-13b.gguf.q4_0.bin",
    n_gpu_layers=n_gpu_layers,
    n_batch=n_batch,
    f16_kv=True,  # 必须设置为 True，否则在几次调用后会遇到问题
    callback_manager=callback_manager,
    verbose=True,  # 必须传递给回调管理器
)
```

控制台日志将显示以下日志以指示 Metal 已正确启用。

```
ggml_metal_init: allocating
ggml_metal_init: using MPS
...
```

您还可以通过观察进程的 GPU 使用情况来检查 `Activity Monitor`，在打开 `n_gpu_layers=1` 后，CPU 使用率将急剧下降。

对于第一次调用 LLM，由于在 Metal GPU 中进行模型编译，性能可能会较慢。

### 语法

我们可以使用[语法](https://github.com/ggerganov/llama.cpp/blob/master/grammars/README.md)来约束模型输出，并根据其中定义的规则对 token 进行采样。

为了演示这个概念，我们在下面的示例中包含了[示例语法文件](https://github.com/langchain-ai/langchain/tree/master/libs/langchain/langchain/llms/grammars)。

创建 gbnf 语法文件可能会耗费一些时间，但如果您有一个重要的输出模式的用例，有两个工具可以帮助：

- [在线语法生成器应用](https://grammar.intrinsiclabs.ai/)，可以将 TypeScript 接口定义转换为 gbnf 文件。

- [Python 脚本](https://github.com/ggerganov/llama.cpp/blob/master/examples/json-schema-to-grammar.py)，用于将 json 模式转换为 gbnf 文件。例如，您可以创建 `pydantic` 对象，使用 `.schema_json()` 方法生成其 JSON 模式，然后使用此脚本将其转换为 gbnf 文件。

在第一个示例中，提供指定的 `json.gbnf` 文件路径以生成 JSON：

```python
n_gpu_layers = 1  # 将要放在 GPU 上的层数。其余将放在 CPU 上。如果您不知道有多少层，可以使用 -1 将所有层移至 GPU。
n_batch = 512  # 应该在 1 和 n_ctx 之间，考虑您的 Apple Silicon 芯片的 RAM 量。
# 确保模型路径对于您的系统是正确的！
llm = LlamaCpp(
    model_path="/Users/rlm/Desktop/Code/llama.cpp/models/openorca-platypus2-13b.gguf.q4_0.bin",
    n_gpu_layers=n_gpu_layers,
    n_batch=n_batch,
    f16_kv=True,  # 必须设置为 True，否则在几次调用后会遇到问题
    callback_manager=callback_manager,
    verbose=True,  # 必须传递给回调管理器
    grammar_path="/Users/rlm/Desktop/Code/langchain-main/langchain/libs/langchain/langchain/llms/grammars/json.gbnf",
)
```

```python
%%capture captured --no-stdout
result = llm.invoke("以 JSON 格式描述一个人：")
```

```output
{
  "name": "John Doe",
  "age": 34,
  "": {
    "title": "软件开发人员",
    "company": "Google"
  },
  "interests": [
    "体育",
    "音乐",
    "烹饪"
  ],
  "address": {
    "street_number": 123,
    "street_name": "橡树街",
    "city": "Mountain View",
    "state": "加利福尼亚",
    "postal_code": 94040
  }}
``````output
llama_print_timings:        加载时间 =   357.51 毫秒
llama_print_timings:      示例时间 =  1213.30 毫秒 /   144 次运行   (    8.43 毫秒每个标记,   118.68 每秒标记)
llama_print_timings: 提示评估时间 =   356.78 毫秒 /     9 个标记 (   39.64 毫秒每个标记,    25.23 每秒标记)
llama_print_timings:        评估时间 =  3947.16 毫秒 /   143 次运行   (   27.60 毫秒每个标记,    36.23 每秒标记)
llama_print_timings:       总时间 =  5846.21 毫秒
```

我们也可以提供 `list.gbnf` 来返回一个列表：

```python
n_gpu_layers = 1
n_batch = 512
llm = LlamaCpp(
    model_path="/Users/rlm/Desktop/Code/llama.cpp/models/openorca-platypus2-13b.gguf.q4_0.bin",
    n_gpu_layers=n_gpu_layers,
    n_batch=n_batch,
    f16_kv=True,  # 必须设置为 True，否则在几次调用后会遇到问题
    callback_manager=callback_manager,
    verbose=True,
    grammar_path="/Users/rlm/Desktop/Code/langchain-main/langchain/libs/langchain/langchain/llms/grammars/list.gbnf",
)
```

```python
%%capture captured --no-stdout
result = llm.invoke("我最喜欢的前三本书的列表：")
```

```output
["麦田里的守望者", "呼啸山庄", "安娜·卡列尼娜"]
``````output
llama_print_timings:        加载时间 =   322.34 毫秒
llama_print_timings:      示例时间 =   232.60 毫秒 /    26 次运行   (    8.95 毫秒每个标记,   111.78 每秒标记)
llama_print_timings: 提示评估时间 =   321.90 毫秒 /    11 个标记 (   29.26 毫秒每个标记,    34.17 每秒标记)
llama_print_timings:        评估时间 =   680.82 毫秒 /    25 次运行   (   27.23 毫秒每个标记,    36.72 每秒标记)
llama_print_timings:       总时间 =  1295.27 毫秒
```