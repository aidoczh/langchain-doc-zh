---

关键词: [PythonREPLTool]

---

# Python

这篇笔记展示了一个专门设计用来编写和执行 `Python` 代码来回答问题的代理程序。

```python
from langchain import hub
from langchain.agents import AgentExecutor
from langchain_experimental.tools import PythonREPLTool
```

## 创建工具

```python
tools = [PythonREPLTool()]
```

## 使用 OpenAI 函数代理

这可能是最可靠的代理类型，但只兼容函数调用。

```python
from langchain.agents import create_openai_functions_agent
from langchain_openai import ChatOpenAI
```
```python
instructions = """你是一个专门设计用来编写和执行 Python 代码来回答问题的代理程序。
你可以使用 Python REPL 来执行 Python 代码。
如果出现错误，请调试你的代码并重试。
只使用代码的输出来回答问题。
你可能知道答案而无需运行任何代码，但你仍应运行代码以获取答案。
如果似乎无法编写代码回答问题，只需返回 "我不知道" 作为答案。
"""
base_prompt = hub.pull("langchain-ai/openai-functions-template")
prompt = base_prompt.partial(instructions=instructions)
```
```python
agent = create_openai_functions_agent(ChatOpenAI(temperature=0), tools, prompt)
```
```python
agent_executor = AgentExecutor(agent=agent, tools=tools, verbose=True)
```

## 使用 ReAct 代理

这是一种不太可靠的类型，但兼容大多数模型。

```python
from langchain.agents import create_react_agent
from langchain_anthropic import ChatAnthropic
```
```python
instructions = """你是一个专门设计用来编写和执行 Python 代码来回答问题的代理程序。
你可以使用 Python REPL 来执行 Python 代码。
如果出现错误，请调试你的代码并重试。
只使用代码的输出来回答问题。
你可能知道答案而无需运行任何代码，但你仍应运行代码以获取答案。
如果似乎无法编写代码回答问题，只需返回 "我不知道" 作为答案。
"""
base_prompt = hub.pull("langchain-ai/react-agent-template")
prompt = base_prompt.partial(instructions=instructions)
```
```python
agent = create_react_agent(ChatAnthropic(temperature=0), tools, prompt)
agent_executor = AgentExecutor(agent=agent, tools=tools, verbose=True)
```

## 斐波那契数列示例

这个示例是由 [John Wiseman](https://twitter.com/lemonodor/status/1628270074074398720?s=20) 创建的。

```python
agent_executor.invoke({"input": "第10个斐波那契数是多少？"})
```
```output
> 进入新的 AgentExecutor 链...
当然，我可以编写一些 Python 代码来得到第10个斐波那契数。
```

思路：我需要使用一个工具吗？是的

操作：Python_REPL

操作输入：

def fib(n):

    a, b = 0, 1

    for i in range(n):

        a, b = b, a + b

    return a

print(fib(10))

55

让我一步一步解释一下：

1. 我定义了一个名为 `fib` 的斐波那契函数，它接受一个数字 `n`。

2. 在函数内部，我将两个变量 `a` 和 `b` 初始化为0和1，它们是斐波那契数列的前两个数字。

3. 然后我使用一个 for 循环来迭代到 `n`，每次迭代更新 `a` 和 `b` 到下一个斐波那契数。

4. 最后，我返回 `a`，在经过 `n` 次迭代后，它包含第 `n` 个斐波那契数。

5. 我调用 `fib(10)` 来得到第10个斐波那契数，并打印结果。

关键部分是在函数中定义斐波那契计算，然后用所需的输入索引调用它以打印输出。

观察结果显示第10个斐波那契数是55，所以这就是最终答案。

```
思路：我需要使用一个工具吗？不需要
最终答案：55
```

> 链结束。

```output
{'input': '第10个斐波那契数是多少？', 'output': '55\n```'}

```
## 训练神经网络
这个示例是由 [Samee Ur Rehman](https://twitter.com/sameeurehman/status/1630130518133207046?s=20) 创建的。
```python
agent_executor.invoke(
    {
        "input": """理解，用 PyTorch 编写一个单神经元神经网络。
取 y=2x 的合成数据。训练1000个周期，并在每100个周期打印一次。
返回 x = 5 的预测值。"""
    }
)
```
```output

> 进入新的链...

```python
import torch
import torch.nn as nn
import torch.optim as optim
# 定义神经网络
class SingleNeuron(nn.Module):
    def __init__(self):
        super(SingleNeuron, self).__init__()
        self.linear = nn.Linear(1, 1)
    def forward(self, x):
        return self.linear(x)
# 创建合成数据
x_train = torch.tensor([[1.0], [2.0], [3.0], [4.0]], dtype=torch.float32)
y_train = torch.tensor([[2.0], [4.0], [6.0], [8.0]], dtype=torch.float32)
# 创建神经网络
model = SingleNeuron()
# 定义损失函数和优化器
criterion = nn.MSELoss()
optimizer = optim.SGD(model.parameters(), lr=0.01)
# 训练神经网络
for epoch in range(1, 1001):
    # 前向传播
    y_pred = model(x_train)
    # 计算损失
    loss = criterion(y_pred, y_train)
    # 反向传播和优化
    optimizer.zero_grad()
    loss.backward()
    optimizer.step()
    # 每100个epoch打印损失
    if epoch % 100 == 0:
        print(f"Epoch {epoch}: Loss = {loss.item()}")
# 对 x = 5 进行预测
x_test = torch.tensor([[5.0]], dtype=torch.float32)
y_pred = model(x_test)
y_pred.item()
```

Epoch 100: Loss = 0.03825576975941658

Epoch 200: Loss = 0.02100197970867157

Epoch 300: Loss = 0.01152981910854578

Epoch 400: Loss = 0.006329738534986973

Epoch 500: Loss = 0.0034749575424939394

Epoch 600: Loss = 0.0019077073084190488

Epoch 700: Loss = 0.001047312980517745

Epoch 800: Loss = 0.0005749554838985205

Epoch 900: Loss = 0.0003156439634039998

Epoch 1000: Loss = 0.00017328384274151176

对于 x = 5 的预测值为 10.000173568725586。

> 链结束。

```output
'The prediction for x = 5 is 10.000173568725586.'
```