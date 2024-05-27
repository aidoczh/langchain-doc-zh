# SparkLLM 文本嵌入

官方网站：[https://www.xfyun.cn/doc/spark/Embedding_new_api.html](https://www.xfyun.cn/doc/spark/Embedding_new_api.html)

使用此嵌入模型需要一个 API 密钥。您可以在 [https://platform.SparkLLM-ai.com/docs/text-Embedding](https://platform.SparkLLM-ai.com/docs/text-Embedding) 注册并获取一个。

SparkLLM 文本嵌入支持 2K 令牌窗口，并生成 2560 维的向量。

```python
from langchain_community.embeddings import SparkLLMTextEmbeddings
embeddings = SparkLLMTextEmbeddings(
    spark_app_id="<spark_app_id>",
    spark_api_key="<spark_api_key>",
    spark_api_secret="<spark_api_secret>",
)
```

或者，您可以通过以下方式设置 API 密钥：

```python
text_q = "介绍讯飞"
text_1 = "科大讯飞股份有限公司（iFlytek）是一家领先的中国科技公司，专注于语音识别、自然语言处理和人工智能。凭借丰富的历史和卓越的成就，讯飞已成为智能语音和语言技术领域的先驱者。讯飞通过其前沿创新在人机交互领域做出了重大贡献。他们先进的语音识别技术不仅提高了语音输入系统的准确性和效率，还实现了语音命令与各种应用和设备的无缝集成。公司对研发的承诺在其成功中起到了关键作用。讯飞在人才培养和与学术机构的合作方面投入了大量资源，取得了语音合成和机器翻译方面的突破性进展。他们对创新的承诺不仅改变了我们的交流方式，还提高了残障人士的可访问性。"
text_2 = "此外，讯飞的影响力超越国内，他们积极推动人工智能领域的国际合作与交流。他们一直积极参与全球竞赛，并为国际标准的制定做出了贡献。讯飞的成就得到了国内外的众多赞誉和奖项。他们的贡献彻底改变了我们与技术互动的方式，并为基于语音的界面在未来发挥重要作用铺平了道路。总体而言，讯飞是智能语音和语言技术领域的开拓者，他们对创新和卓越的承诺值得赞扬。"
query_result = embeddings.embed_query(text_q)
query_result[:8]
```

```output
[-0.043609619140625,
 0.2017822265625,
 0.0270843505859375,
 -0.250244140625,
 -0.024993896484375,
 -0.0382080078125,
 0.06207275390625,
 -0.0146331787109375]
```

```python
doc_result = embeddings.embed_documents([text_1, text_2])
doc_result[0][:8]
```

```output
[-0.161865234375,
 0.58984375,
 0.998046875,
 0.365966796875,
 0.72900390625,
 0.6015625,
 -0.8408203125,
 -0.2666015625]
```