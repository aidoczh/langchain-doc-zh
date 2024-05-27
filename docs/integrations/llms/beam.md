# Beam

调用 Beam API 包装器来部署和对 gpt2 语言模型的实例进行后续调用，部署在云端。需要安装 Beam 库并注册 Beam 客户端 ID 和客户端密钥。通过调用包装器，创建并运行模型实例，返回与提示相关的文本。然后可以通过直接调用 Beam API 进行额外的调用。

[创建账户](https://www.beam.cloud/)，如果还没有账户的话。从 [仪表板](https://www.beam.cloud/dashboard/settings/api-keys) 获取你的 API 密钥。

安装 Beam CLI

```python
!curl https://raw.githubusercontent.com/slai-labs/get-beam/main/get-beam.sh -sSfL | sh
```

注册 API 密钥并设置你的 Beam 客户端 ID 和密钥环境变量:

```python
import os
beam_client_id = "<Your beam client id>"
beam_client_secret = "<Your beam client secret>"
# 设置环境变量
os.environ["BEAM_CLIENT_ID"] = beam_client_id
os.environ["BEAM_CLIENT_SECRET"] = beam_client_secret
# 运行 beam 配置命令
!beam configure --clientId={beam_client_id} --clientSecret={beam_client_secret}
```

安装 Beam SDK:

```python
%pip install --upgrade --quiet  beam-sdk
```

**直接从 langchain 部署和调用 Beam！**

请注意，冷启动可能需要几分钟才能返回响应，但后续调用将更快！

```python
from langchain_community.llms.beam import Beam
llm = Beam(
    model_name="gpt2",
    name="langchain-gpt2-test",
    cpu=8,
    memory="32Gi",
    gpu="A10G",
    python_version="python3.8",
    python_packages=[
        "diffusers[torch]>=0.10",
        "transformers",
        "torch",
        "pillow",
        "accelerate",
        "safetensors",
        "xformers",
    ],
    max_length="50",
    verbose=False,
)
llm._deploy()
response = llm._call("Running machine learning on a remote GPU")
print(response)
```