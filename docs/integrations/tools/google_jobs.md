# 谷歌职位

本文介绍如何使用谷歌职位工具来获取当前的职位发布信息。

首先，您需要在 https://serpapi.com/users/sign_up 注册一个 `SerpApi key`。

然后，您需要使用以下命令安装 `google-search-results`：

```bash
pip install google-search-results
```

接下来，您需要将环境变量 `SERPAPI_API_KEY` 设置为您的 `SerpApi key`。

如果您还没有 `SerpApi key`，您可以在 https://serpapi.com/users/sign_up 注册一个免费账户，并在这里获取您的 api key: https://serpapi.com/manage-api-key

如果您使用 conda 环境，您可以在内核中使用以下命令进行设置：

```bash
conda activate [your env name]
conda env confiv vars SERPAPI_API_KEY='[your serp api key]'
```

## 使用该工具

```python
%pip install --upgrade --quiet  google-search-results
```

```output
Requirement already satisfied: google-search-results in /opt/anaconda3/envs/langchain/lib/python3.10/site-packages (2.4.2)
Requirement already satisfied: requests in /opt/anaconda3/envs/langchain/lib/python3.10/site-packages (from google-search-results) (2.31.0)
Requirement already satisfied: charset-normalizer<4,>=2 in /opt/anaconda3/envs/langchain/lib/python3.10/site-packages (from requests->google-search-results) (3.3.2)
Requirement already satisfied: idna<4,>=2.5 in /opt/anaconda3/envs/langchain/lib/python3.10/site-packages (from requests->google-search-results) (3.4)
Requirement already satisfied: urllib3<3,>=1.21.1 in /opt/anaconda3/envs/langchain/lib/python3.10/site-packages (from requests->google-search-results) (2.1.0)
Requirement already satisfied: certifi>=2017.4.17 in /opt/anaconda3/envs/langchain/lib/python3.10/site-packages (from requests->google-search-results) (2023.11.17)
```

```python
import os
from langchain_community.tools.google_jobs import GoogleJobsQueryRun
from langchain_community.utilities.google_jobs import GoogleJobsAPIWrapper
os.environ["SERPAPI_API_KEY"] = "[your serpapi key]"
tool = GoogleJobsQueryRun(api_wrapper=GoogleJobsAPIWrapper())
```

```python
tool.run("Can I get an entry level job posting related to physics")
```

