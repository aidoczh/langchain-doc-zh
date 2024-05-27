# Bearly 代码解释器

Bearly 代码解释器允许远程执行代码。这使得它非常适合作为代理的代码沙盒，以便安全地实现诸如代码解释器之类的功能。

在这个笔记本中，我们将创建一个使用 Bearly 与数据交互的代理的示例。

```python
from langchain_community.tools import BearlyInterpreterTool
```

```python
from langchain.agents import AgentType, initialize_agent
from langchain_openai import ChatOpenAI
```

初始化解释器

```python
bearly_tool = BearlyInterpreterTool(api_key="...")
```

让我们向沙盒中添加一些文件

```python
bearly_tool.add_file(
    source_path="sample_data/Bristol.pdf", target_path="Bristol.pdf", description=""
)
bearly_tool.add_file(
    source_path="sample_data/US_GDP.csv", target_path="US_GDP.csv", description=""
)
```

现在创建一个 `Tool` 对象。这是必要的，因为我们添加了文件，我们希望工具描述反映这一点

```python
tools = [bearly_tool.as_tool()]
```

```python
tools[0].name
```

```output
'bearly_interpreter'
```

```python
print(tools[0].description)
```

```output
在沙盒环境中评估 Python 代码。每次执行时环境都会重置。您必须每次发送整个脚本并打印您的输出。脚本应该是可以评估的纯 Python 代码。它应该是 Python 格式而不是 markdown。代码不应该用反引号括起来。所有 Python 包，包括 requests、matplotlib、scipy、numpy、pandas 等都可用。如果您有任何输出文件，请将它们写入相对于执行路径的 "output/"。输出只能从目录、标准输出和标准输入中读取。不要使用像 plot.show() 这样的东西，因为它不起作用，而是将它们写出到 `output/`，并返回文件的链接。打印任何输出和结果，以便您可以捕获输出。
在评估环境中可用的以下文件：
- 路径：`Bristol.pdf` 
 前四行：[] 
 描述：``
- 路径：`US_GDP.csv` 
 前四行：['DATE,GDP\n', '1947-01-01,243.164\n', '1947-04-01,245.968\n', '1947-07-01,249.585\n'] 
 描述：``
```

初始化一个代理

```python
llm = ChatOpenAI(model="gpt-4", temperature=0)
agent = initialize_agent(
    tools,
    llm,
    agent=AgentType.OPENAI_FUNCTIONS,
    verbose=True,
    handle_parsing_errors=True,
)
```

```python
# 提取 pdf 内容
agent.run("What is the text on page 3 of the pdf?")
```

