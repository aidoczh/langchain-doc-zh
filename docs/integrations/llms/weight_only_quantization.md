# Intel 仅权重量化
## 使用 Intel Transformers 扩展为 Huggingface 模型进行仅权重量化
Hugging Face 模型可以通过 `WeightOnlyQuantPipeline` 类在本地运行仅权重量化。
[Hugging Face 模型中心](https://huggingface.co/models)托管了超过 120,000 个模型、20,000 个数据集和 50,000 个演示应用程序（Spaces），所有这些都是开源的，公开可用，人们可以在这个在线平台上轻松合作并共同构建机器学习。
这些可以通过 LangChain 通过本地管道包装类调用。
要使用，您应该安装 ``transformers`` python [包](https://pypi.org/project/transformers/)，以及 [pytorch](https://pytorch.org/get-started/locally/)，[intel-extension-for-transformers](https://github.com/intel/intel-extension-for-transformers)。
```python
%pip install transformers --quiet
%pip install intel-extension-for-transformers
```
### 模型加载
可以通过使用 `from_model_id` 方法指定模型参数来加载模型。模型参数包括 `intel_extension_for_transformers` 中的 `WeightOnlyQuantConfig` 类。
```python
from intel_extension_for_transformers.transformers import WeightOnlyQuantConfig
from langchain_community.llms.weight_only_quantization import WeightOnlyQuantPipeline
conf = WeightOnlyQuantConfig(weight_dtype="nf4")
hf = WeightOnlyQuantPipeline.from_model_id(
    model_id="google/flan-t5-large",
    task="text2text-generation",
    quantization_config=conf,
    pipeline_kwargs={"max_new_tokens": 10},
)
```
也可以通过直接传递现有的 `transformers` 管道来加载模型
```python
from intel_extension_for_transformers.transformers import AutoModelForSeq2SeqLM
from transformers import AutoTokenizer, pipeline
model_id = "google/flan-t5-large"
tokenizer = AutoTokenizer.from_pretrained(model_id)
model = AutoModelForSeq2SeqLM.from_pretrained(model_id)
pipe = pipeline(
    "text2text-generation", model=model, tokenizer=tokenizer, max_new_tokens=10
)
hf = WeightOnlyQuantPipeline(pipeline=pipe)
```
### 创建链
将模型加载到内存中后，您可以将其与提示组合起来形成一个链。
```python
from langchain_core.prompts import PromptTemplate
template = """Question: {question}
Answer: Let's think step by step."""
prompt = PromptTemplate.from_template(template)
chain = prompt | hf
question = "What is electroencephalography?"
print(chain.invoke({"question": question}))
```
### CPU 推断
现在 intel-extension-for-transformers 仅支持 CPU 设备推断。将很快支持 Intel GPU。在使用 CPU 的机器上运行时，您可以指定 `device="cpu"` 或 `device=-1` 参数将模型放在 CPU 设备上。
默认为 `-1` 用于 CPU 推断。
```python
conf = WeightOnlyQuantConfig(weight_dtype="nf4")
llm = WeightOnlyQuantPipeline.from_model_id(
    model_id="google/flan-t5-large",
    task="text2text-generation",
    quantization_config=conf,
    pipeline_kwargs={"max_new_tokens": 10},
)
template = """Question: {question}
Answer: Let's think step by step."""
prompt = PromptTemplate.from_template(template)
chain = prompt | llm
question = "What is electroencephalography?"
print(chain.invoke({"question": question}))
```
### 批量 CPU 推断
您还可以以批处理模式在 CPU 上运行推断。
```python
conf = WeightOnlyQuantConfig(weight_dtype="nf4")
llm = WeightOnlyQuantPipeline.from_model_id(
    model_id="google/flan-t5-large",
    task="text2text-generation",
    quantization_config=conf,
    pipeline_kwargs={"max_new_tokens": 10},
)
chain = prompt | llm.bind(stop=["\n\n"])
questions = []
for i in range(4):
    questions.append({"question": f"What is the number {i} in french?"})
answers = chain.batch(questions)
for answer in answers:
    print(answer)
```
### Intel-extension-for-transformers 支持的数据类型
我们支持将权重量化为以下数据类型进行存储（WeightOnlyQuantConfig 中的 weight_dtype）：
- **int8**：使用 8 位数据类型。
- **int4_fullrange**：与正常的 int4 范围 [-7,7] 相比，使用 int4 范围的 -8 值。
- **int4_clip**：剪切并保留在 int4 范围内的值，将其他值设为零。
- **nf4**：使用归一化的 4 位浮点数数据类型。
- **fp4_e2m1**：使用常规的 4 位浮点数数据类型。"e2" 表示使用 2 位作为指数，"m1" 表示使用 1 位作为尾数。
虽然这些技术将权重存储为 4 或 8 位，但计算仍然以 float32、bfloat16 或 int8（WeightOnlyQuantConfig 中的 compute_dtype）进行。
- **fp32**：使用 float32 数据类型进行计算。
- **bf16**：使用 bfloat16 数据类型进行计算。
- **int8**：使用 8 位数据类型进行计算。
### 支持的算法矩阵
在 intel-extension-for-transformers（WeightOnlyQuantConfig 中的算法）中支持的量化算法有：
| 算法 |   PyTorch  |    LLM Runtime    |
|:--------------:|:----------:|:----------:|
|       RTN      |  &#10004;  |  &#10004;  |
|       AWQ      |  &#10004;  | 敬请期待 |
|      TEQ      | &#10004; | 敬请期待 |
**RTN:** 一种直观易懂的量化方法。它不需要额外的数据集，是一种非常快速的量化方法。一般来说，RTN会将权重转换为均匀分布的整数数据类型，但一些算法，如 Qlora，提出了一种非均匀的 NF4 数据类型，并证明了其在理论上的最优性。[20]
**AWQ:** 证明了仅保护 1% 的显著权重可以大大减少量化误差。通过观察每个通道的激活和权重分布来选择显著权重通道。在量化之前，显著权重还会乘以一个大的比例因子以进行保留。[21]
**TEQ:** 一种在仅对权重进行量化时保持 FP32 精度的可训练等效变换。受 AWQ 的启发，同时提供了一种新的解决方案，用于搜索激活和权重之间的每个通道缩放因子的最佳值。[22]