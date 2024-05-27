---

sidebar_class_name: hidden

---

[![在 Colab 中打开](https://colab.research.google.com/assets/colab-badge.svg)](https://colab.research.google.com/github/langchain-ai/langchain/blob/master/docs/docs/use_cases/data_generation.ipynb)

# 生成合成数据

合成数据是人工生成的数据，而不是从真实世界事件中收集的数据。它用于模拟真实数据，而不会泄露隐私信息或遇到真实世界的限制。

合成数据的好处：

1. **隐私和安全**：没有真实个人数据会面临泄震风险。

2. **数据增强**：扩展机器学习数据集。

3. **灵活性**：创建特定或罕见场景。

4. **成本效益**：通常比真实数据收集便宜。

5. **合规性**：有助于遵守严格的数据保护法律。

6. **模型鲁棒性**：可以导致更好泛化的 AI 模型。

7. **快速原型**：无需真实数据即可进行快速测试。

8. **受控实验**：模拟特定条件。

9. **数据访问**：在真实数据不可用时的替代方案。

注意：尽管有这些好处，合成数据应谨慎使用，因为它可能并不总是能捕捉到真实世界的复杂性。

## 快速开始

在这个笔记本中，我们将深入探讨使用 langchain 库生成合成医疗账单记录。当您因隐私问题或数据可用性问题不想使用真实患者数据时，这个工具尤为有用。

### 设置

首先，您需要安装 langchain 库及其依赖项。由于我们使用 OpenAI 生成器链，我们也会安装它。由于这是一个实验性库，我们需要在安装时包含 `langchain_experimental`。然后我们导入必要的模块。

```python
%pip install --upgrade --quiet  langchain langchain_experimental langchain-openai
# 设置环境变量 OPENAI_API_KEY 或从 .env 文件加载：
# import dotenv
# dotenv.load_dotenv()
from langchain.prompts import FewShotPromptTemplate, PromptTemplate
from langchain_core.pydantic_v1 import BaseModel
from langchain_experimental.tabular_synthetic_data.openai import (
    OPENAI_TEMPLATE,
    create_openai_data_generator,
)
from langchain_experimental.tabular_synthetic_data.prompts import (
    SYNTHETIC_FEW_SHOT_PREFIX,
    SYNTHETIC_FEW_SHOT_SUFFIX,
)
from langchain_openai import ChatOpenAI
```

## 1. 定义您的数据模型

每个数据集都有一个结构或“模式”。下面的 MedicalBilling 类作为我们合成数据的模式。通过定义这个模式，我们告诉合成数据生成器我们期望的数据形状和性质。

```python
class MedicalBilling(BaseModel):
    patient_id: int
    patient_name: str
    diagnosis_code: str
    procedure_code: str
    total_charge: float
    insurance_claim_amount: float
```

例如，每条记录将具有一个整数 `patient_id`，一个字符串 `patient_name`，以此类推。

## 2. 示例数据

为了指导合成数据生成器，提供一些类似真实世界的示例非常有用。这些示例作为“种子” - 它们代表您想要的数据类型，生成器将使用它们创建更多看起来类似的数据。

以下是一些虚构的医疗账单记录：

```python
examples = [
    {
        "example": """患者 ID: 123456, 患者姓名: 约翰·多, 诊断代码: 
        J20.9, 手术代码: 99203, 总费用: $500, 保险索赔金额: $350"""
    },
    {
        "example": """患者 ID: 789012, 患者姓名: 约翰逊·史密斯, 诊断 
        代码: M54.5, 手术代码: 99213, 总费用: $150, 保险索赔金额: $120"""
    },
    {
        "example": """患者 ID: 345678, 患者姓名: 艾米莉·斯通, 诊断代码: 
        E11.9, 手术代码: 99214, 总费用: $300, 保险索赔金额: $250"""
    },
]
```

## 3. 制作提示模板

生成器并不会神奇地知道如何创建我们的数据；我们需要引导它。我们通过创建提示模板来做到这一点。这个模板有助于指导底层语言模型如何以所需格式生成合成数据。

```python
OPENAI_TEMPLATE = PromptTemplate(input_variables=["example"], template="{example}")
prompt_template = FewShotPromptTemplate(
    prefix=SYNTHETIC_FEW_SHOT_PREFIX,
    examples=examples,
    suffix=SYNTHETIC_FEW_SHOT_SUFFIX,
    input_variables=["subject", "extra"],
    example_prompt=OPENAI_TEMPLATE,
)
```

`FewShotPromptTemplate` 包括：

- `prefix` 和 `suffix`：这些可能包含引导上下文或说明。

- `examples`：我们之前定义的示例数据。

- `input_variables`: 这些变量（"subject"，"extra"）是您可以动态填充的占位符。例如，"subject" 可能被填充为 "medical_billing" 以进一步指导模型。

- `example_prompt`: 这个提示模板是我们希望每个示例行在提示中采用的格式。

## 4. 创建数据生成器

有了模式和提示准备好之后，下一步是创建数据生成器。这个对象知道如何与底层语言模型通信以获取合成数据。

```python
synthetic_data_generator = create_openai_data_generator(
    output_schema=MedicalBilling,
    llm=ChatOpenAI(
        temperature=1
    ),  # 您需要用实际的语言模型实例替换
    prompt=prompt_template,
)
```

## 5. 生成合成数据

最后，让我们获取我们的合成数据！

```python
synthetic_results = synthetic_data_generator.generate(
    subject="medical_billing",
    extra="the name must be chosen at random. Make it something you wouldn't normally choose.",
    runs=10,
)
```

这个命令要求生成器生成 10 条合成的医疗账单记录。结果存储在 `synthetic_results` 中。输出将是 MedicalBilling pydantic 模型的列表。

### 其他实现

```python
from langchain_experimental.synthetic_data import (
    DatasetGenerator,
    create_data_generation_chain,
)
from langchain_openai import ChatOpenAI
```

```python
# 语言模型
model = ChatOpenAI(model="gpt-3.5-turbo", temperature=0.7)
chain = create_data_generation_chain(model)
```

```python
chain({"fields": ["blue", "yellow"], "preferences": {}})
```

```output
{'fields': ['blue', 'yellow'],
 'preferences': {},
 'text': '明亮的蓝天与明亮的黄色太阳形成鲜明对比，创造出令所有凝视者立刻振奋的色彩绚丽的景象。'}
```

```python
chain(
    {
        "fields": {"colors": ["blue", "yellow"]},
        "preferences": {"style": "Make it in a style of a weather forecast."},
    }
)
```

```output
{'fields': {'colors': ['blue', 'yellow']},
 'preferences': {'style': 'Make it in a style of a weather forecast.'},
 'text': "早上好！今天的天气预报为天空带来了美丽的色彩组合，蓝色和黄色的色调温柔地融合在一起，如同一幅迷人的画作。"}
```

```python
chain(
    {
        "fields": {"actor": "Tom Hanks", "movies": ["Forrest Gump", "Green Mile"]},
        "preferences": None,
    }
)
```

```output
{'fields': {'actor': 'Tom Hanks', 'movies': ['Forrest Gump', 'Green Mile']},
 'preferences': None,
 'text': '汤姆·汉克斯，这位以其出色的多才多艺和魅力而闻名的著名演员，曾在《阿甘正传》和《绿里奇迹》等令人难忘的电影中出演。'}
```

```python
chain(
    {
        "fields": [
            {"actor": "Tom Hanks", "movies": ["Forrest Gump", "Green Mile"]},
            {"actor": "Mads Mikkelsen", "movies": ["Hannibal", "Another round"]},
        ],
        "preferences": {"minimum_length": 200, "style": "gossip"},
    }
)
```

```output
{'fields': [{'actor': 'Tom Hanks', 'movies': ['Forrest Gump', 'Green Mile']},
  {'actor': 'Mads Mikkelsen', 'movies': ['Hannibal', 'Another round']}],
 'preferences': {'minimum_length': 200, 'style': 'gossip'},
 'text': '您是否知道，汤姆·汉克斯，这位备受喜爱的好莱坞演员，以其在《阿甘正传》和《绿里奇迹》中的角色而闻名，曾与备受国际赞誉的马兹·米凯尔森一同出演，后者以《汉尼拔》和《再一轮》中的表演而赢得了国际声誉。这两位出色的演员将其卓越的技能和迷人的魅力带到大银幕上，呈现出令人难忘的表演，深深吸引了全球观众。无论是汉克斯饰演的阿甘还是米凯尔森饰演的汉尼拔·莱克特，这些电影都巩固了它们在电影史上的地位，给观众留下了深刻印象，巩固了它们作为银幕真正偶像的地位。'}
```

我们可以看到，创建的示例多样化，并具有我们希望它们具有的信息。此外，它们的风格很好地反映了给定的偏好。

## 为提取基准目的生成示例数据集

```python
inp = [
    {
        "Actor": "Tom Hanks",
        "Film": [
            "Forrest Gump",
            "Saving Private Ryan",
            "The Green Mile",
            "Toy Story",
            "Catch Me If You Can",
        ],
    },
    {
        "Actor": "Tom Hardy",
        "Film": [
            "Inception",
            "The Dark Knight Rises",
            "Mad Max: Fury Road",
            "The Revenant",
            "Dunkirk",
        ],
    },
]
generator = DatasetGenerator(model, {"style": "informal", "minimal length": 500})
dataset = generator(inp)
```

```python
数据集
```

```output
[{'fields': {'Actor': 'Tom Hanks',
   'Film': ['Forrest Gump',
    'Saving Private Ryan',
    'The Green Mile',
    'Toy Story',
    'Catch Me If You Can']},
  'preferences': {'style': 'informal', 'minimal length': 500},
  'text': '汤姆·汉克斯，这位多才多艺、魅力非凡的演员，在许多标志性电影中出演，包括温馨励志的《阿甘正传》，紧张感人的战争剧《拯救大兵瑞恩》，情感充沛发人深省的《绿里奇迹》，备受喜爱的经典动画片《玩具总动员》，以及扣人心弦的真实故事改编片《逍遥法外》。凭借其出色的表演范围和真挚的才华，汉克斯持续吸引全球观众，给电影界留下了不可磨灭的印记。'},
 {'fields': {'Actor': 'Tom Hardy',
   'Film': ['Inception',
    'The Dark Knight Rises',
    'Mad Max: Fury Road',
    'The Revenant',
    'Dunkirk']},
  'preferences': {'style': 'informal', 'minimal length': 500},
  'text': '汤姆·哈迪，这位多才多艺的演员以其激烈的表演而闻名，出演了许多标志性电影，包括《盗梦空间》，《黑暗骑士崛起》，《疯狂麦克斯：狂暴之路》，《荒野猎人》，以及《敦刻尔克》。无论是深入潜意识的深度，扮演臭名昭著的班恩，还是在荒凉的废墟中扮演神秘的麦克斯·洛卡塔斯基，哈迪对自己的表演执着始终显而易见。从在《盗梦空间》中无情的伊姆斯的惊艳表演，到在《疯狂麦克斯：狂暴之路》中变身为凶猛的麦克斯，哈迪的动态表现和磁性存在感吸引观众，给电影界留下了不可磨灭的印记。在迄今为止最具挑战性的角色中，他在《荒野猎人》中扮演坚韧的拓荒者约翰·菲茨杰拉德，忍受着严酷的冻野条件，赢得了广泛好评和奥斯卡提名。在克里斯托弗·诺兰的战争史诗片《敦刻尔克》中，哈迪扮演皇家空军飞行员法里尔，刻画出他通过细腻表演传达深情的能力。凭借他变色龙般的才能，扮演各种角色的能力，以及对自己工作的坚定承诺，汤姆·哈迪无疑巩固了自己作为这一代最有才华和备受追捧的演员之一的地位。'}]
```

## 从生成的示例中提取信息

好的，让我们看看现在是否可以从这些生成的数据中提取输出，以及与我们的案例相比如何！

```python
from typing import List
from langchain.chains import create_extraction_chain_pydantic
from langchain_core.output_parsers import PydanticOutputParser
from langchain_core.prompts import PromptTemplate
from langchain_openai import OpenAI
from pydantic import BaseModel, Field
```

```python
class Actor(BaseModel):
    Actor: str = Field(description="演员的姓名")
    Film: List[str] = Field(description="他们主演的电影名称列表")
```

### 解析器

```python
llm = OpenAI()
parser = PydanticOutputParser(pydantic_object=Actor)
prompt = PromptTemplate(
    template="从给定的文本中提取字段。\n{format_instructions}\n{text}\n",
    input_variables=["text"],
    partial_variables={"format_instructions": parser.get_format_instructions()},
)
_input = prompt.format_prompt(text=dataset[0]["text"])
output = llm(_input.to_string())
parsed = parser.parse(output)
parsed
```

```output
Actor(Actor='汤姆·汉克斯', Film=['阿甘正传', '拯救大兵瑞恩', '绿里奇迹', '玩具总动员', '骗中骗'])
```

```python
(parsed.Actor == inp[0]["Actor"]) & (parsed.Film == inp[0]["Film"])
```

```output
True
```

### 提取器

```python
extractor = create_extraction_chain_pydantic(pydantic_schema=Actor, llm=model)
extracted = extractor.run(dataset[1]["text"])
extracted
```

```output
[Actor(Actor='汤姆·哈迪', Film=['盗梦空间', '黑暗骑士崛起', '疯狂麦克斯：狂暴之路', '荒野猎人', '敦刻尔克'])]
```

```python
(extracted[0].Actor == inp[1]["Actor"]) & (extracted[0].Film == inp[1]["Film"])
```

```output
True
```
