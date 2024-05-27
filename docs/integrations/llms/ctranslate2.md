

# CTranslate2

**CTranslate2** 是一个用于 Transformer 模型高效推断的 C++ 和 Python 库。

该项目实现了一个自定义运行时，应用了许多性能优化技术，如权重量化、层融合、批次重排序等，以加速并减少 CPU 和 GPU 上 Transformer 模型的内存使用。

有关功能和支持的模型的完整列表包含在[项目的存储库](https://opennmt.net/CTranslate2/guides/transformers.html)中。要开始，请查看官方的[快速入门指南](https://opennmt.net/CTranslate2/quickstart.html)。

要使用，您应该已安装 `ctranslate2` Python 包。

```python
%pip install --upgrade --quiet  ctranslate2
```

要在 CTranslate2 中使用 Hugging Face 模型，首先必须使用 `ct2-transformers-converter` 命令将其转换为 CTranslate2 格式。该命令接受预训练模型名称和转换后模型目录的路径。

```python
# 转换可能需要几分钟时间
!ct2-transformers-converter --model meta-llama/Llama-2-7b-hf --quantization bfloat16 --output_dir ./llama-2-7b-ct2 --force
```

```output
加载检查点分片: 100%|██████████████████| 2/2 [00:01<00:00,  1.81it/s]
```

```python
from langchain_community.llms import CTranslate2
llm = CTranslate2(
    # 上述的 output_dir:
    model_path="./llama-2-7b-ct2",
    tokenizer_name="meta-llama/Llama-2-7b-hf",
    device="cuda",
    # device_index 可以是单个整数或整数列表，表示用于推断的 GPU 的 id：
    device_index=[0, 1],
    compute_type="bfloat16",
)
```

## 单次调用

```python
print(
    llm.invoke(
        "他向我提出了关于独角兽存在的可信证据：",
        max_length=256,
        sampling_topk=50,
        sampling_temperature=0.2,
        repetition_penalty=2,
        cache_static_prompt=False,
    )
)
```

```output
他向我提出了关于独角兽存在的可信证据：1) 它们在古代文本中被提及；更重要的是对他而言（并不是大多数人会被说服的事情），他曾见过其中一只。
我持怀疑态度，但我不想让我的朋友因为他的信仰被完全忽视而感到不安，而没有任何考虑或论证支持 - 这就是为什么我们正在进行这次对话！所以我问是否除了“独角兽”之外可能有其他解释……也许它可能是一只鸵鸟？或者只是另一种类似马的动物，就像斑马一样确实存在，尽管今天没有人类亲眼见过它们，因为缺乏可及性/可用性等……但再说，这些动物在这里并不为人所知……”于是我们开始讨论这些生物是否在地球之外的任何其他地方真的存在，那里只有少数科学家在我们之前冒险，因为技术使得探索超越了曾经认为不可能的边界，当旅行意味着自己步行到达目的地A->B，仅通过脚步声导航，直到通过黑暗夜晚小时穿过树林到达目的地
```

## 多次调用

```python
print(
    llm.generate(
        ["最受欢迎的浪漫歌曲列表：\n1.", "最受欢迎的说唱歌曲列表：\n1."],
        max_length=128,
    )
)
```

```output
generations=[[Generation(text='最受欢迎的浪漫歌曲列表：\n1. 惠特尼·休斯顿的“I Will Always Love You”\n2. 埃尔维斯·普雷斯利的“Can’t Help Falling in Love”\n3. 正义兄弟的“Unchained Melody”\n4. 多莉·帕顿的“I Will Always Love You”\n5. 惠特尼·休斯顿的“I Will Always Love You”\n6. 多莉·帕顿的“I Will Always Love You”\n7. 披头士的“I Will Always Love You”\n8. The Rol', generation_info=None)], [Generation(text='最受欢迎的说唱歌曲列表：\n1. 德雷克的“God’s Plan”\n2. Post Malone的“Rockstar”\n3. Migos的“Bad and Boujee”\n4. Kendrick Lamar的“Humble”\n5. Cardi B的“Bodak Yellow”\n6. DJ Khaled的“I’m the One”\n7. Migos的“Motorsport”\n8. G-Eazy的“No Limit”\n9. Big Sean的“Bounce Back”\n10. “', generation_info=None)]]
```

## 将模型集成到 LLMChain 中

```python
from langchain.chains import LLMChain
from langchain_core.prompts import PromptTemplate
template = """{question}
让我们一步一步地思考。"""
prompt = PromptTemplate.from_template(template)
llm_chain = LLMChain(prompt=prompt, llm=llm)
question = "第一个宝可梦游戏发布的年份中美国总统是谁？"
print(llm_chain.run(question))
```

```output
第一个宝可梦游戏发布的年份中美国总统是谁？
让我们一步一步地思考。1996年是第一个宝可梦游戏发布的年份。
\begin{blockquote}
\begin{itemize}
  \item 1996年是比尔·克林顿担任总统的年份。
  \item 1996年是第一个宝可梦游戏发布的年份。
  \item 1996年是第一个宝可梦游戏发布的年份。
\end{itemize}
\end{blockquote}
我不确定这是否是一个有效的问题，但我确定这是一个有趣的问题。
评论：我不确定这是否是一个有效的问题，但我确定这是一个有趣的问题。
评论：@JoeZ. 我不确定这是否是一个有效的问题，但我确定这是一个有趣的问题。
评论：@JoeZ. 我不确定这是否是一个有效的问题，但我确定这是一个有趣的问题。
评论：@JoeZ. 我不确定这是否是一个有效的问题，但我确定这是一个有趣的问题。
评论：@JoeZ. 我不确定这是否是一个有效的问题，但我确定这是一个有趣的问题。
评论：@JoeZ. 我不确定这是否是一个有效的问题，但我确定这是一个有趣的问题。
评论：@JoeZ. 我不确定这是否是一个有效的问题，但我确定这是一个有趣的问题。
评论：@JoeZ. 我不确定这是否是一个有效的问题，但我确定这是一个有趣的问题。
评论：@JoeZ. 我不确定这是否是一个有效的问题，但我确定这是一个有趣的问题。
评论：@JoeZ. 我不确定这是否是一个有效的问题，但我确定这是一个有趣的问题。
评论：@JoeZ. 我不确定这是否是一个有效的问题，但我确定这是一个有趣的问题。
```