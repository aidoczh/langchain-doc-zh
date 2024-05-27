

# OpenVINO

[OpenVINO™](https://github.com/openvinotoolkit/openvino) 是一个用于优化和部署 AI 推断的开源工具包。OpenVINO™ Runtime 可以在各种硬件[设备](https://github.com/openvinotoolkit/openvino?tab=readme-ov-file#supported-hardware-matrix)上运行经过优化的相同模型。加速您的深度学习性能，涵盖语言模型、计算机视觉、自动语音识别等用例。

OpenVINO 模型可以通过 `HuggingFacePipeline` [类](https://python.langchain.com/docs/integrations/llms/huggingface_pipeline) 在本地运行。要使用 OpenVINO 部署模型，可以指定 `backend="openvino"` 参数以触发 OpenVINO 作为后端推断框架。

要使用，您应该安装带有 OpenVINO 加速器 Python [包的``optimum-intel``](https://github.com/huggingface/optimum-intel?tab=readme-ov-file#installation)。

```python
%pip install --upgrade-strategy eager "optimum[openvino,nncf]" --quiet
```

### 模型加载

可以通过使用 `from_model_id` 方法指定模型参数来加载模型。

如果您有英特尔 GPU，可以指定 `model_kwargs={"device": "GPU"}` 在其上运行推断。

```python
from langchain_huggingface import HuggingFacePipeline
ov_config = {"PERFORMANCE_HINT": "LATENCY", "NUM_STREAMS": "1", "CACHE_DIR": ""}
ov_llm = HuggingFacePipeline.from_model_id(
    model_id="gpt2",
    task="text-generation",
    backend="openvino",
    model_kwargs={"device": "CPU", "ov_config": ov_config},
    pipeline_kwargs={"max_new_tokens": 10},
)
```

也可以通过直接传递现有的 [`optimum-intel`](https://huggingface.co/docs/optimum/main/en/intel/inference) pipeline 来加载模型。

```python
from optimum.intel.openvino import OVModelForCausalLM
from transformers import AutoTokenizer, pipeline
model_id = "gpt2"
device = "CPU"
tokenizer = AutoTokenizer.from_pretrained(model_id)
ov_model = OVModelForCausalLM.from_pretrained(
    model_id, export=True, device=device, ov_config=ov_config
)
ov_pipe = pipeline(
    "text-generation", model=ov_model, tokenizer=tokenizer, max_new_tokens=10
)
ov_llm = HuggingFacePipeline(pipeline=ov_pipe)
```

### 创建链

将加载到内存中的模型与提示组合起来形成一个链。

```python
from langchain_core.prompts import PromptTemplate
template = """Question: {question}
Answer: Let's think step by step."""
prompt = PromptTemplate.from_template(template)
chain = prompt | ov_llm
question = "What is electroencephalography?"
print(chain.invoke({"question": question}))
```

### 使用本地 OpenVINO 模型进行推断

可以使用 CLI [导出您的模型](https://github.com/huggingface/optimum-intel?tab=readme-ov-file#export) 到 OpenVINO IR 格式，并从本地文件夹加载模型。

```python
!optimum-cli export openvino --model gpt2 ov_model_dir
```

建议使用 `--weight-format` 应用 8 或 4 位权重量化以减少推断延迟和模型占用空间。

```python
!optimum-cli export openvino --model gpt2  --weight-format int8 ov_model_dir # 8 位量化
!optimum-cli export openvino --model gpt2  --weight-format int4 ov_model_dir # 4 位量化
```

```python
ov_llm = HuggingFacePipeline.from_model_id(
    model_id="ov_model_dir",
    task="text-generation",
    backend="openvino",
    model_kwargs={"device": "CPU", "ov_config": ov_config},
    pipeline_kwargs={"max_new_tokens": 10},
)
chain = prompt | ov_llm
question = "What is electroencephalography?"
print(chain.invoke({"question": question}))
```

您可以通过动态量化激活和 KV-cache 量化获得额外的推断速度提升。这些选项可以通过 `ov_config` 启用如下：

```python
ov_config = {
    "KV_CACHE_PRECISION": "u8",
    "DYNAMIC_QUANTIZATION_GROUP_SIZE": "32",
    "PERFORMANCE_HINT": "LATENCY",
    "NUM_STREAMS": "1",
    "CACHE_DIR": "",
}
```

### 流式处理

要获得 LLM 输出的流式处理，您可以为 `_forward_params` 创建一个 Huggingface `TextIteratorStreamer`。

```python
from threading import Thread
from transformers import TextIteratorStreamer
streamer = TextIteratorStreamer(
    ov_llm.pipeline.tokenizer,
    timeout=30.0,
    skip_prompt=True,
    skip_special_tokens=True,
)
ov_llm.pipeline._forward_params = {"streamer": streamer, "max_new_tokens": 100}
t1 = Thread(target=chain.invoke, args=({"question": question},))
t1.start()
for new_text in streamer:
    print(new_text, end="", flush=True)
```

更多信息请参考：

- [OpenVINO LLM 指南](https://docs.openvino.ai/2024/learn-openvino/llm_inference_guide.html)

- [OpenVINO 文档](https://docs.openvino.ai/2024/home.html)

* [OpenVINO 入门指南](https://www.intel.com/content/www/us/en/content-details/819067/openvino-get-started-guide.html)。

* [LangChain 上的 RAG 笔记本](https://github.com/openvinotoolkit/openvino_notebooks/tree/latest/notebooks/llm-rag-langchain)。