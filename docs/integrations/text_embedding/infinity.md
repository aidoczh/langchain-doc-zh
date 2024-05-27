# 无穷大

`Infinity` 允许使用 MIT 许可的嵌入服务器创建 `Embeddings`。

本笔记将介绍如何使用 [Infinity Github 项目](https://github.com/michaelfeil/infinity) 中的 Embeddings 与 Langchain。

## 导入

```python
from langchain_community.embeddings import InfinityEmbeddings, InfinityEmbeddingsLocal
```

# 选项 1：从 Python 使用 infinity

#### 可选：安装 infinity

要安装 infinity，请使用以下命令。有关更多详细信息，请查看 [Github 上的文档](https://github.com/michaelfeil/infinity)。

安装 torch 和 onnx 依赖项。

```bash
pip install infinity_emb[torch,optimum]
```

```python
documents = [
    "法棍是一道菜。",
    "巴黎是法国的首都。",
    "numpy 是用于线性代数的库",
    "你逃脱了我逃脱的东西 - 你也会在巴黎玩得很疯狂",
]
query = "巴黎在哪里？"
```

```python
embeddings = InfinityEmbeddingsLocal(
    model="sentence-transformers/all-MiniLM-L6-v2",
    # revision
    revision=None,
    # 最好保持在 32
    batch_size=32,
    # 适用于 AMD/Nvidia GPU 的 torch
    device="cuda",
    # 执行前先热身模型
)
```

```output
/home/michael/langchain/libs/langchain/.venv/lib/python3.10/site-packages/tqdm/auto.py:21: TqdmWarning: IProgress not found. Please update jupyter and ipywidgets. See https://ipywidgets.readthedocs.io/en/stable/user_install.html
  from .autonotebook import tqdm as notebook_tqdm
The BetterTransformer implementation does not support padding during training, as the fused kernels do not support attention masks. Beware that passing padded batched data during training may result in unexpected outputs. Please refer to https://huggingface.co/docs/optimum/bettertransformer/overview for more details.
/home/michael/langchain/libs/langchain/.venv/lib/python3.10/site-packages/optimum/bettertransformer/models/encoder_models.py:301: UserWarning: The PyTorch API of nested tensors is in prototype stage and will change in the near future. (Triggered internally at ../aten/src/ATen/NestedTensorImpl.cpp:177.)
  hidden_states = torch._nested_tensor_from_mask(hidden_states, ~attention_mask)
```

```python
# 运行异步代码，可以按照您的喜好进行操作
# 如果您在 jupyter notebook 中，可以使用以下方法
documents_embedded, query_result = await embed()
```

```python
# (演示) 计算相似度
import numpy as np
scores = np.array(documents_embedded) @ np.array(query_result).T
dict(zip(documents, scores))
```

# 选项 2：运行服务器，并通过 API 连接

#### 可选：确保启动 Infinity 实例

要安装 infinity，请使用以下命令。有关更多详细信息，请查看 [Github 上的文档](https://github.com/michaelfeil/infinity)。

```bash
pip install infinity_emb[all]
```

```python
# 安装 infinity 包
%pip install --upgrade --quiet  infinity_emb[all]
```

启动服务器 - 最好从单独的终端进行，而不是在 Jupyter Notebook 中进行

```bash
model=sentence-transformers/all-MiniLM-L6-v2
port=7797
infinity_emb --port $port --model-name-or-path $model
```

或者可以使用 docker：

```bash
model=sentence-transformers/all-MiniLM-L6-v2
port=7797
docker run -it --gpus all -p $port:$port michaelf34/infinity:latest --model-name-or-path $model --port $port
```

## 使用您的 Infinity 实例嵌入您的文档

```python
documents = [
    "法棍是一道菜。",
    "巴黎是法国的首都。",
    "numpy 是用于线性代数的库",
    "你逃脱了我逃脱的东西 - 你也会在巴黎玩得很疯狂",
]
query = "巴黎在哪里？"
```

```python
#
infinity_api_url = "http://localhost:7797/v1"
# model is currently not validated.
embeddings = InfinityEmbeddings(
    model="sentence-transformers/all-MiniLM-L6-v2", infinity_api_url=infinity_api_url
)
try:
    documents_embedded = embeddings.embed_documents(documents)
    query_result = embeddings.embed_query(query)
    print("embeddings created successful")
except Exception as ex:
    print(
        "Make sure the infinity instance is running. Verify by clicking on "
        f"{infinity_api_url.replace('v1','docs')} Exception: {ex}. "
    )
```

确保无限实例正在运行。通过单击 http://localhost:7797/docs 进行验证。异常: HTTPConnectionPool(host='localhost', port=7797): Max retries exceeded with url: /v1/embeddings (Caused by NewConnectionError('<urllib3.connection.HTTPConnection object at 0x7f91c35dbd30>: Failed to establish a new connection: [Errno 111] Connection refused'))。

```python
# (演示) 计算相似度
import numpy as np
scores = np.array(documents_embedded) @ np.array(query_result).T
dict(zip(documents, scores))
```

{'Baguette is a dish.': 0.31344215908661155,

 'Paris is the capital of France.': 0.8148670296896388,

 'numpy is a lib for linear algebra': 0.004429399861302009,

 "You escaped what I've escaped - You'd be in Paris getting fucked up too": 0.5088476180154582}

```