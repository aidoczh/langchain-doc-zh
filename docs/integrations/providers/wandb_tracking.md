# 权重和偏差

本笔记将介绍如何将您的 LangChain 实验跟踪到一个集中的权重和偏差仪表板中。要了解更多关于提示工程和回调的信息，请参考这份报告，该报告解释了两者以及您可以期望看到的结果仪表板。

[查看报告](https://wandb.ai/a-sh0ts/langchain_callback_demo/reports/Prompt-Engineering-LLMs-with-LangChain-and-W-B--VmlldzozNjk1NTUw#👋-how-to-build-a-callback-in-langchain-for-better-prompt-engineering)

**注意**：_`WandbCallbackHandler` 将被弃用，建议使用 `WandbTracer`。未来，请使用 `WandbTracer`，因为它更灵活，允许更细粒度的日志记录。要了解有关 `WandbTracer` 的更多信息，请参阅 [agent_with_wandb_tracing](http://wandb.me/prompts-quickstart) 笔记或使用以下 [colab 笔记](http://wandb.me/prompts-quickstart)。要了解有关权重和偏差提示的更多信息，请参阅以下 [提示文档](https://docs.wandb.ai/guides/prompts)。

```python
%pip install --upgrade --quiet  wandb
%pip install --upgrade --quiet  pandas
%pip install --upgrade --quiet  textstat
%pip install --upgrade --quiet  spacy
!python -m spacy download en_core_web_sm
```
```python
import os
os.environ["WANDB_API_KEY"] = ""
# os.environ["OPENAI_API_KEY"] = ""
# os.environ["SERPAPI_API_KEY"] = ""
```
```python
from datetime import datetime
from langchain_community.callbacks import WandbCallbackHandler
from langchain_core.callbacks import StdOutCallbackHandler
from langchain_openai import OpenAI
```
```
用于记录权重和偏差的回调处理程序。
参数：
    job_type (str): 作业类型。
    project (str): 要记录的项目。
    entity (str): 要记录的实体。
    tags (list): 要记录的标签。
    group (str): 要记录的组。
    name (str): 运行的名称。
    notes (str): 要记录的注释。
    visualize (bool): 是否可视化运行。
    complexity_metrics (bool): 是否记录复杂度指标。
    stream_logs (bool): 是否将回调操作流式传输到 W&B
```
```
WandbCallbackHandler(...) 的默认值
visualize: bool = False,
complexity_metrics: bool = False,
stream_logs: bool = False,
```

**注意**：对于 beta 工作流程，我们已经基于 textstat 进行了默认分析，并基于 spacy 进行了可视化。

```python
"""主要函数。
此函数用于尝试回调处理程序。
场景：
1. OpenAI LLM
2. 在多代上使用多个 SubChains 进行链式处理
3. 带有工具的代理
"""
session_group = datetime.now().strftime("%m.%d.%Y_%H.%M.%S")
wandb_callback = WandbCallbackHandler(
    job_type="inference",
    project="langchain_callback_demo",
    group=f"minimal_{session_group}",
    name="llm",
    tags=["test"],
)
callbacks = [StdOutCallbackHandler(), wandb_callback]
llm = OpenAI(temperature=0, callbacks=callbacks)
```
```output
wandb: 当前登录为: harrison-chase。使用 `wandb login --relogin` 强制重新登录
```
```html
使用 wandb 版本 0.14.0 跟踪运行数据
```
```html
运行数据已本地保存在 <code>/Users/harrisonchase/workplace/langchain/docs/ecosystem/wandb/run-20230318_150408-e47j1914</code>
```
```html
将运行 <strong><a href='https://wandb.ai/harrison-chase/langchain_callback_demo/runs/e47j1914' target="_blank">llm</a></strong> 同步到 <a href='https://wandb.ai/harrison-chase/langchain_callback_demo' target="_blank">权重和偏差</a> (<a href='https://wandb.me/run' target="_blank">文档</a>)<br/>
```
```html
在 <a href='https://wandb.ai/harrison-chase/langchain_callback_demo' target="_blank">https://wandb.ai/harrison-chase/langchain_callback_demo</a> 查看项目
```
```html
在 <a href='https://wandb.ai/harrison-chase/langchain_callback_demo/runs/e47j1914' target="_blank">https://wandb.ai/harrison-chase/langchain_callback_demo/runs/e47j1914</a> 查看运行
```
```output
wandb: 警告：wandb 回调当前处于 beta 阶段，可能会根据 `langchain` 的更新而更改。请将任何问题报告到 https://github.com/wandb/wandb/issues，并使用标签 `langchain`。
```
```
# WandbCallbackHandler.flush_tracker(...) 的默认值
reset: bool = True,
finish: bool = False,
```

`flush_tracker` 函数用于将 LangChain 会话记录到权重和偏差。它至少记录了提示和生成，以及 LangChain 模块的序列化形式到指定的权重和偏差项目。默认情况下，我们重置会话，而不是直接结束会话。

```python
# SCENARIO 1 - LLM
llm_result = llm.generate(["Tell me a joke", "Tell me a poem"] * 3)
wandb_callback.flush_tracker(llm, name="simple_sequential")
```
```html
等待 W&B 进程完成... <strong style="color:green">(成功)。</strong> 
```
```html
在此处查看运行 <strong style="color:#cdcd00">llm</strong>： <a href='https://wandb.ai/harrison-chase/langchain_callback_demo/runs/e47j1914' target="_blank">https://wandb.ai/harrison-chase/langchain_callback_demo/runs/e47j1914</a><br/>同步了 5 个 W&B 文件，2 个媒体文件，5 个 artifact 文件和 0 个其他文件 
```
```html
在此处找到日志：<code>./wandb/run-20230318_150408-e47j1914/logs</code> 
```
```output
VBox(children=(Label(value='等待 wandb.init()...\r'), FloatProgress(value=0.016745895149999985, max=1.0…
```
```html
使用 wandb 版本 0.14.0 追踪运行数据 
```
```html
运行数据保存在本地：<code>/Users/harrisonchase/workplace/langchain/docs/ecosystem/wandb/run-20230318_150534-jyxma7hu</code> 
```
```html
将运行 <strong><a href='https://wandb.ai/harrison-chase/langchain_callback_demo/runs/jyxma7hu' target="_blank">simple_sequential</a></strong> 同步到 <a href='https://wandb.ai/harrison-chase/langchain_callback_demo' target="_blank">Weights & Biases</a> (<a href='https://wandb.me/run' target="_blank">文档</a>)<br/> 
```
```html
在此处查看项目：<a href='https://wandb.ai/harrison-chase/langchain_callback_demo' target="_blank">https://wandb.ai/harrison-chase/langchain_callback_demo</a> 
```
```html
在此处查看运行：<a href='https://wandb.ai/harrison-chase/langchain_callback_demo/runs/jyxma7hu' target="_blank">https://wandb.ai/harrison-chase/langchain_callback_demo/runs/jyxma7hu</a> 
```
```python
from langchain.chains import LLMChain
from langchain_core.prompts import PromptTemplate
```
```python
# SCENARIO 2 - Chain
template = """你是一位剧作家。根据剧名，你的任务是为该剧写一个简介。
剧名：{title}
剧作家：这是上述剧的简介："""
prompt_template = PromptTemplate(input_variables=["title"], template=template)
synopsis_chain = LLMChain(llm=llm, prompt=prompt_template, callbacks=callbacks)
test_prompts = [
    {
        "title": "关于推动游戏设计边界的优秀视频游戏的纪录片"
    },
    {"title": "可卡因熊对海洛因狼"},
    {"title": "最佳 MLOps 工具"},
]
synopsis_chain.apply(test_prompts)
wandb_callback.flush_tracker(synopsis_chain, name="agent")
```
```html
等待 W&B 进程完成... <strong style="color:green">(成功)。</strong> 
```
```html
在此处查看运行 <strong style="color:#cdcd00">simple_sequential</strong>： <a href='https://wandb.ai/harrison-chase/langchain_callback_demo/runs/jyxma7hu' target="_blank">https://wandb.ai/harrison-chase/langchain_callback_demo/runs/jyxma7hu</a><br/>同步了 4 个 W&B 文件，2 个媒体文件，6 个 artifact 文件和 0 个其他文件 
```
```html
在此处找到日志：<code>./wandb/run-20230318_150534-jyxma7hu/logs</code> 
```
```output
VBox(children=(Label(value='等待 wandb.init()...\r'), FloatProgress(value=0.016736786816666675, max=1.0…
```
```html
使用 wandb 版本 0.14.0 追踪运行数据 
```
```html
运行数据保存在本地：<code>/Users/harrisonchase/workplace/langchain/docs/ecosystem/wandb/run-20230318_150550-wzy59zjq</code> 
```
```html
将运行 <strong><a href='https://wandb.ai/harrison-chase/langchain_callback_demo/runs/wzy59zjq' target="_blank">agent</a></strong> 同步到 <a href='https://wandb.ai/harrison-chase/langchain_callback_demo' target="_blank">Weights & Biases</a> (<a href='https://wandb.me/run' target="_blank">文档</a>)<br/> 
```
```html
在此处查看项目：<a href='https://wandb.ai/harrison-chase/langchain_callback_demo' target="_blank">https://wandb.ai/harrison-chase/langchain_callback_demo</a> 
```
```html
在此处查看运行：<a href='https://wandb.ai/harrison-chase/langchain_callback_demo/runs/wzy59zjq' target="_blank">https://wandb.ai/harrison-chase/langchain_callback_demo/runs/wzy59zjq</a> 
```
```python
from langchain.agents import AgentType, initialize_agent, load_tools
```
```python
# SCENARIO 3 - Agent with Tools
tools = load_tools(["serpapi", "llm-math"], llm=llm)
agent = initialize_agent(
    tools,
    llm,
    agent=AgentType.ZERO_SHOT_REACT_DESCRIPTION,
)
agent.run(
    "谁是莱昂纳多·迪卡普里奥的女朋友？她的年龄开 0.43 次方是多少？",
    callbacks=callbacks,
)
wandb_callback.flush_tracker(agent, reset=False, finish=True)
```
```output
> 进入新的 AgentExecutor 链...
我需要找出莱昂纳多·迪卡普里奥的女朋友是谁，然后计算她的年龄开 0.43 次方。
动作：搜索
动作输入："莱昂纳多·迪卡普里奥 女朋友"
观察结果：迪卡普里奥有一个稳定的女朋友卡米拉·莫罗内。他们已经交往了将近五年，因为他们首次被传言于 2017 年底开始约会。现年 26 岁的莫罗内对好莱坞并不陌生。
思考：我需要计算她的年龄开 0.43 次方。
行动：计算器
行动输入：26^0.43
观察：答案：4.059182145592686
思考：我现在知道最终答案了。
最终答案：莱昂纳多·迪卡普里奥的女朋友是卡米拉·莫罗内，她目前的年龄提高到0.43次方等于4.059182145592686。
> 链结束。
```html
等待 W&B 进程完成... <strong style="color:green">(成功)。</strong> 
```
```html

查看运行 <strong style="color:#cdcd00">代理</strong> 在: <a href='https://wandb.ai/harrison-chase/langchain_callback_demo/runs/wzy59zjq' target="_blank">https://wandb.ai/harrison-chase/langchain_callback_demo/runs/wzy59zjq</a><br/>已同步 5 个 W&B 文件，2 个媒体文件，7 个工件文件和 0 个其他文件 

```
```html

在此处查找日志: <code>./wandb/run-20230318_150550-wzy59zjq/logs</code> 

```