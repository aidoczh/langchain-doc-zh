# 人类作为工具

人类是AGI，因此可以被用作工具来帮助AI代理当它感到困惑时。

```python
from langchain.agents import AgentType, initialize_agent, load_tools
from langchain_openai import ChatOpenAI, OpenAI
llm = ChatOpenAI(temperature=0.0)
math_llm = OpenAI(temperature=0.0)
tools = load_tools(
    ["human", "llm-math"],
    llm=math_llm,
)
agent_chain = initialize_agent(
    tools,
    llm,
    agent=AgentType.ZERO_SHOT_REACT_DESCRIPTION,
    verbose=True,
)
```

在上述代码中，您可以看到该工具直接从命令行接收输入。您可以根据需要自定义`prompt_func`和`input_func`（如下所示）。

```python
agent_chain.run("我的朋友Eric的姓是什么？")
# 回答应为'Zhu'
```

```output
> 进入新的AgentExecutor链...
我不知道Eric的姓，所以我应该向人类寻求指导。
动作：人类
动作输入："Eric的姓是什么？"
Eric的姓是什么？
``````output
 Zhu
``````output
观察：Zhu
思考：我现在知道Eric的姓是Zhu。
最终答案：Eric的姓是Zhu。
> 链结束。
```

```output
"Eric的姓是Zhu。"
```

## 配置输入函数

默认情况下，`HumanInputRun`工具使用Python的`input`函数从用户那里获取输入。您可以自定义`input_func`为任何您喜欢的内容。例如，如果您想接受多行输入，可以执行以下操作：

```python
def get_input() -> str:
    print("插入您的文本。输入'q'或按Ctrl-D（或Windows上的Ctrl-Z）结束。")
    contents = []
    while True:
        try:
            line = input()
        except EOFError:
            break
        if line == "q":
            break
        contents.append(line)
    return "\n".join(contents)
# 加载工具时可以修改
tools = load_tools(["human", "ddg-search"], llm=math_llm, input_func=get_input)
```

```python
# 或者您可以直接实例化该工具
from langchain_community.tools import HumanInputRun
tool = HumanInputRun(input_func=get_input)
```

```python
agent_chain = initialize_agent(
    tools,
    llm,
    agent=AgentType.ZERO_SHOT_REACT_DESCRIPTION,
    verbose=True,
)
```

```python
agent_chain.run("我需要帮助归因一句引语")
```

```output
> 进入新的AgentExecutor链...
我应该向人类寻求指导
动作：人类
动作输入："你能帮我归因一句引语吗？"
你能帮我归因一句引语吗？
插入您的文本。输入'q'或按Ctrl-D（或Windows上的Ctrl-Z）结束。
``````output
 vini
 vidi
 vici
 q
``````output
观察：vini
vidi
vici
思考：我需要提供关于引语的更多背景信息
动作：人类
动作输入："引语是'Veni, vidi, vici'"
引语是'Veni, vidi, vici'
插入您的文本。输入'q'或按Ctrl-D（或Windows上的Ctrl-Z）结束。
``````output
 哦，谁说的
 q
``````output
观察：哦，谁说的
思考：我可以使用DuckDuckGo搜索找出是谁说的这句引语
动作：DuckDuckGo搜索
动作输入："谁说过'Veni, vidi, vici'?"
观察：更新于2019年9月6日。据说罗马皇帝尤利乌斯·恺撒（公元前100-公元前44年）曾说过“Veni, vidi, vici”这句著名的话，这种时髦的吹嘘让他当时以及后来的许多作家都印象深刻。这句话的大致意思是“我来了，我看到了，我征服了”，大约可以发音为Vehnee，Veedee ... Veni, vidi, vici（古典拉丁语：[weːniː wiːdiː wiːkiː]，教会拉丁语：[ˈveni ˈvidi ˈvitʃi]；“我来了；我看到了；我征服了”）是一个拉丁短语，用来指代迅速、决定性的胜利。这句话通常被归因于尤利乌斯·恺撒，根据阿庇安的说法，他在公元前47年取得快速胜利后在写给罗马参议院的一封信中使用了这句话... veni, vidi, vici 尤利乌斯·恺撒拉丁语引文 ve· ni, vi· di, vi· ci ˌwā-nē ˌwē-dē ˈwē-kē ˌvā-nē ˌvē-dē ˈvē-chē：我来了，我看到了，我征服了与veni, vidi, vici相关的文章“在酒中见真理”等拉丁语... veni, vidi, vici Venite veni, vidi, vici Venizélos 更多附近词条引用此条目风格veni, vidi, vici是一句广为人知且频繁引用的拉丁表达，几个世纪以来被用作胜利的表达。据说这句话是恺撒在庆祝胜利时说的。
思考：我现在知道最终答案
最终答案：尤利乌斯·恺撒说过引语“Veni, vidi, vici”，意为“我来了，我看到了，我征服了”。
> 链结束。
```

```output
'尤利乌斯·恺撒说过引语“Veni, vidi, vici”，意为“我来了，我看到了，我征服了”。'
```