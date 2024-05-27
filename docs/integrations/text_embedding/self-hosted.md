# 自托管

让我们加载 `SelfHostedEmbeddings`、`SelfHostedHuggingFaceEmbeddings` 和 `SelfHostedHuggingFaceInstructEmbeddings` 类。

```python
import runhouse as rh
from langchain_community.embeddings import (
    SelfHostedEmbeddings,
    SelfHostedHuggingFaceEmbeddings,
    SelfHostedHuggingFaceInstructEmbeddings,
)
```

```python
# 对于 GCP、Azure 或 Lambda 上的按需 A100
gpu = rh.cluster(name="rh-a10x", instance_type="A100:1", use_spot=False)
# 对于 AWS 上的按需 A10G（AWS 上没有单个 A100）
# gpu = rh.cluster(name='rh-a10x', instance_type='g5.2xlarge', provider='aws')
# 对于现有集群
# gpu = rh.cluster(ips=['<集群的 IP>'],
#                  ssh_creds={'ssh_user': '...', 'ssh_private_key':'<密钥路径>'},
#                  name='my-cluster')
```

```python
embeddings = SelfHostedHuggingFaceEmbeddings(hardware=gpu)
```

```python
text = "这是一个测试文档。"
```

```python
query_result = embeddings.embed_query(text)
```

类似地，对于 `SelfHostedHuggingFaceInstructEmbeddings`：

```python
embeddings = SelfHostedHuggingFaceInstructEmbeddings(hardware=gpu)
```

现在让我们使用自定义加载函数加载嵌入模型：

```python
def get_pipeline():
    from transformers import (
        AutoModelForCausalLM,
        AutoTokenizer,
        pipeline,
    )
    model_id = "facebook/bart-base"
    tokenizer = AutoTokenizer.from_pretrained(model_id)
    model = AutoModelForCausalLM.from_pretrained(model_id)
    return pipeline("feature-extraction", model=model, tokenizer=tokenizer)
def inference_fn(pipeline, prompt):
    # 返回模型的最后隐藏状态
    if isinstance(prompt, list):
        return [emb[0][-1] for emb in pipeline(prompt)]
    return pipeline(prompt)[0][-1]
```

```python
embeddings = SelfHostedEmbeddings(
    model_load_fn=get_pipeline,
    hardware=gpu,
    model_reqs=["./", "torch", "transformers"],
    inference_fn=inference_fn,
)
```

```python
query_result = embeddings.embed_query(text)
```