# UpTrain

[UpTrain](https://uptrain.ai/) 是一个开源的统一平台，用于评估和改进生成式人工智能应用。它提供了超过20个预配置评估项目（涵盖语言、代码、嵌入式用例），对失败案例进行根本原因分析，并提供解决方案的见解。

## 安装和设置

```bash
pip install uptrain
```

## 回调函数

```python
from langchain_community.callbacks.uptrain_callback import UpTrainCallbackHandler
```

查看一个[示例](/docs/integrations/callbacks/uptrain)。