```output
> 进入新的 AgentExecutor 链...
调用：`bearly_interpreter` with `{'python_code': "import PyPDF2\n\n# Open the PDF file in read-binary mode\npdf_file = open('Bristol.pdf', 'rb')\n\n# Create a PDF file reader object\npdf_reader = PyPDF2.PdfFileReader(pdf_file)\n\n# Get the text from page 3\npage_obj = pdf_reader.getPage(2)\npage_text = page_obj.extractText()\n\n# Close the PDF file\npdf_file.close()\n\nprint(page_text)"}`
{'stdout': '', 'stderr': 'Traceback (most recent call last):\n  File "/tmp/project/main.py", line 7, in <module>\n    pdf_reader = PyPDF2.PdfFileReader(pdf_file)\n  File "/venv/lib/python3.10/site-packages/PyPDF2/_reader.py", line 1974, in __init__\n    deprecation_with_replacement("PdfFileReader", "PdfReader", "3.0.0")\n  File "/venv/lib/python3.10/site-packages/PyPDF2/_utils.py", line 369, in deprecation_with_replacement\n    deprecation(DEPR_MSG_HAPPENED.format(old_name, removed_in, new_name))\n  File "/venv/lib/python3.10/site-packages/PyPDF2/_utils.py", line 351, in deprecation\n    raise DeprecationError(msg)\nPyPDF2.errors.DeprecationError: PdfFileReader is deprecated and was removed in PyPDF2 3.0.0. Use PdfReader instead.\n', 'fileLinks': [], 'exitCode': 1}
调用：`bearly_interpreter` with `{'python_code': "from PyPDF2 import PdfReader\n\n# Open the PDF file\npdf = PdfReader('Bristol.pdf')\n\n# Get the text from page 3\npage = pdf.pages[2]\npage_text = page.extract_text()\n\nprint(page_text)"}`
{'stdout': '1 COVID-19 at Work: \nExposing h ow risk is assessed and its consequences in England and Sweden \nPeter Andersson and Tonia Novitz* \n1.Introduction\nT\nhe crisis which arose suddenly at the beginning of 2020 relating to coronavirus was immediately \ncentred on risk. Predictions ha d to be made swiftly regarding how it would spread, who it might \naffect and what measures could be taken to prevent exposure in everyday so cial interaction, \nincluding in the workplace. This was in no way a straightforward assessment, because initially so \nmuch was unknown. Those gaps in our knowledge have since, partially, been ameliorated. It is \nevident that not all those exposed to COVID-19 become ill, and many who contract the virus remain \nasymptomatic, so that the odds on becoming seriously ill may seem small. But those odds are also stacked against certain segments of the population. The likelihood of mortality and morbidity are associated  with age and ethnicity as well as pre -existing medical conditions (such as diabetes), but \nalso with poverty which correlates to the extent of exposure in certain occupations.\n1 Some risks \narise which remain  less predictable, as previously healthy people with no signs of particular \nvulnerability can experience serious long term illness as well and in rare cases will even die.2 \nPerceptions of risk in different countries have led to particular measures taken, ranging from handwashing to social distancing, use of personal protective equipment (PPE) such as face coverings, and even ‘lockdowns’ which have taken various forms.\n3 Use of testing and vaccines \nalso became part of the remedial landscape, with their availability and administration  being \n*This paper is part of the project An  i nclusive and sustainable Swedish labour law – the way\nahead, dnr. 2017-03134 financed by the Swedish research council led by Petra Herzfeld Olssonat Stockholm University. The authors would like to thank her and other participants, Niklas\nBruun and Erik Sjödin for their helpful comments on earlier drafts. A much shorter article titled\n‘Risk Assessment and COVID -19: Systems at work (or not) in England and Sweden’ is published\nin the (2021) Comparative Labour and Social Security Review /\n Revue de droit comparé du\ntravail et de la sécurité sociale.\n1 Public Health England, Disparities in the risk and outcomes of COVID-19 (2 June 2020 -\nhttps://assets.publishing.service.gov.uk/government/uploads/ system /uploads/attachment_data/file\n/890258/disparities_review.pdf.\n2 Nisreen A. Alwan, ‘Track COVID- 19 sickness, not just positive tests and deaths’ ( 2020)\n584.7820 Nature  170- 171; Elisabeth Mahase, ‘Covid-19: What do we know about “long covid”?’\n(2020) BMJ  370.\n3 Sarah Dryhurst, Claudia R. Schneider, John Kerr, Alexandra LJ Freeman, Gabriel Recchia,\nAnne Marthe Van Der Bles, David Spiegelhalter, and Sander van der Linden, ‘Risk perceptionsof COVID-19 around the world’ (2020) 23(7- 8) Journal of Risk Research  994; Wändi Bruine de\nBruin, and Daniel Bennett, ‘Relationships between initial COVID -19 risk perceptions and\nprotective health behaviors: A national survey’ (2020) 59(2) American Journal of Preventive\nMedicine  157; and Simon Deakin and Gaofeng Meng, ‘The Governance of Covid- 19:\nAnthropogenic Risk, Evolutionary Learning, and the Future of the Social State’ (2020)49(4) Industrial Law Journal  539.\n', 'stderr': '', 'fileLinks': [], 'exitCode': 0}
PDF 的第 3 页上的文本是：
"1 COVID-19 at Work: 
Exposing how risk is assessed and its consequences in England and Sweden 
Peter Andersson and Tonia Novitz* 
1.Introduction
The crisis which arose suddenly at the beginning of 2020 relating to coronavirus was immediately 
centred on risk. Predictions had to be made swiftly regarding how it would spread, who it might 
affect and what measures could be taken to prevent exposure in everyday social interaction, 
including in the workplace. This was in no way a straightforward assessment, because initially so 
much was unknown. Those gaps in our knowledge have since, partially, been ameliorated. It is 
evident that not all those exposed to COVID-19 become ill, and many who contract the virus remain 
asymptomatic, so that the odds on becoming seriously ill may seem small. But those odds are also stacked against certain segments of the population. The likelihood of mortality and morbidity are associated  with age and ethnicity as well as pre-existing medical conditions (such as diabetes), but 
also with poverty which correlates to the extent of exposure in certain occupations.
1 Some risks 
arise which remain  less predictable, as previously healthy people with no signs of particular 
vulnerability can experience serious long term illness as well and in rare cases will even die.2 
Perceptions of risk in different countries have led to particular measures taken, ranging from handwashing to social distancing, use of personal protective equipment (PPE) such as face coverings, and even ‘lockdowns’ which have taken various forms.
3 Use of testing and vaccines 
also became part of the remedial landscape, with their availability and administration  being 
*This paper is part of the project An  inclusive and sustainable Swedish labour law – the way
ahead, dnr. 2017-03134 financed by the Swedish research council led by Petra Herzfeld Olssonat Stockholm University. The authors would like to thank her and other participants, Niklas
Bruun and Erik Sjödin for their helpful comments on earlier drafts. A much shorter article titled
‘Risk Assessment and COVID -19: Systems at work (or not) in England and Sweden’ is published
in the (2021) Comparative Labour and Social Security Review /
 Revue de droit comparé du
travail et de la sécurité sociale.
1 Public Health England, Disparities in the risk and outcomes of COVID-19 (2 June 2020 -
https://assets.publishing.service.gov.uk/government/uploads/ system /uploads/attachment_data/file
/890258/disparities_review.pdf.
2 Nisreen A. Alwan, ‘Track COVID- 19 sickness, not just positive tests and deaths’ ( 2020)
584.7820 Nature  170- 171; Elisabeth Mahase, ‘Covid-19: What do we know about “long covid”?’
(2020) BMJ  370.
3 Sarah Dryhurst, Claudia R. Schneider, John Kerr, Alexandra LJ Freeman, Gabriel Recchia,
Anne Marthe Van Der Bles, David Spiegelhalter, and Sander van der Linden, ‘Risk perceptionsof COVID-19 around the world’ (2020) 23(7- 8) Journal of Risk Research  994; Wändi Bruine de
Bruin, and Daniel Bennett, ‘Relationships between initial COVID -19 risk perceptions and
protective health behaviors: A national survey’ (2020) 59(2) American Journal of Preventive
Medicine  157; and Simon Deakin and Gaofeng Meng, ‘The Governance of Covid- 19:
Anthropogenic Risk, Evolutionary Learning, and the Future of the Social State’ (2020)49(4) Industrial Law Journal  539."
> 链结束。
```

