# OCI数据科学模型部署端点

[OCI数据科学](https://docs.oracle.com/en-us/iaas/data-science/using/home.htm) 是一款完全托管且无服务器的平台，供数据科学团队在Oracle云基础设施中构建、训练和管理机器学习模型。

本笔记介绍了如何使用托管在[OCI数据科学模型部署](https://docs.oracle.com/en-us/iaas/data-science/using/model-dep-about.htm)上的LLM。

为了进行身份验证，使用了[oracle-ads](https://accelerated-data-science.readthedocs.io/en/latest/user_guide/cli/authentication.html) 来自动加载调用端点的凭据。

```python
!pip3 install oracle-ads
```

## 先决条件

### 部署模型

查看[Oracle GitHub示例存储库](https://github.com/oracle-samples/oci-data-science-ai-samples/tree/main/model-deployment/containers/llama2) 来了解如何在OCI数据科学模型部署上部署您的LLM。

### 策略

确保具有访问OCI数据科学模型部署端点所需的[策略](https://docs.oracle.com/en-us/iaas/data-science/using/model-dep-policies-auth.htm#model_dep_policies_auth__predict-endpoint)。

## 设置

### vLLM

在部署模型后，您必须设置`OCIModelDeploymentVLLM`调用的以下必需参数：

- **`endpoint`**: 部署模型的模型HTTP端点，例如 `https://<MD_OCID>/predict`。 

- **`model`**: 模型的位置。

### 文本生成推理（TGI）

您必须设置`OCIModelDeploymentTGI`调用的以下必需参数：

- **`endpoint`**: 部署模型的模型HTTP端点，例如 `https://<MD_OCID>/predict`。 

### 身份验证

您可以通过ads或环境变量设置身份验证。当您在OCI数据科学笔记本会话中工作时，可以利用资源主体来访问其他OCI资源。查看[这里](https://accelerated-data-science.readthedocs.io/en/latest/user_guide/cli/authentication.html) 以查看更多选项。

## 示例

```python
import ads
from langchain_community.llms import OCIModelDeploymentVLLM
# 通过ads设置身份验证
# 在配置了基于资源主体的身份验证的OCI服务中使用
ads.set_auth("resource_principal")
# 创建OCI模型部署端点的实例
# 用您自己的端点URI和模型名称替换
llm = OCIModelDeploymentVLLM(endpoint="https://<MD_OCID>/predict", model="model_name")
# 运行LLM
llm.invoke("Who is the first president of United States?")
```

```python
import os
from langchain_community.llms import OCIModelDeploymentTGI
# 通过环境变量设置身份验证
# 在从本地工作站或不支持资源主体的平台上工作时使用API密钥设置
os.environ["OCI_IAM_TYPE"] = "api_key"
os.environ["OCI_CONFIG_PROFILE"] = "default"
os.environ["OCI_CONFIG_LOCATION"] = "~/.oci"
# 通过环境变量设置端点
# 用您自己的端点URI替换
os.environ["OCI_LLM_ENDPOINT"] = "https://<MD_OCID>/predict"
# 创建OCI模型部署端点的实例
llm = OCIModelDeploymentTGI()
# 运行LLM
llm.invoke("Who is the first president of United States?")
```