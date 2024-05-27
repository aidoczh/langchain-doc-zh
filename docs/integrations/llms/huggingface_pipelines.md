

# Hugging Face 本地管道

Hugging Face 模型可以通过 `HuggingFacePipeline` 类在本地运行。

[Hugging Face 模型中心](https://huggingface.co/models) 托管了超过 120,000 个模型、20,000 个数据集和 50,000 个演示应用程序（Spaces），所有这些都是开源且公开可用的，位于一个在线平台上，人们可以轻松合作并共同构建机器学习模型。

这些模型可以通过 LangChain 中的本地管道包装器调用，也可以通过调用 HuggingFaceHub 类的托管推理端点来调用。

要使用，您应该安装 ``transformers`` python [包](https://pypi.org/project/transformers/)，以及 [pytorch](https://pytorch.org/get-started/locally/)。您还可以安装 `xformer` 以获得更节省内存的注意力实现。

```python
%pip install --upgrade --quiet  transformers --quiet
```

### 模型加载

可以通过使用 `from_model_id` 方法指定模型参数来加载模型。

```python
from langchain_huggingface.llms import HuggingFacePipeline
hf = HuggingFacePipeline.from_model_id(
    model_id="gpt2",
    task="text-generation",
    pipeline_kwargs={"max_new_tokens": 10},
)
```

也可以通过直接传递现有的 `transformers` 管道来加载模型。

```python
from langchain_huggingface.llms import HuggingFacePipeline
from transformers import AutoModelForCausalLM, AutoTokenizer, pipeline
model_id = "gpt2"
tokenizer = AutoTokenizer.from_pretrained(model_id)
model = AutoModelForCausalLM.from_pretrained(model_id)
pipe = pipeline("text-generation", model=model, tokenizer=tokenizer, max_new_tokens=10)
hf = HuggingFacePipeline(pipeline=pipe)
```

### 创建链

将加载到内存中的模型与提示组合起来形成一个链。

```python
from langchain_core.prompts import PromptTemplate
template = """Question: {question}
Answer: Let's think step by step."""
prompt = PromptTemplate.from_template(template)
chain = prompt | hf
question = "What is electroencephalography?"
print(chain.invoke({"question": question}))
```

### GPU 推理

在具有 GPU 的计算机上运行时，您可以指定 `device=n` 参数将模型放在指定的设备上。默认为 `-1` 用于 CPU 推理。

如果您有多个 GPU 和/或模型对于单个 GPU 太大，可以指定 `device_map="auto"`，这需要并使用 [Accelerate](https://huggingface.co/docs/accelerate/index) 库自动确定如何加载模型权重。

*注意*：`device` 和 `device_map` 不应该同时指定，可能导致意外行为。

```python
gpu_llm = HuggingFacePipeline.from_model_id(
    model_id="gpt2",
    task="text-generation",
    device=0,  # 替换为 device_map="auto" 以使用 accelerate 库。
    pipeline_kwargs={"max_new_tokens": 10},
)
gpu_chain = prompt | gpu_llm
question = "What is electroencephalography?"
print(gpu_chain.invoke({"question": question}))
```

### 批量 GPU 推理

如果在具有 GPU 的设备上运行，还可以以批处理模式在 GPU 上运行推理。

```python
gpu_llm = HuggingFacePipeline.from_model_id(
    model_id="bigscience/bloom-1b7",
    task="text-generation",
    device=0,  # -1 为 CPU
    batch_size=2,  # 根据 GPU 映射和模型大小进行调整。
    model_kwargs={"temperature": 0, "max_length": 64},
)
gpu_chain = prompt | gpu_llm.bind(stop=["\n\n"])
questions = []
for i in range(4):
    questions.append({"question": f"What is the number {i} in french?"})
answers = gpu_chain.batch(questions)
for answer in answers:
    print(answer)
```

### 使用 OpenVINO 后端进行推理

要使用 OpenVINO 部署模型，可以指定 `backend="openvino"` 参数以触发 OpenVINO 作为后端推理框架。

如果您有英特尔 GPU，可以指定 `model_kwargs={"device": "GPU"}` 在其上运行推理。

```python
%pip install --upgrade-strategy eager "optimum[openvino,nncf]" --quiet
```

```python
ov_config = {"PERFORMANCE_HINT": "LATENCY", "NUM_STREAMS": "1", "CACHE_DIR": ""}
ov_llm = HuggingFacePipeline.from_model_id(
    model_id="gpt2",
    task="text-generation",
    backend="openvino",
    model_kwargs={"device": "CPU", "ov_config": ov_config},
    pipeline_kwargs={"max_new_tokens": 10},
)
ov_chain = prompt | ov_llm
question = "What is electroencephalography?"
print(ov_chain.invoke({"question": question}))
```

### 使用本地 OpenVINO 模型进行推理

可以使用 CLI 将模型[导出](https://github.com/huggingface/optimum-intel?tab=readme-ov-file#export)到 OpenVINO IR 格式，并从本地文件夹加载模型。

```python
!optimum-cli export openvino --model gpt2 ov_model_dir
```

建议应用 8 位或 4 位权重量化以减少推理延迟和模型占用空间，使用 `--weight-format`：

```python
!optimum-cli export openvino --model gpt2  --weight-format int8 ov_model_dir # 用于 8 位量化
!optimum-cli export openvino --model gpt2  --weight-format int4 ov_model_dir # 用于 4 位量化
```

```python
ov_llm = HuggingFacePipeline.from_model_id(
    model_id="ov_model_dir",
    task="text-generation",
    backend="openvino",
    model_kwargs={"device": "CPU", "ov_config": ov_config},
    pipeline_kwargs={"max_new_tokens": 10},
)
ov_chain = prompt | ov_llm
question = "什么是脑电图？"
print(ov_chain.invoke({"question": question}))
```

通过动态量化激活和 KV-cache 量化，您可以获得额外的推理速度提升。可以通过以下方式使用 `ov_config` 启用这些选项：

```python
ov_config = {
    "KV_CACHE_PRECISION": "u8",
    "DYNAMIC_QUANTIZATION_GROUP_SIZE": "32",
    "PERFORMANCE_HINT": "LATENCY",
    "NUM_STREAMS": "1",
    "CACHE_DIR": "",
}
```

更多信息请参考 [OpenVINO LLM 指南](https://docs.openvino.ai/2024/learn-openvino/llm_inference_guide.html) 和 [OpenVINO 本地管道笔记本](/docs/integrations/llms/openvino/)。