# MLX 本地管道

MLX 模型可以通过 `MLXPipeline` 类在本地运行。

[MLX 社区](https://huggingface.co/mlx-community) 托管了超过 150 个模型，所有这些模型都是开源的，可以在 Hugging Face Model Hub 上公开获取，这是一个在线平台，人们可以轻松地进行协作并共同构建机器学习模型。

这些模型可以通过 LangChain 中的本地管道包装器调用，也可以通过调用它们托管的推理端点来使用 `MlXPipeline` 类。有关 mlx 的更多信息，请参阅 [示例存储库](https://github.com/ml-explore/mlx-examples/tree/main/llms) 笔记本。

要使用这些功能，您应该安装 ``mlx-lm`` python [包](https://pypi.org/project/mlx-lm/)，以及 [transformers](https://pypi.org/project/transformers/)。您也可以安装 `huggingface_hub`。

```python
%pip install --upgrade --quiet  mlx-lm transformers huggingface_hub
```

### 模型加载

可以通过使用 `from_model_id` 方法指定模型参数来加载模型。

```python
from langchain_community.llms.mlx_pipeline import MLXPipeline
pipe = MLXPipeline.from_model_id(
    "mlx-community/quantized-gemma-2b-it",
    pipeline_kwargs={"max_tokens": 10, "temp": 0.1},
)
```

也可以通过直接传入现有的 `transformers` 管道来加载模型。

```python
from mlx_lm import load
model, tokenizer = load("mlx-community/quantized-gemma-2b-it")
pipe = MLXPipeline(model=model, tokenizer=tokenizer)
```

### 创建链

将模型加载到内存后，您可以将其与提示组合起来形成一个链条。

```python
from langchain_core.prompts import PromptTemplate
template = """Question: {question}
Answer: Let's think step by step."""
prompt = PromptTemplate.from_template(template)
chain = prompt | pipe
question = "What is electroencephalography?"
print(chain.invoke({"question": question}))
```