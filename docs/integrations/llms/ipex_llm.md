# IPEX-LLM

[IPEX-LLM](https://github.com/intel-analytics/ipex-llm/) 是一个用于在英特尔 CPU 和 GPU 上运行 LLM 的 PyTorch 库（例如，配备 iGPU 的本地 PC，以及 Arc、Flex 和 Max 等独立 GPU），具有非常低的延迟。

这个示例演示了如何使用 LangChain 与 `ipex-llm` 进行文本生成交互。

## 设置

```python
# 更新 Langchain
%pip install -qU langchain langchain-community
```

安装 IEPX-LLM 以在英特尔 CPU 上本地运行 LLM。

```python
%pip install --pre --upgrade ipex-llm[all]
```

## 基本用法

```python
import warnings
from langchain.chains import LLMChain
from langchain_community.llms import IpexLLM
from langchain_core.prompts import PromptTemplate
warnings.filterwarnings("ignore", category=UserWarning, message=".*padding_mask.*")
```

为您的模型指定提示模板。在本示例中，我们使用 [vicuna-1.5](https://huggingface.co/lmsys/vicuna-7b-v1.5) 模型。如果您使用不同的模型，请相应选择适当的模板。

```python
template = "USER: {question}\nASSISTANT:"
prompt = PromptTemplate(template=template, input_variables=["question"])
```

使用 `IpexLLM.from_model_id` 在本地加载模型。它将直接以 Huggingface 格式加载模型，并自动转换为低比特格式以进行推断。

```python
llm = IpexLLM.from_model_id(
    model_id="lmsys/vicuna-7b-v1.5",
    model_kwargs={"temperature": 0, "max_length": 64, "trust_remote_code": True},
)
```

在 Chains 中使用：

```python
llm_chain = prompt | llm
question = "What is AI?"
output = llm_chain.invoke(question)
```

## 保存/加载低比特模型

或者，您可以将低比特模型保存到磁盘一次，然后在以后使用 `from_model_id_low_bit` 而不是 `from_model_id` 进行重新加载 - 即使跨不同机器使用也可以。低比特模型占用的磁盘空间明显少于原始模型，因此更加高效。而且 `from_model_id_low_bit` 在速度和内存使用方面也比 `from_model_id` 更高效，因为它跳过了模型转换步骤。

要保存低比特模型，请使用 `save_low_bit` 如下所示。

```python
saved_lowbit_model_path = "./vicuna-7b-1.5-low-bit"  # 保存低比特模型的路径
llm.model.save_low_bit(saved_lowbit_model_path)
del llm
```

从保存的低比特模型路径加载模型如下。

> 请注意，低比特模型的保存路径仅包括模型本身，而不包括分词器。如果您希望将所有内容放在一个地方，您需要手动从原始模型目录下载或复制分词器文件到保存低比特模型的位置。

```python
llm_lowbit = IpexLLM.from_model_id_low_bit(
    model_id=saved_lowbit_model_path,
    tokenizer_id="lmsys/vicuna-7b-v1.5",
    # tokenizer_name=saved_lowbit_model_path,  # 如果您想以这种方式使用，请将分词器复制到保存路径
    model_kwargs={"temperature": 0, "max_length": 64, "trust_remote_code": True},
)
```

在 Chains 中使用加载的模型：

```python
llm_chain = prompt | llm_lowbit
question = "What is AI?"
output = llm_chain.invoke(question)
```