```markdown
在 PDF 的第 3 页上的文本如下：
"1 COVID-19 at Work: 
Exposing how risk is assessed and its consequences in England and Sweden 
Peter Andersson and Tonia Novitz* 
1.引言
2020 年初突然出现的冠状病毒危机立即引起了人们对风险的关注。必须迅速做出关于病毒传播方式、可能受到影响的人群以及可以采取的措施，包括在工作场所预防接触的预测。这绝非一项简单的评估，因为最初有太多未知因素。我们对知识的这些空白部分已经在一定程度上得到弥补。显而易见的是，并非所有接触到 COVID-19 的人都会生病，许多感染病毒的人仍然没有症状，因此患重病的几率似乎很小。但这些几率也对某些人群构成了威胁。死亡和发病的可能性与年龄、种族以及既往疾病状况（如糖尿病）以及贫困有关，而贫困与某些职业中的接触程度相关。1 一些风险仍然不太可预测，因为原本健康、没有特殊脆弱迹象的人也可能出现严重的长期疾病，甚至在个别情况下会死亡。2 不同国家对风险的认知导致采取了特定的措施，从洗手到保持社交距离，使用个人防护装备（PPE）如口罩，甚至实施了各种形式的“封锁”。3 检测和疫苗的使用也成为了应对措施的一部分，它们的可用性和使用情况是......"
![图](https://assets.publishing.service.gov.uk/government/uploads/system/uploads/attachment_data/file/890258/disparities_review.pdf)
*本文是瑞典斯德哥尔摩大学 Petra Herzfeld Olsson 领导的瑞典研究理事会资助的项目“包容和可持续的瑞典劳动法-未来之路”（dnr. 2017-03134）的一部分。作者要感谢她和其他参与者 Niklas Bruun 和 Erik Sjödin 对早期草案的帮助性评论。题为“Risk Assessment and COVID-19: Systems at work (or not) in England and Sweden”的一篇更短的文章已发表在《比较劳工和社会保障评论/法律比较评论》（2021）。
1 Public Health England，《COVID-19 风险和结果的差异》（2020 年 6 月 2 日 - https://assets.publishing.service.gov.uk/government/uploads/system/uploads/attachment_data/file/890258/disparities_review.pdf）。
2 Nisreen A. Alwan，《追踪 COVID-19 疾病，而不仅仅是阳性测试和死亡》（2020）584.7820 Nature 170-171；Elisabeth Mahase，《COVID-19：我们对“长期 COVID”了解多少？》（2020）BMJ 370。
3 Sarah Dryhurst, Claudia R. Schneider, John Kerr, Alexandra LJ Freeman, Gabriel Recchia, Anne Marthe Van Der Bles, David Spiegelhalter 和 Sander van der Linden，《全球对 COVID-19 的风险认知》（2020）23(7-8) Journal of Risk Research 994；Wändi Bruine de Bruin 和 Daniel Bennett，《最初 COVID-19 风险认知与健康防护行为之间的关系：一项全国调查》（2020）59(2) American Journal of Preventive Medicine 157；Simon Deakin 和 Gaofeng Meng，《COVID-19 的治理：人为风险、进化学习和社会国家的未来》（2020）49(4) Industrial Law Journal 539。"
```

如果最新的国内生产总值（GDP）数字增长了50%，到2030年，GDP将约为40,594.518亿美元。