```output
"\n_______________________________________________\n职位名称: 应用物理学家 (有经验或高级)\n公司名称: 波音\n地点:   加利福尼亚州亨廷顿比奇   \n描述: 职位描述\n\n在波音，我们进行创新和合作，使世界变得更美好。从海底到外太空，您可以为一个重视多样性、公平和包容的公司做出有意义的贡献。我们致力于营造一个对每个团队成员都友好、尊重和包容的环境，并提供良好的专业成长机会。与我们一起，找到您的...未来。\n\n我们是波音研究与技术（BR&T）：波音的全球研发团队，致力于创造和实施创新技术，使不可能成为可能，并推动航空航天的未来。我们是工程师和技术人员，是熟练的科学家和大胆的创新者；加入我们，将您的激情、决心和技能投入到构建未来的工作中！\n\n加入一个不断壮大的团队，将新兴技术转化为创新产品和服务。波音处于重新构想商用和政府市场航空航天平台设计、制造和运营未来的前沿。\n\n此职位将设在加利福尼亚州亨廷顿比奇。\n\n职位职责:\n• 为各种复杂的通信、传感器、电子战和其他电磁系统和部件开发和验证要求。\n• 为电气/电子系统、机械系统、互连和结构开发和验证电磁要求。\n• 开发用于将复杂系统和部件集成到更高级系统和平台的架构。\n• 进行复杂的权衡研究、建模、仿真和其他形式的分析，以预测组件、互连和系统的性能，并围绕已建立的要求优化设计。\n• 定义并进行各种关键测试，以验证设计对要求的性能。管理适当的供应商和合作伙伴的关键绩效，以确保符合要求。\n• 通过提供指导和支持解决复杂问题，支持产品从制造到客户使用的整个生命周期。\n• 通过提供协调工作声明、预算、时间表和其他所需输入，并进行适当的审查，支持项目管理。\n• 生成提案的重要部分，以支持新业务的发展。\n• 在最小的指导下工作。\n\n应用物理学家（有经验或高级），BR&T/先进计算技术 – 候选人将应用他们的量子物理知识来构建实验量子传感或量子网络的全面能力套件。成功的候选人将对以下至少一个领域有深入的理论和实验室实践的理解：\n• 光钟\n• 光时间传输\n• 光频梳基础计量\n• 量子网络基础上的量子系统纠缠（例如，原子、离子或量子点系统）\n\n成功的候选人将开发\n• 由机构内和机构外资金支持的充满活力的研发计划\n• 撰写项目提案\n• 开发未来产品概念\n• 协助将量子技术整合到波音商用和国防业务的未来产品和服务中\n\n此职位允许远程办公。所选候选人将需要在列出的位置选项之一的现场执行一些工作。\n\n此职位要求能够获得美国安全许可。美国政府要求美国公民获得美国安全许可。入职后需要获得临时和/或最终的美国机密许可。\n\n基本资格（必需技能/经验）\n• 物理学、化学、电气工程或其他与量子传感和/或量子信息科学相关领域的博士学位\n• 撰写并发表研究论文和项目（学术界或专业界）\n• 在以下领域进行大学研究和实验室实践之一：光钟、光时间传输、原子干涉仪或量子网络基础上的量子系统纠缠（例如，原子、离子或量子点系统）\n\n首选资格（期望技能/经验）\n• 9年以上相关工作经验或教育和经验的等效组合\n• 有效的美国安全许可\n\n典型教育/经验：\n\n有经验：通常通过来自工程、计算机科学、数学、物理或化学的认可课程学习获得的高级技术教育（例如学士学位），通常具有9年或更多相关工作经验或技术教育和经验的等效组合（例如博士学位+4年相关工作经验，硕士+7年相关工作经验）。在美国，ABET 认证是首选的，尽管不是必需的认证标准\n\n高级：通常通过来自工程、计算机科学、数学、物理或化学的认可课程学习获得的高级技术教育（例如学士学位），通常具有14年或更多相关工作经验或技术教育和经验的等效组合（例如博士学位+9年相关工作经验，硕士+12年相关工作经验）。在美国，ABET 认证是首选的，尽管不是必需的认证标准。\n\n搬迁：此职位根据候选人的资格提供搬迁服务\n\n波音是一个无毒工作场所，符合条件的入职申请者和员工在符合我们政策规定的情况下需接受大麻、可卡因、阿片类药物、安非他明、PCP 和酒精测试。\n\n班次：此职位是第一班次。\n\n在波音，我们努力提供一个全面的奖励计划，以吸引、激励和留住顶尖人才。奖励计划的要素包括有竞争力的基本工资和有机会的变动报酬。\n\n波音公司还为符合条件的员工提供了参加各种福利计划的机会，通常包括健康保险、灵活支出账户、健康储蓄账户、退休储蓄计划、人身和伤残保险计划，以及一些提供带薪和无薪休假的计划。任何员工可用的具体计划和选项可能会因资格因素（如地理位置、入职日期）和集体谈判协议的适用性等因素而有所不同。\n\n请注意，下面显示的薪资信息仅供参考。薪资根据候选人的经验和资格、市场和业务考虑因素而定。\n\n有经验的薪资范围: $126,000 – $171,000\n\n高级的薪资范围: $155,000 - $210,00\n\n出口管制要求：美国政府出口管制状态：此职位必须符合出口管制合规要求。为满足出口管制合规要求，必须符合 22 C.F.R. §120.15 定义的“美国人”要求。 “美国人”包括美国公民、合法永久居民、难民或寻求庇护者。\n\n出口管制详细信息：基于美国的工作，需要美国人\n\n平等机会雇主：\n\n波音是一家平等机会雇主。就业决策不会受到种族、肤色、宗教、国籍、性别、性取向、性别认同、年龄、身体或精神残疾、遗传因素、军事/退伍军人身份或法律保护的其他特征的影响\n_______________________________________________\n\n"
```

