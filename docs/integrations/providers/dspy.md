# DSPy
[DSPy](https://github.com/stanfordnlp/dspy) 是一个出色的 LLM 框架，引入了一个自动编译器，教会语言模型如何执行程序中的声明性步骤。具体来说，DSPy 编译器会内部跟踪您的程序，然后为大型语言模型制作高质量提示（或为小型语言模型训练自动微调），以教会它们执行您的任务步骤。
感谢 [Omar Khattab](https://twitter.com/lateinteraction) 的集成！它可以与任何 LCEL 链集成，只需进行一些小的修改。
这个简短的教程演示了这个概念验证功能的工作原理。*这不会给您 DSPy 或 LangChain 的全部功能，但如果有很高的需求，我们将扩展它。*
注意：这是从 Omar 为 DSPy 编写的原始示例稍作修改。如果您对 LangChain \<\> DSPy 感兴趣，但是从 DSPy 方面来的，我建议您查看一下。您可以在 [这里](https://github.com/stanfordnlp/dspy/blob/main/examples/tweets/compiling_langchain.ipynb) 找到它。
让我们来看一个例子。在这个例子中，我们将创建一个简单的 RAG pipeline。我们将使用 DSPy 来“编译”我们的程序并学习一个优化的提示。
## 安装依赖
```python
!pip install -U dspy-ai 
!pip install -U openai jinja2
!pip install -U langchain langchain-community langchain-openai langchain-core
```
## 设置
我们将使用 OpenAI，因此我们应该设置一个 API 密钥
```python
import getpass
import os
os.environ["OPENAI_API_KEY"] = getpass.getpass()
```
现在我们可以设置我们的检索器。对于我们的检索器，我们将通过 DSPy 使用 ColBERT 检索器，尽管这将适用于任何检索器。
```python
import dspy
colbertv2 = dspy.ColBERTv2(url="http://20.102.90.50:2017/wiki17_abstracts")
```
```python
from langchain.globals import set_llm_cache
from langchain_community.cache import SQLiteCache
from langchain-openai import OpenAI
set_llm_cache(SQLiteCache(database_path="cache.db"))
llm = OpenAI(model_name="gpt-3.5-turbo-instruct", temperature=0)
def retrieve(inputs):
    return [doc["text"] for doc in colbertv2(inputs["question"], k=5)]
```
```python
colbertv2("cycling")
```
```output
[{'text': 'Cycling | Cycling, also called bicycling or biking, is the use of bicycles for transport, recreation, exercise or sport. Persons engaged in cycling are referred to as "cyclists", "bikers", or less commonly, as "bicyclists". Apart from two-wheeled bicycles, "cycling" also includes the riding of unicycles, tricycles, quadracycles, recumbent and similar human-powered vehicles (HPVs).', 'pid': 2201868, 'rank': 1, 'score': 27.078739166259766, 'prob': 0.3544841299722533, 'long_text': 'Cycling | Cycling, also called bicycling or biking, is the use of bicycles for transport, recreation, exercise or sport. Persons engaged in cycling are referred to as "cyclists", "bikers", or less commonly, as "bicyclists". Apart from two-wheeled bicycles, "cycling" also includes the riding of unicycles, tricycles, quadracycles, recumbent and similar human-powered vehicles (HPVs).'}, 
{'text': 'Cycling (ice hockey) | In ice hockey, cycling is an offensive strategy that moves the puck along the boards in the offensive zone to create a scoring chance by making defenders tired or moving them out of position.', 'pid': 312153, 'rank': 2, 'score': 26.109302520751953, 'prob': 0.13445464524590262, 'long_text': 'Cycling (ice hockey) | In ice hockey, cycling is an offensive strategy that moves the puck along the boards in the offensive zone to create a scoring chance by making defenders tired or moving them out of position.'}, 
{'text': 'Bicycle | A bicycle, also called a cycle or bike, is a human-powered, pedal-driven, single-track vehicle, having two wheels attached to a frame, one behind the other. A is called a cyclist, or bicyclist.', 'pid': 2197695, 'rank': 3, 'score': 25.849220275878906, 'prob': 0.10366294133944996, 'long_text': 'Bicycle | A bicycle, also called a cycle or bike, is a human-powered, pedal-driven, single-track vehicle, having two wheels attached to a frame, one behind the other. A is called a cyclist, or bicyclist.'}, 
{'text': 'USA Cycling | USA Cycling or USAC, based in Colorado Springs, Colorado, is the national governing body for bicycle racing in the United States. It covers the disciplines of road, track, mountain bike, cyclo-cross, and BMX across all ages and ability levels. In 2015, USAC had a membership of 61,631 individual members.', 'pid': 3821927, 'rank': 4, 'score': 25.61395263671875, 'prob': 0.08193096873942958, 'long_text': 'USA Cycling | USA Cycling or USAC, based in Colorado Springs, Colorado, is the national governing body for bicycle racing in the United States. It covers the disciplines of road, track, mountain bike, cyclo-cross, and BMX across all ages and ability levels. In 2015, USAC had a membership of 61,631 individual members.'}, 
{'text': 'Vehicular cycling | Vehicular cycling (also known as bicycle driving) is the practice of riding bicycles on roads in a manner that is in accordance with the principles for driving in traffic.', 'pid': 3058888, 'rank': 5, 'score': 25.35515785217285, 'prob': 0.06324918635213703, 'long_text': 'Vehicular cycling | Vehicular cycling (also known as bicycle driving) is the practice of riding bicycles on roads in a manner that is in accordance with the principles for driving in traffic.'}, 
{'text': 'Road cycling | Road cycling is the most widespread form of cycling. It includes recreational, racing, and utility cycling. Road cyclists are generally expected to obey the same rules and laws as other vehicle drivers or riders and may also be vehicular cyclists.', 'pid': 3392359, 'rank': 6, 'score': 25.274639129638672, 'prob': 0.058356079351563846, 'long_text': 'Road cycling | Road cycling is the most widespread form of cycling. It includes recreational, racing, and utility cycling. Road cyclists are generally expected to obey the same rules and laws as other vehicle drivers or riders and may also be vehicular cyclists.'}, 
{'text': 'Cycling South Africa | Cycling South Africa or Cycling SA is the national governing body of cycle racing in South Africa. Cycling SA is a member of the "Confédération Africaine de Cyclisme" and the "Union Cycliste Internationale" (UCI). It is affiliated to the South African Sports Confederation and Olympic Committee (SASCOC) as well as the Department of Sport and Recreation SA. Cycling South Africa regulates the five major disciplines within the sport, both amateur and professional, which include: road cycling, mountain biking, BMX biking, track cycling and para-cycling.', 'pid': 2508026, 'rank': 7, 'score': 25.24260711669922, 'prob': 0.05651643767006817, 'long_text': 'Cycling South Africa | Cycling South Africa or Cycling SA is the national governing body of cycle racing in South Africa. Cycling SA is a member of the "Confédération Africaine de Cyclisme" and the "Union Cycliste Internationale" (UCI). It is affiliated to the South African Sports Confederation and Olympic Committee (SASCOC) as well as the Department of Sport and Recreation SA. Cycling South Africa regulates the five major disciplines within the sport, both amateur and professional, which include: road cycling, mountain biking, BMX biking, track cycling and para-cycling.'}, 
{'text': 'Cycle sport | Cycle sport is competitive physical activity using bicycles. There are several categories of bicycle racing including road bicycle racing, time trialling, cyclo-cross, mountain bike racing, track cycling, BMX, and cycle speedway. Non-racing cycling sports include artistic cycling, cycle polo, freestyle BMX and mountain bike trials. The Union Cycliste Internationale (UCI) is the world governing body for cycling and international competitive cycling events. The International Human Powered Vehicle Association is the governing body for human-powered vehicles that imposes far fewer restrictions on their design than does the UCI. The UltraMarathon Cycling Association is the governing body for many ultra-distance cycling races.', 'pid': 3394121, 'rank': 8, 'score': 25.170495986938477, 'prob': 0.05258444735141742, 'long_text': 'Cycle sport | Cycle sport is competitive physical activity using bicycles. There are several categories of bicycle racing including road bicycle racing, time trialling, cyclo-cross, mountain bike racing, track cycling, BMX, and cycle speedway. Non-racing cycling sports include artistic cycling, cycle polo, freestyle BMX and mountain bike trials. The Union Cycliste Internationale (UCI) is the world governing body for cycling and international competitive cycling events. The International Human Powered Vehicle Association is the governing body for human-powered vehicles that imposes far fewer restrictions on their design than does the UCI. The UltraMarathon Cycling Association is the governing body for many ultra-distance cycling races.'}, 
{'text': "Cycling UK | Cycling UK is the brand name of the Cyclists' Touring Club or CTC. It is a charitable membership organisation supporting cyclists and promoting bicycle use. Cycling UK is registered at Companies House (as “Cyclists’ Touring Club”), and covered by company law; it is the largest such organisation in the UK. It works at a national and local level to lobby for cyclists' needs and wants, provides services to members, and organises local groups for local activism and those interested in recreational cycling. The original Cyclists' Touring Club began in the nineteenth century with a focus on amateur road cycling but these days has a much broader sphere of interest encompassing everyday transport, commuting and many forms of recreational cycling. Prior to April 2016, Cycling UK operated under the brand CTC, the national cycling charity. As of January 2007, the organisation's president was the newsreader Jon Snow.", 'pid': 1841483, 'rank': 9, 'score': 25.166988372802734, 'prob': 0.05240032450529368, 'long_text': "Cycling UK | Cycling UK is the brand name of the Cyclists' Touring Club or CTC. It is a charitable membership organisation supporting cyclists and promoting bicycle use. Cycling UK is registered at Companies House (as “Cyclists’ Touring Club”), and covered by company law; it is the largest such organisation in the UK. It works at a national and local level to lobby for cyclists' needs and wants, provides services to members, and organises local groups for local activism and those interested in recreational cycling. The original Cyclists' Touring Club began in the nineteenth century with a focus on amateur road cycling but these days has a much broader sphere of interest encompassing everyday transport, commuting and many forms of recreational cycling. Prior to April 2016, Cycling UK operated under the brand CTC, the national cycling charity. As of January 2007, the organisation's president was the newsreader Jon Snow."}, 
{'text': 'Cycling in the Netherlands | Cycling is a ubiquitous mode of transport in the Netherlands, with 36% of the people listing the bicycle as their most frequent mode of transport on a typical day as opposed to the car by 45% and public transport by 11%. Cycling has a modal share of 27% of all trips (urban and rural) nationwide. In cities this is even higher, such as Amsterdam which has 38%, though the smaller Dutch cities well exceed that: for instance Zwolle (pop. ~123,000) has 46% and the university town of Groningen (pop. ~198,000) has 31%. This high modal share for bicycle travel is enabled by excellent cycling infrastructure such as cycle paths, cycle tracks, protected intersections, ubiquitous bicycle parking and by making cycling routes shorter, quicker and more direct than car routes.', 'pid': 1196118, 'rank': 10, 'score': 24.954299926757812, 'prob': 0.0423608394724844, 'long_text': 'Cycling in the Netherlands | Cycling is a ubiquitous mode of transport in the Netherlands, with 36% of the people listing the bicycle as their most frequent mode of transport on a typical day as opposed to the car by 45% and public transport by 11%. Cycling has a modal share of 27% of all trips (urban and rural) nationwide. In cities this is even higher, such as Amsterdam which has 38%, though the smaller Dutch cities well exceed that: for instance Zwolle (pop. ~123,000) has 46% and the university town of Groningen (pop. ~198,000) has 31%. This high modal share for bicycle travel is enabled by excellent cycling infrastructure such as cycle paths, cycle tracks, protected intersections, ubiquitous bicycle parking and by making cycling routes shorter, quicker and more direct than car routes.'}]
```
## 普通 LCEL
首先，让我们像平常一样使用 LCEL 创建一个简单的 RAG pipeline。
为了说明问题，让我们来解决以下任务。
**任务：** 构建一个 RAG 系统，用于生成信息丰富的推文。
- **输入：** 一个事实性问题，可能相当复杂。
- **输出：** 一个引人入胜的推文，正确回答从检索到的信息中得出的问题。
让我们使用 LangChain 的表达语言（LCEL）来说明这一点。这里的任何提示都可以，我们将使用 DSPy 优化最终提示。
考虑到这一点，让我们只保留基本要素：**给定{context}，以推文形式回答问题{question}。**
```python
# 从 LangChain 导入用于提示的标准模块。
from langchain_core.output_parsers import StrOutputParser
from langchain_core.prompts import PromptTemplate
from langchain_core.runnables import RunnablePassthrough
# 为这个任务创建一个简单的提示。如果复杂也没关系。
prompt = PromptTemplate.from_template(
    "给定{context}，以推文形式回答问题`{question}`。"
)
# 这是您通常使用 LCEL 构建链的方式。该链首先进行检索，然后生成（RAG）。
vanilla_chain = (
    RunnablePassthrough.assign(context=retrieve) | prompt | llm | StrOutputParser()
)
```
## LCEL \<\> DSPy
为了将 LangChain 与 DSPy 一起使用，您需要进行两个微小的修改
**LangChainPredict**
您需要从执行 `prompt | llm` 更改为使用 `dspy` 中的 `LangChainPredict(prompt, llm)`。
这是一个包装器，将您的提示和 llm 绑定在一起，以便您可以对它们进行优化
**LangChainModule**
这是一个包装器，用于包装您的最终 LCEL 链，以便 DSPy 可以优化整个链
```python
# 从 DSPy 导入知道如何与 LangChain LCEL 交互的模块。
from dspy.predict.langchain import LangChainModule, LangChainPredict
# 这是将其包装为类似 DSPy 程序的方法。
# 只需将所有类似 `prompt | llm` 的模式替换为 `LangChainPredict(prompt, llm)`。
zeroshot_chain = (
    RunnablePassthrough.assign(context=retrieve)
    | LangChainPredict(prompt, llm)
    | StrOutputParser()
)
# 现在我们将其包装在 LangChainModule 中
zeroshot_chain = LangChainModule(
    zeroshot_chain
)  # 然后将链包装在一个 DSPy 模块中。
```
## 尝试该模块
之后，我们可以将其用作 LangChain 可运行程序和 DSPy 模块！
```python
question = "Eddy Mazzoleni 出生在哪个地区？"
zeroshot_chain.invoke({"question": question})
```
```output
' Eddy Mazzoleni 出生在意大利贝加莫，是一名为 UCI ProTour Astana Team 骑行的专业公路自行车手。#自行车手 #意大利'
```
嗯，听起来差不多对了！（严格来说并不完美：我们询问的是地区，而不是城市。我们可以在下面做得更好。）
手动检查问题和答案对于了解您的系统非常重要。然而，一个优秀的系统设计师总是试图通过迭代基准测试他们的工作，以量化进展！
为此，我们需要两样东西：我们想要最大化的度量标准和我们系统的（小型）示例数据集。
是否有用于好推文的预定义度量标准？我应该手动标记 100,000 条推文吗？可能不需要。不过，在开始生产数据之前，我们可以轻松做一些合理的事情！
## 加载数据
为了编译我们的链，我们需要一个数据集。这个数据集只需要是原始输入和输出。为了我们的目的，我们将使用 HotPotQA 数据集。
注意：请注意，我们的数据集实际上并不包含任何推文！它只有问题和答案。没关系，我们的度量标准将负责评估推文形式的输出。
```python
import dspy
from dspy.datasets import HotPotQA
# 加载数据集。
dataset = HotPotQA(
    train_seed=1,
    train_size=200,
    eval_seed=2023,
    dev_size=200,
    test_size=0,
    keep_details=True,
)
# 告诉 DSPy，“question” 字段是输入。其他字段是标签和/或元数据。
trainset = [x.without("id", "type").with_inputs("question") for x in dataset.train]
devset = [x.without("id", "type").with_inputs("question") for x in dataset.dev]
valset, devset = devset[:50], devset[50:]
```
```output
/Users/harrisonchase/.pyenv/versions/3.11.1/envs/langchain-3-11/lib/python3.11/site-packages/datasets/table.py:1421: FutureWarning: promote has been superseded by mode='default'.
  table = cls._concat_blocks(blocks, axis=0)
```
## 定义度量标准
现在，我们需要定义一个度量标准。这将用于确定哪些运行是成功的，以及我们可以从中学习。在这里，我们将使用 DSPy 的度量标准，不过您也可以编写自己的。
```python
# 定义用于自动评估的签名。
class Assess(dspy.Signature):
    """评估推文质量的指定维度。"""
    context = dspy.InputField(desc="如果不适用，请忽略")
    assessed_text = dspy.InputField()
    assessment_question = dspy.InputField()```
```python
评估策略 = 无
```
## 评估基线
好的，让我们评估我们的链条的未优化的“零-shot”版本，该版本是从我们的 LangChain LCEL 对象转换而来。
```python
from dspy.evaluate.evaluate import Evaluate
```
```python
evaluate = Evaluate(
    metric=metric, devset=devset, num_threads=8, display_progress=True, display_table=5
)
evaluate(zeroshot_chain)
```
```output
平均指标: 62.99999999999998 / 150  (42.0%): 100%|██| 150/150 [01:14<00:00,  2.02it/s]
```
```html
<style type="text/css">
#T_390d8 th {
  text-align: left;
}
#T_390d8 td {
  text-align: left;
}
#T_390d8_row0_col0, #T_390d8_row0_col1, #T_390d8_row0_col2, #T_390d8_row0_col3, #T_390d8_row0_col4, #T_390d8_row0_col5, #T_390d8_row1_col0, #T_390d8_row1_col1, #T_390d8_row1_col2, #T_390d8_row1_col3, #T_390d8_row1_col4, #T_390d8_row1_col5, #T_390d8_row2_col0, #T_390d8_row2_col1, #T_390d8_row2_col2, #T_390d8_row2_col3, #T_390d8_row2_col4, #T_390d8_row2_col5, #T_390d8_row3_col0, #T_390d8_row3_col1, #T_390d8_row3_col2, #T_390d8_row3_col3, #T_390d8_row3_col4, #T_390d8_row3_col5, #T_390d8_row4_col0, #T_390d8_row4_col1, #T_390d8_row4_col2, #T_390d8_row4_col3, #T_390d8_row4_col4, #T_390d8_row4_col5 {
  text-align: left;
  white-space: pre-wrap;
  word-wrap: break-word;
  max-width: 400px;
}
</style>
<table id="T_390d8">
  <thead>
    <tr>
      <th class="blank level0" >&nbsp;</th>
      <th id="T_390d8_level0_col0" class="col_heading level0 col0" >问题</th>
      <th id="T_390d8_level0_col1" class="col_heading level0 col1" >答案</th>
      <th id="T_390d8_level0_col2" class="col_heading level0 col2" >金标题目</th>
      <th id="T_390d8_level0_col3" class="col_heading level0 col3" >输出</th>
      <th id="T_390d8_level0_col4" class="col_heading level0 col4" >推文回复</th>
      <th id="T_390d8_level0_col5" class="col_heading level0 col5" >指标</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th id="T_390d8_level0_row0" class="row_heading level0 row0" >0</th>
      <td id="T_390d8_row0_col0" class="data row0 col0" >谁是制作人，为摇滚乐队 Juke Karten 和 Thirty Seconds to Mars 制作专辑？</td>
      <td id="T_390d8_row0_col1" class="data row0 col1" >Brian Virtue</td>
      <td id="T_390d8_row0_col2" class="data row0 col2" >{'Thirty Seconds to Mars', 'Levolution (album)'}</td>
      <td id="T_390d8_row0_col3" class="data row0 col3" >Brian Virtue, who has worked with bands like Jane's Addiction and Velvet Revolver, produced albums for both Juke Kartel and Thirty Seconds to Mars. #BrianVirtue...</td>
      <td id="T_390d8_row0_col4" class="data row0 col4" >Brian Virtue, who has worked with bands like Jane's Addiction and Velvet Revolver, produced albums for both Juke Kartel and Thirty Seconds to Mars. #BrianVirtue...</td>
      <td id="T_390d8_row0_col5" class="data row0 col5" >1.0</td>
    </tr>
    <tr>
      <th id="T_390d8_level0_row1" class="row_heading level0 row1" >1</th>
```
| 问题 | 回答 | 实体 | 简介 | 摘要 | 得分 |
| ---- | ---- | ---- | ---- | ---- | ---- |
| 1 | 两所大学都是公立大学吗？ | 否 | {'Syracuse University', 'University of Chicago'} | 不是，只有雪城大学是公立大学。芝加哥大学是一所私立研究型大学。#大学#公立vs私立 | 不是，只有雪城大学是公立大学。芝加哥大学是一所私立研究型大学。#大学#公立vs私立 | 0.3333333333333333 |
| 2 | Eddy Mazzoleni 出生在哪个地区？ | 意大利北部的伦巴第 | {'Eddy Mazzoleni', 'Bergamo'} | Eddy Mazzoleni 出生在意大利的贝加莫，是一名为国际自行车联盟职业车队阿斯塔纳车队骑行的专业公路自行车手。#自行车手#意大利 | Eddy Mazzoleni 出生在意大利的贝加莫，是一名为国际自行车联盟职业车队阿斯塔纳车队骑行的专业公路自行车手。#自行车手#意大利 | 0.0 |
| 3 | 谁编辑了加里·马歇尔执导的 1990 年美国浪漫喜剧电影？ | Raja Raymond Gosnell | {'Raja Gosnell', 'Pretty Woman'} | J. F. Lawton 编写了加里·马歇尔执导的 1990 年美国浪漫喜剧电影《风月俏佳人》的剧本。#风月俏佳人#加里马歇尔#JFLawton | J. F. Lawton 编写了加里·马歇尔执导的 1990 年美国浪漫喜剧电影《风月俏佳人》的剧本。#风月俏佳人#加里马歇尔#JFLawton | 0.0 |
| 4 | Burrs Country Park 火车站是 Heywood 和 Rawtenstall 之间铁路线上的第几站？ | 第七站 | {'Burrs Country Park railway station', 'East Lancashire Railway'} | Burrs Country Park 火车站是 Heywood 和 Rawtenstall 之间东兰开夏郡铁路线上的第七站。 | Burrs Country Park 火车站是 Heywood 和 Rawtenstall 之间东兰开夏郡铁路线上的第七站。 | 1.0 |
...
42.0
```
好的，很棒。我们的 zeroshot_chain 在 devset 的 150 个问题中获得了约 42.00% 的准确率。
上面的表格显示了一些例子。例如：
- 问题：谁是既为摇滚乐队 Juke Karten 又为 Thirty Seconds to Mars 制作专辑的制作人？
- 推文：Brian Virtue 曾与 Jane's Addiction 和 Velvet Revolver 等乐队合作，为 Juke Kartel 和 Thirty Seconds to Mars 制作了专辑，展示了……[被截断]
- 评分：1.0（一个正确、忠实且引人入胜的推文！*）
注脚：* 至少根据我们的度量标准，这只是一个 DSPy 程序，所以如果您愿意，它也可以进行优化！不过这是另一个笔记本的话题。
## 优化
现在，让我们优化性能
```python
from dspy.teleprompt import BootstrapFewShotWithRandomSearch
```
```python
# 设置优化器。对于这个示例，我们将使用非常少的超参数。
# 只需进行随机搜索，尝试次数约为 3 次，在每次尝试中，引导 <= 3 条轨迹。
optimizer = BootstrapFewShotWithRandomSearch(
    metric=metric, max_bootstrapped_demos=3, num_candidate_programs=3
)
# 现在使用优化器来 *编译* 链。这可能需要 5-10 分钟，除非它已被缓存。
```
```output
将 optimized_chain 编译为 optimizer.compile(zeroshot_chain, trainset=trainset, valset=valset)。
```
```output
将在每个预测器中采样 1 到 3 条轨迹。
将尝试训练 3 个候选集。
```
```output
平均指标：22.33333333333334 / 50（44.7）：100%|█████| 50/50 [00:26<00:00, 1.87it/s]
/Users/harrisonchase/.pyenv/versions/3.11.1/envs/langchain-3-11/lib/python3.11/site-packages/dspy/evaluate/evaluate.py:126: FutureWarning: DataFrame.applymap 已被弃用。请改用 DataFrame.map。
  df = df.applymap(truncate_cell)
```
```output
平均指标：22.33333333333334 / 50（44.7%）
得分：44.67，集合：[0]
新的最佳得分：44.67，种子为 -3
迄今为止的得分：[44.67]
最佳得分：44.67
```
```output
平均指标：22.33333333333334 / 50（44.7）：100%|█████| 50/50 [00:00<00:00, 79.51it/s]
/Users/harrisonchase/.pyenv/versions/3.11.1/envs/langchain-3-11/lib/python3.11/site-packages/dspy/evaluate/evaluate.py:126: FutureWarning: DataFrame.applymap 已被弃用。请改用 DataFrame.map。
  df = df.applymap(truncate_cell)
```
```output
平均指标：22.33333333333334 / 50（44.7%）
得分：44.67，集合：[16]
迄今为止的得分：[44.67, 44.67]
最佳得分：44.67
```
```output
  4%|██                                                   | 8/200 [00:33<13:21,  4.18s/it]
```
```output
在第 0 轮的 9 个示例后，对 3 条完整轨迹进行了自举抽样。
```
```output
平均指标：24.666666666666668 / 50（49.3）：100%|████| 50/50 [00:28<00:00, 1.77it/s]
/Users/harrisonchase/.pyenv/versions/3.11.1/envs/langchain-3-11/lib/python3.11/site-packages/dspy/evaluate/evaluate.py:126: FutureWarning: DataFrame.applymap 已被弃用。请改用 DataFrame.map。
  df = df.applymap(truncate_cell)
```
```output
平均指标：24.666666666666668 / 50（49.3%）
得分：49.33，集合：[16]
新的最佳得分：49.33，种子为 -1
迄今为止的得分：[44.67, 44.67, 49.33]
最佳得分：49.33
在前 1 个得分中每个条目的最大平均值：0.49333333333333335
在前 2 个得分中每个条目的最大平均值：0.5533333333333335
在前 3 个得分中每个条目的最大平均值：0.5533333333333335
在前 5 个得分中每个条目的最大平均值：0.5533333333333335
在前 8 个得分中每个条目的最大平均值：0.5533333333333335
在前 9999 个得分中每个条目的最大平均值：0.5533333333333335
```
```output
  6%|███                                                 | 12/200 [00:31<08:16,  2.64s/it]
```
```output
在第 0 轮的 13 个示例后，对 2 条完整轨迹进行了自举抽样。
```
```output
平均指标：25.66666666666667 / 50（51.3）：100%|█████| 50/50 [00:25<00:00, 1.92it/s]
/Users/harrisonchase/.pyenv/versions/3.11.1/envs/langchain-3-11/lib/python3.11/site-packages/dspy/evaluate/evaluate.py:126: FutureWarning: DataFrame.applymap 已被弃用。请改用 DataFrame.map。
  df = df.applymap(truncate_cell)
```
```output
平均指标：25.66666666666667 / 50（51.3%）
得分：51.33，集合：[16]
新的最佳得分：51.33，种子为 0
迄今为止的得分：[44.67, 44.67, 49.33, 51.33]
最佳得分：51.33
在前 1 个得分中每个条目的最大平均值：0.5133333333333334
在前 2 个得分中每个条目的最大平均值：0.5666666666666668
在前 3 个得分中每个条目的最大平均值：0.6000000000000001
在前 5 个得分中每个条目的最大平均值：0.6000000000000001
在前 8 个得分中每个条目的最大平均值：0.6000000000000001
在前 9999 个得分中每个条目的最大平均值：0.6000000000000001
```
```output
  0%|▎                                                    | 1/200 [00:02<08:37,  2.60s/it]
```
```output
在第 0 轮的 2 个示例后，对 1 条完整轨迹进行了自举抽样。
```
```output
平均指标：26.33333333333334 / 50（52.7）：100%|█████| 50/50 [00:23<00:00, 2.11it/s]
/Users/harrisonchase/.pyenv/versions/3.11.1/envs/langchain-3-11/lib/python3.11/site-packages/dspy/evaluate/evaluate.py:126: FutureWarning: DataFrame.applymap 已被弃用。请改用 DataFrame.map。
  df = df.applymap(truncate_cell)
```
```output
平均指标：26.33333333333334 / 50（52.7%）
得分：52.67，集合：[16]
新的最佳得分：52.67，种子为 1
迄今为止的得分：[44.67, 44.67, 49.33, 51.33, 52.67]
最佳得分：52.67
在前 1 个得分中每个条目的最大平均值：0.5266666666666667
在前 2 个得分中每个条目的最大平均值：0.56
在前 3 个得分中每个条目的最大平均值：0.5666666666666668
在前 5 个得分中每个条目的最大平均值：0.6000000000000001
在前 8 个得分中每个条目的最大平均值：0.6000000000000001
在前 9999 个得分中每个条目的最大平均值：0.6000000000000001
```
```output
  0%|▎                                                    | 1/200 [00:02<07:11,  2.17s/it]
```
```output
在第 0 轮的 2 个示例后，对 1 条完整轨迹进行了自举抽样。
```
```output
平均指标：25.666666666666668 / 50（51.3）：100%|████| 50/50 [00:21<00:00, 2.29it/s]
```
```output
平均指标：25.666666666666668 / 50（51.3%）
得分：51.33，集合：[16]
迄今为止的得分：[44.67, 44.67, 49.33, 51.33, 52.67, 51.33]
最佳得分：52.67
```
## 评估优化链
平均每个条目在前1个分数中的最大值：0.5266666666666667
平均每个条目在前2个分数中的最大值：0.56
平均每个条目在前3个分数中的最大值：0.6000000000000001
平均每个条目在前5个分数中的最大值：0.6133333333333334
平均每个条目在前8个分数中的最大值：0.6133333333333334
平均每个条目在前9999个分数中的最大值：0.6133333333333334
找到6个候选程序。
```python
/Users/harrisonchase/.pyenv/versions/3.11.1/envs/langchain-3-11/lib/python3.11/site-packages/dspy/evaluate/evaluate.py:126: FutureWarning: DataFrame.applymap has been deprecated. Use DataFrame.map instead.
  df = df.applymap(truncate_cell)
```
好了，这个优化链有多好呢？让我们进行一些适当的评估！
```python
evaluate(optimized_chain)
```
```output
平均指标：74.66666666666666 / 150  (49.8)：100%|██| 150/150 [00:54<00:00,  2.74it/s]
```
```output
平均指标：74.66666666666666 / 150  (49.8%)
```
```html
<style type="text/css">
#T_b4366 th {
  text-align: left;
}
#T_b4366 td {
  text-align: left;
}
#T_b4366_row0_col0, #T_b4366_row0_col1, #T_b4366_row0_col2, #T_b4366_row0_col3, #T_b4366_row0_col4, #T_b4366_row0_col5, #T_b4366_row1_col0, #T_b4366_row1_col1, #T_b4366_row1_col2, #T_b4366_row1_col3, #T_b4366_row1_col4, #T_b4366_row1_col5, #T_b4366_row2_col0, #T_b4366_row2_col1, #T_b4366_row2_col2, #T_b4366_row2_col3, #T_b4366_row2_col4, #T_b4366_row2_col5, #T_b4366_row3_col0, #T_b4366_row3_col1, #T_b4366_row3_col2, #T_b4366_row3_col3, #T_b4366_row3_col4, #T_b4366_row3_col5, #T_b4366_row4_col0, #T_b4366_row4_col1, #T_b4366_row4_col2, #T_b4366_row4_col3, #T_b4366_row4_col4, #T_b4366_row4_col5 {
  text-align: left;
  white-space: pre-wrap;
  word-wrap: break-word;
  max-width: 400px;
}
</style>
<table id="T_b4366">
  <thead>
    <tr>
      <th class="blank level0" >&nbsp;</th>
      <th id="T_b4366_level0_col0" class="col_heading level0 col0" >问题</th>
      <th id="T_b4366_level0_col1" class="col_heading level0 col1" >答案</th>
      <th id="T_b4366_level0_col2" class="col_heading level0 col2" >黄金标题</th>
      <th id="T_b4366_level0_col3" class="col_heading level0 col3" >输出</th>
      <th id="T_b4366_level0_col4" class="col_heading level0 col4" >推文回复</th>
      <th id="T_b4366_level0_col5" class="col_heading level0 col5" >指标</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th id="T_b4366_level0_row0" class="row_heading level0 row0" >0</th>
      <td id="T_b4366_row0_col0" class="data row0 col0" >谁是制作人，为摇滚乐队Juke Karten和Thirty Seconds to Mars制作专辑？</td>
      <td id="T_b4366_row0_col1" class="data row0 col1" >布赖恩·维尔蒂</td>
      <td id="T_b4366_row0_col2" class="data row0 col2" >{'Thirty Seconds to Mars', 'Levolution (album)'}</td>
      <td id="T_b4366_row0_col3" class="data row0 col3" >布赖恩·维尔蒂以与简·艾达琼和天堂乐队合作而闻名，为Juke Kartel和Thirty Seconds to Mars制作了专辑。#BrianVirtue #Producer...</td>
      <td id="T_b4366_row0_col4" class="data row0 col4" >布赖恩·维尔蒂以与简·艾达琼和天堂乐队合作而闻名，为Juke Kartel和Thirty Seconds to Mars制作了专辑。#BrianVirtue #Producer...</td>
      <td id="T_b4366_row0_col5" class="data row0 col5" >1.0</td>
    </tr>
    <tr>
      <th id="T_b4366_level0_row1" class="row_heading level0 row1" >1</th>
      <td id="T_b4366_row1_col0" class="data row1 col0" >芝加哥大学和锡拉丘兹大学都是公立大学吗？</td>
      <td id="T_b4366_row1_col1" class="data row1 col1" >不是</td>
      <td id="T_b4366_row1_col2" class="data row1 col2" >{'Syracuse University', 'University of Chicago'}</td>
      <td id="T_b4366_row1_col3" class="data row1 col3" >不是，只有伊利诺伊州立大学是一所公立州立大学。锡拉丘兹大学是一所私立研究型大学。#University #PublicPrivate #HigherEd</td>
      <td id="T_b4366_row1_col4" class="data row1 col4" >不是，只有伊利诺伊州立大学是一所公立州立大学。锡拉丘兹大学是一所私立研究型大学。#University #PublicPrivate #HigherEd</td>
      <td id="T_b4366_row1_col5" class="data row1 col5" >0.0</td>
    </tr>
    <tr>
      <th id="T_b4366_level0_row2" class="row_heading level0 row2" >2</th>
      <td id="T_b4366_row2_col0" class="data row2 col0" >Eddy Mazzoleni出生在哪个地区？</td>
      <td id="T_b4366_row2_col1" class="data row2 col1" >意大利北部的伦巴第大区</td>
      <td id="T_b4366_row2_col2" class="data row2 col2" >{'Eddy Mazzoleni', 'Bergamo'}</td>
