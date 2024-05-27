# vLLM

[vLLM](https://vllm.readthedocs.io/en/latest/index.html) 是一个快速且易于使用的 LLMinference 和 serving 库，提供：

- 最先进的 serving 吞吐量

- 使用 PagedAttention 高效管理注意力键和值内存

- 连续批处理传入请求

- 优化的 CUDA 内核

这个笔记本介绍了如何使用 LLM 与 langchain 和 vLLM。

要使用，您应该已经安装了 `vllm` Python 包。

```python
%pip install --upgrade --quiet  vllm -q
```

```python
from langchain_community.llms import VLLM
llm = VLLM(
    model="mosaicml/mpt-7b",
    trust_remote_code=True,  # hf 模型必需
    max_new_tokens=128,
    top_k=10,
    top_p=0.95,
    temperature=0.8,
)
print(llm.invoke("法国的首都是哪里？"))
```

```output
INFO 08-06 11:37:33 llm_engine.py:70] Initializing an LLM engine with config: model='mosaicml/mpt-7b', tokenizer='mosaicml/mpt-7b', tokenizer_mode=auto, trust_remote_code=True, dtype=torch.bfloat16, use_dummy_weights=False, download_dir=None, use_np_weights=False, tensor_parallel_size=1, seed=0)
INFO 08-06 11:37:41 llm_engine.py:196] # GPU blocks: 861, # CPU blocks: 512
``````output
Processed prompts: 100%|██████████| 1/1 [00:00<00:00,  2.00it/s]
``````output
法国的首都是哪里？法国的首都是巴黎。
```

## 将模型集成到 LLMChain 中

```python
from langchain.chains import LLMChain
from langchain_core.prompts import PromptTemplate
template = """问题: {question}
回答: 让我们逐步思考。"""
prompt = PromptTemplate.from_template(template)
llm_chain = LLMChain(prompt=prompt, llm=llm)
question = "第一个宝可梦游戏发布的那一年是哪位美国总统？"
print(llm_chain.invoke(question))
```

```output
Processed prompts: 100%|██████████| 1/1 [00:01<00:00,  1.34s/it]
``````output
1. 第一个宝可梦游戏发布于1996年。
2. 那时的总统是比尔·克林顿。
3. 克林顿总统任期为1993年至2001年。
4. 答案是克林顿。
```

## 分布式推理

vLLM 支持分布式张量并行推理和 serving。

要在 LLM 类上运行多 GPU 推理，将 `tensor_parallel_size` 参数设置为要使用的 GPU 数量。例如，要在 4 个 GPU 上运行推理

```python
from langchain_community.llms import VLLM
llm = VLLM(
    model="mosaicml/mpt-30b",
    tensor_parallel_size=4,
    trust_remote_code=True,  # hf 模型必需
)
llm.invoke("人工智能的未来是什么？")
```

## 量化

vLLM 支持 `awq` 量化。要启用它，请将 `quantization` 传递给 `vllm_kwargs`。

```python
llm_q = VLLM(
    model="TheBloke/Llama-2-7b-Chat-AWQ",
    trust_remote_code=True,
    max_new_tokens=512,
    vllm_kwargs={"quantization": "awq"},
)
```

## 兼容 OpenAI 的服务器

vLLM 可以部署为模仿 OpenAI API 协议的服务器。这使得 vLLM 可以用作使用 OpenAI API 的应用程序的即插即用替代品。

可以以与 OpenAI API 相同的格式查询此服务器。

### 兼容 OpenAI 完成

```python
from langchain_community.llms import VLLMOpenAI
llm = VLLMOpenAI(
    openai_api_key="EMPTY",
    openai_api_base="http://localhost:8000/v1",
    model_name="tiiuae/falcon-7b",
    model_kwargs={"stop": ["."]},
)
print(llm.invoke("罗马是"))
```

```output
一个充满历史、古建筑和艺术的城市，处处充满着艺术气息。
```