```python
# 与 langchain 一起使用
import os
from langchain.agents import AgentType, initialize_agent, load_tools
from langchain_openai import OpenAI
OpenAI.api_key = os.environ["OPENAI_API_KEY"]
llm = OpenAI()
tools = load_tools(["google-jobs"], llm=llm)
agent = initialize_agent(tools, llm, agent=AgentType.ZERO_SHOT_REACT_DESCRIPTION, verbose=True)
agent.run("给我一个与物理相关的初级职位招聘信息")
```

```output
> 进入新的 AgentExecutor 链...
我应该使用一个工具来搜索与物理相关的职位招聘信息
动作：google_jobs
动作输入：entry level physics
观察结果：
_______________________________________________
职位名称：Entry-level Calibration Technician (great for new Physics or EE grads)
公司名称：Modis Engineering
地点：Everett, WA
描述：Akkodis 提供了 Everett, WA 的 Electronics Calibration Technician 合同到聘用职位。
这是一个很好的机会，适合物理、BSEET、BSEE、机电一体化或其他毕业生喜欢动手工作的人。或者适合有1-5年经验的技术员...
班次：晚班或夜班
薪酬：$28 - $32/小时，根据经验而定
如果表现良好，一旦职位转为全职/薪酬/直聘职位，可能会增加每小时$2。
• **将提供全面培训***
工作职责包括：
- 校准和测试光纤测试设备。
- 记录校准过程中收集的数据。
- 识别超出容差范围的情况。
- 与其他技术人员、客户服务代表和客户互动。
资格要求：
- 物理学或电气工程学士学位 - 或 - 电子学（或类似学科）的副学士学位 - 或 - 电子经验。
- 必须具备良好的书面和口头沟通能力。
- 从学校实验室工作或其他地方获得的基本电路测试/故障排除经验。
- 必须具备基本的计算机技能。
- 校准经验是一个加分项。
- 光纤和计量设备经验是一个加分项。
- 这个职位不涉及作为维修技术员的故障排除，但在这个领域有经验是可以接受和有帮助的。
- 这个职位需要良好的职业道德和学习和成功的意愿。
平等机会雇主/退伍军人/残疾人
福利包括医疗、牙科、视觉、灵活支出账户、健康储蓄账户、短期和长期残疾保险以及401K计划。还包括全面的带薪休假计划。
免责声明：这些福利不适用于由客户招聘的工作和直接雇佣给客户的工作。
要阅读我们的候选人隐私信息声明，了解我们将如何使用您的信息，请访问 https://www.modis.com/en-us/candidate-privacy
_______________________________________________
思考：我现在知道最终答案了
最终答案：与物理相关的初级职位招聘信息是位于 Everett, WA 的 Modis Engineering 公司的校准技术员职位。该职位提供每小时 $28 - $32 的竞争性薪酬，并提供全面培训。资格要求包括物理学或电气工程学士学位，或电子学或类似学科的副学士学位，福利包括医疗、牙科、视觉等。
> 完成链。
```

```output
'与物理相关的初级职位招聘信息是位于 Everett, WA 的 Modis Engineering 公司的校准技术员职位。该职位提供每小时 $28 - $32 的竞争性薪酬，并提供全面培训。资格要求包括物理学或电气工程学士学位，或电子学或类似学科的副学士学位，福利包括医疗、牙科、视觉等。'
```