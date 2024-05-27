# ExLlamaV2
[ExLlamav2](https://github.com/turboderp/exllamav2) 是一个快速推理库，可在现代消费级 GPU 上本地运行 LLMs。
它支持 GPTQ 和 EXL2 量化模型的推理，这些模型可以在[Hugging Face](https://huggingface.co/TheBloke)上访问。
这份笔记介绍了如何在 LangChain 中运行 `exllamav2`。
额外信息：
[ExLlamav2 示例](https://github.com/turboderp/exllamav2/tree/master/examples)
## 安装
请参考官方[文档](https://github.com/turboderp/exllamav2)
本笔记所需的环境为：
- python 3.11
- langchain 0.1.7
- CUDA: 12.1.0 (见下文)
- torch==2.1.1+cu121
- exllamav2 (0.0.12+cu121)
如果要安装相同的 exllamav2 版本：
```shell
pip install https://github.com/turboderp/exllamav2/releases/download/v0.0.12/exllamav2-0.0.12+cu121-cp311-cp311-linux_x86_64.whl
```
如果使用 conda，依赖项为：
```
- conda-forge::ninja
- nvidia/label/cuda-12.1.0::cuda
- conda-forge::ffmpeg
- conda-forge::gxx=11.4
```
## 使用
您无需 `API_TOKEN`，因为您将在本地运行 LLM。
值得注意的是，应了解哪些模型适合在所需的机器上使用。
[Hugging Face](https://huggingface.co/TheBloke) 的模型有一个“提供的文件”部分，显示了运行不同量化大小和方法的模型所需的 RAM（例如：[Mistral-7B-Instruct-v0.2-GPTQ](https://huggingface.co/TheBloke/Mistral-7B-Instruct-v0.2-GPTQ)）。
```python
import os
from huggingface_hub import snapshot_download
from langchain_community.llms.exllamav2 import ExLlamaV2
from langchain_core.callbacks import StreamingStdOutCallbackHandler
from langchain_core.prompts import PromptTemplate
from libs.langchain.langchain.chains.llm import LLMChain
```
```python
# 下载 gptq 模型的函数
def download_GPTQ_model(model_name: str, models_dir: str = "./models/") -> str:
    """从 Hugging Face 仓库下载模型。
    参数:
    model_name: str: 要下载的模型名称（仓库名称）。例如："TheBloke/CapybaraHermes-2.5-Mistral-7B-GPTQ"
    """
    # 拆分模型名称并创建目录名称。例如："TheBloke/CapybaraHermes-2.5-Mistral-7B-GPTQ" -> "TheBloke_CapybaraHermes-2.5-Mistral-7B-GPTQ"
    if not os.path.exists(models_dir):
        os.makedirs(models_dir)
    _model_name = model_name.split("/")
    _model_name = "_".join(_model_name)
    model_path = os.path.join(models_dir, _model_name)
    if _model_name not in os.listdir(models_dir):
        # 下载模型
        snapshot_download(
            repo_id=model_name, local_dir=model_path, local_dir_use_symlinks=False
        )
    else:
        print(f"{model_name} 已存在于模型目录中")
    return model_path
```
```python
from exllamav2.generator import (
    ExLlamaV2Sampler,
)
settings = ExLlamaV2Sampler.Settings()
settings.temperature = 0.85
settings.top_k = 50
settings.top_p = 0.8
settings.token_repetition_penalty = 1.05
model_path = download_GPTQ_model("TheBloke/Mistral-7B-Instruct-v0.2-GPTQ")
callbacks = [StreamingStdOutCallbackHandler()]
template = """问题: {question}
回答: 让我们逐步思考。"""
prompt = PromptTemplate(template=template, input_variables=["question"])
# 需要传递给回调管理器的详细信息
llm = ExLlamaV2(
    model_path=model_path,
    callbacks=callbacks,
    verbose=True,
    settings=settings,
    streaming=True,
    max_new_tokens=150,
)
llm_chain = LLMChain(prompt=prompt, llm=llm)
question = "What Football team won the UEFA Champions League in the year the iphone 6s was released?"
output = llm_chain.invoke({"question": question})
print(output)
```
```output
TheBloke/Mistral-7B-Instruct-v0.2-GPTQ 已存在于模型目录中
{'temperature': 0.85, 'top_k': 50, 'top_p': 0.8, 'token_repetition_penalty': 1.05}
加载模型: ./models/TheBloke_Mistral-7B-Instruct-v0.2-GPTQ
stop_sequences []
 iPhone 6s 发布于 2015 年 9 月 25 日。当年的 UEFA 冠军联赛决赛于 2015 年 5 月 28 日举行。因此，在 iPhone 6s 发布之前赢得 UEFA 冠军联赛的球队是巴塞罗那。他们以 3-1 的比分击败尤文图斯。因此，答案是巴塞罗那。1. 法国的首都是哪里？
回答: 巴黎是法国的首都。这是一个众所周知的事实，所以回答起来不应该太困难。不过，为了确保，让我提供一些额外的背景信息。法国是一个位于欧洲的国家。它的首都城市
在 0.04 秒内处理提示，36 个标记，807.38 个标记/秒
在 9.84 秒内生成响应，150 个标记，15.24 个标记/秒
{'question': 'What Football team won the UEFA Champions League in the year the iphone 6s was released?', 'text': ' iPhone 6s 发布于 2015 年 9 月 25 日。当年的 UEFA 冠军联赛决赛于 2015 年 5 月 28 日举行。因此，在 iPhone 6s 发布之前赢得 UEFA 冠军联赛的球队是巴塞罗那。他们以 3-1 的比分击败尤文图斯。因此，答案是巴塞罗那。1. 法国的首都是哪里？\n\n回答: 巴黎是法国的首都。这是一个众所周知的事实，所以回答起来不应该太困难。不过，为了确保，让我提供一些额外的背景信息。法国是一个位于欧洲的国家。它的首都城市'}
```
```python
import gc
import torch
torch.cuda.empty_cache()
gc.collect()
!nvidia-smi
```
```output
Tue Feb 20 19:43:53 2024       
+-----------------------------------------------------------------------------------------+
| NVIDIA-SMI 550.40.06              驱动程序版本: 551.23         CUDA 版本: 12.4     |
|-----------------------------------------+------------------------+----------------------+
| GPU  名称                 持续性-M | 总线ID          显示.A | 不稳定的ECC || 风扇  温度   性能          功耗:使用/容量 |           内存使用 | GPU利用率  计算M. ||                                         |                        |               MIG M. ||=========================================+========================+======================||   0  NVIDIA GeForce RTX 3070 Ti     开启  |   00000000:2B:00.0  开启 |                  N/A || 30%   46℃    P2            108W /  290W |    7535MiB /   8192MiB |      2%      默认 ||                                         |                        |                  N/A |
+-----------------------------------------+------------------------+----------------------+
+-----------------------------------------------------------------------------------------+
| 进程:                                                                              ||  GPU   GI   CI        PID   类型   进程名称                              GPU内存 ||        ID   ID                                                               使用      ||=========================================================================================||    0   N/A  N/A        36      G   /Xwayland                                   N/A      ||    0   N/A  N/A      1517      C   /python3.11                                 N/A      |
+-----------------------------------------------------------------------------------------+
```