## 翻译结果
<table>
<thead>
<tr>
<th>序号</th>
<th>问题</th>
<th>回答</th>
<th>上下文</th>
<th>得分</th>
</tr>
</thead>
<tbody>
<tr>
<th>1</th>
<td>意大利职业公路自行车手Eddy Mazzoleni出生在意大利的贝加莫。#EddyMazzoleni #Cycling #Italy</td>
<td>意大利职业公路自行车手Eddy Mazzoleni出生在意大利的贝加莫。#EddyMazzoleni #Cycling #Italy</td>
<td>0.0</td>
</tr>
<tr>
<th>3</th>
<td>谁编辑了由Garry Marshall执导的1990年美国浪漫喜剧电影？</td>
<td>Raja Raymond Gosnell</td>
<td>J. F. Lawton为1990年由Garry Marshall执导的浪漫喜剧电影《漂亮女人》编写了剧本。#PrettyWoman #GarryMarshall #RomanticComedy</td>
<td>0.0</td>
</tr>
<tr>
<th>4</th>
<td>Burrs Country Park火车站是连接Heywood和Rawtenstall之间的铁路线上的第几站？</td>
<td>第七站</td>
<td>Burrs Country Park火车站是连接Heywood和Rawtenstall之间的East Lancashire Railway上的第七站。#EastLancashireRailway #BurrsCountryPark #RailwayStation</td>
<td>1.0</td>
</tr>
</tbody>
</table>
我们的链条从42%提高到了近50%！这是怎么发生的呢？我们可以通过查看优化后的链条来了解。
## 检查优化后的链条
我们可以通过两种方式来查看优化后的链条。
### 查看使用的提示
我们可以查看实际使用的提示。我们可以通过查看`dspy.settings`来做到这一点。
```python
prompt_used, output = dspy.settings.langchain_history[-1]
```
```python
print(prompt_used)
```
```output
基本说明：根据给定的上下文，以推文的形式回答提供的问题，确保回答简洁，符合推特的字符限制（最多280个字符）。
---
按照以下格式进行回答。
上下文：${context}
问题：${question}
推文回答：${tweet_response}
---
上下文：
[1] “Brutus（Funny Car）| Brutus是20世纪60年代中期由吉姆·利伯曼（Jim Liberman）驾驶和由机组长Lew Arrington准备的开创性的Funny Car。”
[2] “USS Brutus（AC-15）| USS“Brutus”，原名为“Peter Jebsen”号，是美国海军的一艘煤船。她于1894年在英格兰泰恩河畔南盾（South Shields-on-Tyne）由约翰·里德黑德和儿子（John Readhead & Sons）建造，并于1898年初从L. F. Chapman & Company手中被美国海军收购。她改名为“Brutus”，并于1898年5月27日在Mare Island海军造船厂服役，指挥官为Vincendon L. Cottman中尉，执行官为Randolph H. Miner中尉。”
[3] “Brutus Beefcake | Ed Leslie是一位美国半退休职业摔跤手，以Brutus“The Barber”Beefcake的名字在世界摔角联盟（WWF）中最为著名。他后来在世界冠军摔角（WCW）中使用了多个名字。”
[4] “Brutus Hamilton | Brutus Kerr Hamilton（1900年7月19日-1970年12月28日）是一位美国田径运动员、教练和田径管理人员。”
[5] “Big Brutus | Big Brutus是Bucyrus-Erie 1850B型电动铲车的昵称，它是20世纪60年代和70年代运营的同类设备中第二大的。Big Brutus是堪萨斯州西米纳尔（West Mineral）的一个采煤博物馆的核心，它曾用于煤炭露天采矿作业。这台铲车设计用于在相对浅的煤层中挖掘20到英尺。”
问题：这位美国赛车手的绰号是什么？
```
### 探索演示
这种优化的方式是我们收集了一些例子（或“演示”）放入提示中。我们可以检查 optimized_chain 来了解这些例子是什么。
```python
demos = [
    eg
    for eg in optimized_chain.modules[0].demos
    if hasattr(eg, "augmented") and eg.augmented
]
```
```python
demos
```
```output
[Example({'augmented': True, 'question': 'What is the nickname for this United States drag racer who drove Brutus?', 'context': ['Brutus (Funny Car) | Brutus is a pioneering funny car driven by Jim Liberman and prepared by crew chief Lew Arrington in the middle 1960s.', 'USS Brutus (AC-15) | USS "Brutus", formerly the steamer "Peter Jebsen", was a collier in the United States Navy. She was built in 1894 at South Shields-on-Tyne, England, by John Readhead & Sons and was acquired by the U.S. Navy early in 1898 from L. F. Chapman & Company. She was renamed "Brutus" and commissioned at the Mare Island Navy Yard on 27 May 1898, with Lieutenant Vincendon L. Cottman, commanding officer and Lieutenant Randolph H. Miner, executive officer.', 'Brutus Beefcake | Ed Leslie is an American semi-retired professional wrestler, best known for his work in the World Wrestling Federation (WWF) under the ring name Brutus "The Barber" Beefcake. He later worked for World Championship Wrestling (WCW) under a variety of names.', 'Brutus Hamilton | Brutus Kerr Hamilton (July 19, 1900 – December 28, 1970) was an American track and field athlete, coach and athletics administrator.', 'Big Brutus | Big Brutus is the nickname of the Bucyrus-Erie model 1850B electric shovel, which was the second largest of its type in operation in the 1960s and 1970s. Big Brutus is the centerpiece of a mining museum in West Mineral, Kansas where it was used in coal strip mining operations. The shovel was designed to dig from 20 to in relatively shallow coal seams.'], 'tweet_response': ' Jim Liberman, also known as "Jungle Jim", drove the pioneering funny car Brutus in the 1960s. #Brutus #FunnyCar #DragRacing'}) (input_keys=None)]
```