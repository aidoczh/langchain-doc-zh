# 阿里云 PAI EAS

[阿里云机器学习平台](https://www.alibabacloud.com/help/en/pai)（PAI）是专为企业和开发人员设计的机器学习或深度学习工程平台。它提供易于使用、成本效益高、性能卓越且易于扩展的插件，可应用于各种行业场景。`机器学习平台 for AI` 提供了超过 140 种内置优化算法，包括数据标注（`PAI-iTAG`）、模型构建（`PAI-Designer` 和 `PAI-DSW`）、模型训练（`PAI-DLC`）、编译优化和推理部署（`PAI-EAS`）等全流程 AI 工程能力。`PAI-EAS` 支持不同类型的硬件资源，包括 CPU 和 GPU，并具有高吞吐量和低延迟。它允许您通过几次点击部署大规模复杂模型，并实时执行弹性的缩放进和缩放出。此外，它还提供了全面的运维和监控系统。

```python
from langchain.chains import LLMChain
from langchain_community.llms.pai_eas_endpoint import PaiEasEndpoint
from langchain_core.prompts import PromptTemplate
template = """问题：{question}
回答：让我们一步一步来思考。"""
prompt = PromptTemplate.from_template(template)
```

想要使用 EAS LLMs 的用户必须首先设置 EAS 服务。当 EAS 服务启动时，可以获得 `EAS_SERVICE_URL` 和 `EAS_SERVICE_TOKEN`。用户可以参考 https://www.alibabacloud.com/help/en/pai/user-guide/service-deployment/ 获取更多信息。

```python
import os
os.environ["EAS_SERVICE_URL"] = "Your_EAS_Service_URL"
os.environ["EAS_SERVICE_TOKEN"] = "Your_EAS_Service_Token"
llm = PaiEasEndpoint(
    eas_service_url=os.environ["EAS_SERVICE_URL"],
    eas_service_token=os.environ["EAS_SERVICE_TOKEN"],
)
```

```python
llm_chain = prompt | llm
question = "Justin Bieber 出生年份的超级碗冠军是哪支 NFL 球队？"
llm_chain.invoke({"question": question})
```

```output
'  谢谢您的提问！但是，我必须尊重地指出，这个问题存在错误。Justin Bieber 出生于 1994 年，而超级碗是在 1967 年首次举行的。因此，任何 NFL 球队都不可能在 Justin Bieber 出生的那一年赢得超级碗。\n\n希望这能澄清事情！如果您有其他问题，请随时提问。'
```