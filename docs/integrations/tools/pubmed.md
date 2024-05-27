# PubMed

[PubMed®](https://pubmed.ncbi.nlm.nih.gov/) 包含来自 `MEDLINE`、生命科学期刊和在线图书的超过 3500 万篇生物医学文献引用。引用可能包括来自 PubMed Central 和出版商网站的全文内容链接。

本笔记将介绍如何使用 `PubMed` 作为一个工具。

```python
%pip install xmltodict
```

```python
from langchain_community.tools.pubmed.tool import PubmedQueryRun
```

```python
tool = PubmedQueryRun()
```

```python
tool.invoke("What causes lung cancer?")
```

```output
'Published: 2024-02-10\nTitle: circEPB41L2 blocks the progression and metastasis in non-small cell lung cancer by promoting TRIP12-triggered PTBP1 ubiquitylation.\nCopyright Information: © 2024. The Author(s).\nSummary::\n非小细胞肺癌（NSCLC）的转移是导致 NSCLC 患者死亡的主要原因，需要新的生物标志物来进行精确诊断和治疗。循环 RNA（circRNA）作为新型非编码 RNA，在调节蛋白代谢方面参与了各种癌症的进展。我们揭示了 circEPB41L2（hsa_circ_0077837）通过调节 E3 泛素连接酶 TRIP12 对 PTBP1 的蛋白代谢，阻断了 NSCLC 的有氧糖酵解、进展和转移的机制。通过核糖体 RNA 去除的 RNA 测序，鉴定了 LUAD 组织中的 57 个上调和 327 个下调的 circRNA。由于在 NSCLC 组织和细胞中 circEPB41L2 的显著降低水平，我们选择了 circEPB41L2。有趣的是，circEPB41L2 在体内外阻断了葡萄糖摄取、乳酸产生、NSCLC 细胞的增殖、迁移和侵袭。在机制上，circEPB41L2 作为一个支架，结合 PTBP1 的 RRM1 结构域和 E3 泛素连接酶 TRIP12，促进了 TRIP12 介导的 PTBP1 多泛素化和降解，这可以通过 TRIP12 的 HECT 结构域突变和 circEPB41L2 缺失来逆转。因此，circEPB41L2 诱导的 PTBP1 抑制导致了 PTBP1 诱导的 PKM2 和 Vimentin 活化，但 PKM1 和 E-cadherin 的失活。这些发现突出了 circEPB41L2 依赖的机制，调节了"Warburg 效应"和 EMT，抑制了 NSCLC 的发展和转移，为 NSCLC 的治疗提供了一个抑制性靶点。\n\nPublished: 2024-01-17\nTitle: The safety of seasonal influenza vaccination among adults prescribed immune checkpoint inhibitors: A self-controlled case series study using administrative data.\nCopyright Information: Copyright © 2024 The Author(s). Published by Elsevier Ltd.. All rights reserv'
```