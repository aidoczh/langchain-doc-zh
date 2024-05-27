# Runhouse

[Runhouse](https://github.com/run-house/runhouse) 允许在不同环境和用户之间进行远程计算和数据交互。请查看[Runhouse文档](https://runhouse-docs.readthedocs-hosted.com/en/latest/)。

这个示例演示了如何使用 LangChain 和 [Runhouse](https://github.com/run-house/runhouse) 与托管在您自己的 GPU 上的模型，或者在 AWS、GCP、AWS 或 Lambda 上的按需 GPU 进行交互。

**注意**：代码中使用 `SelfHosted` 而不是 `Runhouse`。

```python
%pip install --upgrade --quiet  runhouse
```

```python
import runhouse as rh
from langchain.chains import LLMChain
from langchain_community.llms import SelfHostedHuggingFaceLLM, SelfHostedPipeline
from langchain_core.prompts import PromptTemplate
```

```output
INFO | 2023-04-17 16:47:36,173 | 未提供身份验证令牌，因此不使用 RNS API 来保存和加载配置
```

```python
# 对于在 GCP、Azure 或 Lambda 上按需使用 A100
gpu = rh.cluster(name="rh-a10x", instance_type="A100:1", use_spot=False)
# 对于在 AWS 上按需使用 A10G（AWS 上没有单个 A100）
# gpu = rh.cluster(name='rh-a10x', instance_type='g5.2xlarge', provider='aws')
# 对于现有集群
# gpu = rh.cluster(ips=['<集群的 IP>'],
#                  ssh_creds={'ssh_user': '...', 'ssh_private_key':'<密钥路径>'},
#                  name='rh-a10x')
```

```python
template = """问题: {question}
回答: 让我们逐步思考。"""
prompt = PromptTemplate.from_template(template)
```

```python
llm = SelfHostedHuggingFaceLLM(
    model_id="gpt2", hardware=gpu, model_reqs=["pip:./", "transformers", "torch"]
)
```

```python
llm_chain = LLMChain(prompt=prompt, llm=llm)
```

```python
question = "贾斯汀·比伯出生年份的超级碗冠军是哪支NFL球队？"
llm_chain.run(question)
```

```output
INFO | 2023-02-17 05:42:23,537 | 通过 gRPC 运行 _generate_text
INFO | 2023-02-17 05:42:24,016 | 发送消息所需时间：0.48 秒
```

```output
"\n\n假设我们正在讨论在贾斯汀·比伯出生年份赢得超级碗冠军的体育队伍"
```

您还可以通过 SelfHostedHuggingFaceLLM 接口加载更多自定义模型：

```python
llm = SelfHostedHuggingFaceLLM(
    model_id="google/flan-t5-small",
    task="text2text-generation",
    hardware=gpu,
)
```

```python
llm("德国的首都是哪里？")
```

```output
INFO | 2023-02-17 05:54:21,681 | 通过 gRPC 运行 _generate_text
INFO | 2023-02-17 05:54:21,937 | 发送消息所需时间：0.25 秒
```

```output
'柏林'
```

通过自定义加载函数，我们可以直接在远程硬件上加载自定义管道：

```python
def load_pipeline():
    from transformers import (
        AutoModelForCausalLM,
        AutoTokenizer,
        pipeline,
    )
    model_id = "gpt2"
    tokenizer = AutoTokenizer.from_pretrained(model_id)
    model = AutoModelForCausalLM.from_pretrained(model_id)
    pipe = pipeline(
        "text-generation", model=model, tokenizer=tokenizer, max_new_tokens=10
    )
    return pipe
def inference_fn(pipeline, prompt, stop=None):
    return pipeline(prompt)[0]["generated_text"][len(prompt) :]
```

```python
llm = SelfHostedHuggingFaceLLM(
    model_load_fn=load_pipeline, hardware=gpu, inference_fn=inference_fn
)
```

```python
llm("现任美国总统是谁？")
```

```output
INFO | 2023-02-17 05:42:59,219 | 通过 gRPC 运行 _generate_text
INFO | 2023-02-17 05:42:59,522 | 发送消息所需时间：0.3 秒
```

```output
'约翰·W·布什'
```

您可以直接将管道发送到您的模型，但这仅适用于小型模型（<2 Gb），速度会比较慢：

```python
pipeline = load_pipeline()
llm = SelfHostedPipeline.from_pipeline(
    pipeline=pipeline, hardware=gpu, model_reqs=["pip:./", "transformers", "torch"]
)
```

相反，我们也可以将其发送到硬件的文件系统，这样会快得多。

```python
import pickle
rh.blob(pickle.dumps(pipeline), path="models/pipeline.pkl").save().to(
    gpu, path="models"
)
llm = SelfHostedPipeline.from_pipeline(pipeline="models/pipeline.pkl", hardware=gpu)
```