> 完成链。

```python
# 图表输出
agent.run("创建一个美观且标记清晰的国内生产总值（GDP）随时间变化的图表")
```

```output
> 进入新的 AgentExecutor 链...
无法解析工具输入：{'name': 'bearly_interpreter', 'arguments': '{\n  "python_code": "\nimport pandas as pd\nimport matplotlib.pyplot as plt\n\n# Load the data\ndf = pd.read_csv(\'US_GDP.csv\')\n\n# Convert the \'DATE\' column to datetime format\ndf[\'DATE\'] = pd.to_datetime(df[\'DATE\'])\n\n# Plot the data\nplt.figure(figsize=(10,6))\nplt.plot(df[\'DATE\'], df[\'GDP\'], label=\'US GDP\')\nplt.xlabel(\'Year\')\nplt.ylabel(\'GDP (in billions)\')\nplt.title(\'US GDP Over Time\')\nplt.legend()\nplt.grid(True)\nplt.savefig(\'output/US_GDP.png\')\n"\n}'}，因为 `arguments` 不是有效的 JSON。无效或不完整的响应
调用：`bearly_interpreter`，使用 `{'python_code': "\nimport pandas as pd\nimport matplotlib.pyplot as plt\n\n# Load the data\ndf = pd.read_csv('US_GDP.csv')\n\n# Convert the 'DATE' column to datetime format\ndf['DATE'] = pd.to_datetime(df['DATE'])\n\n# Plot the data\nplt.figure(figsize=(10,6))\nplt.plot(df['DATE'], df['GDP'], label='US GDP')\nplt.xlabel('Year')\nplt.ylabel('GDP (in billions)')\nplt.title('US GDP Over Time')\nplt.legend()\nplt.grid(True)\nplt.savefig('output/US_GDP.png')\n"}`
{'stdout': '', 'stderr': '', 'fileLinks': [{'pathname': 'US_GDP.png', 'tempLink': 'https://bearly-cubby.c559ae877a0a39985f534614a037d899.r2.cloudflarestorage.com/prod/bearly-cubby/temp/interpreter/2023_10/089daf37e9e343ba5ff21afaaa78b967c3466a550b3b11bd5c710c052b559e97/sxhM8gop2AYP88n5uHCsOJ6yTYNQm-HimZ70DcwQ4VI.png?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Credential=c058d02de50a3cf0bb7e21c8e2d062c5%2F20231010%2F%2Fs3%2Faws4_request&X-Amz-Date=20231010T000000Z&X-Amz-Expires=604800&X-Amz-SignedHeaders=host&X-Amz-Signature=104dc0d4a4b71eeea1030dda1830059920cb0f354fa00197b439eb8565bf141a', 'size': 34275}], 'exitCode': 0}以下是美国国内生产总值（GDP）随时间变化的图表：
![US GDP Over Time](https://bearly-cubby.c559ae877a0a39985f534614a037d899.r2.cloudflarestorage.com/prod/bearly-cubby/temp/interpreter/2023_10/089daf37e9e343ba5ff21afaaa78b967c3466a550b3b11bd5c710c052b559e97/sxhM8gop2AYP88n5uHCsOJ6yTYNQm-HimZ70DcwQ4VI.png?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Credential=c058d02de50a3cf0bb7e21c8e2d062c5%2F20231010%2F%2Fs3%2Faws4_request&X-Amz-Date=20231010T000000Z&X-Amz-Expires=604800&X-Amz-SignedHeaders=host&X-Amz-Signature=104dc0d4a4b71eeea1030dda1830059920cb0f354fa00197b439eb8565bf141a)
横轴表示年份，纵轴表示以亿计的国内生产总值（GDP）。折线图展示了美国国内生产总值随时间的增长。
> 完成链。
```

```output
'以下是美国国内生产总值（GDP）随时间变化的图表：\n\n![US GDP Over Time](https://bearly-cubby.c559ae877a0a39985f534614a037d899.r2.cloudflarestorage.com/prod/bearly-cubby/temp/interpreter/2023_10/089daf37e9e343ba5ff21afaaa78b967c3466a550b3b11bd5c710c052b559e97/sxhM8gop2AYP88n5uHCsOJ6yTYNQm-HimZ70DcwQ4VI.png?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Credential=c058d02de50a3cf0bb7e21c8e2d062c5%2F20231010%2F%2Fs3%2Faws4_request&X-Amz-Date=20231010T000000Z&X-Amz-Expires=604800&X-Amz-SignedHeaders=host&X-Amz-Signature=104dc0d4a4b71eeea1030dda1830059920cb0f354fa00197b439eb8565bf141a)\n\n横轴表示年份，纵轴表示以亿计的国内生产总值（GDP）。折线图展示了美国国内生产总值随时间的增长。'
```