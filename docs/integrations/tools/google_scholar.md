# 谷歌学术

本文介绍如何使用谷歌学术工具。

```python
%pip install --upgrade --quiet  google-search-results
```

```output
Requirement already satisfied: google-search-results in /home/mohtashimkhan/mambaforge/envs/langchain/lib/python3.9/site-packages (2.4.2)
Requirement already satisfied: requests in /home/mohtashimkhan/mambaforge/envs/langchain/lib/python3.9/site-packages (from google-search-results) (2.31.0)
Requirement already satisfied: charset-normalizer<4,>=2 in /home/mohtashimkhan/mambaforge/envs/langchain/lib/python3.9/site-packages (from requests->google-search-results) (3.3.0)
Requirement already satisfied: idna<4,>=2.5 in /home/mohtashimkhan/mambaforge/envs/langchain/lib/python3.9/site-packages (from requests->google-search-results) (3.4)
Requirement already satisfied: urllib3<3,>=1.21.1 in /home/mohtashimkhan/mambaforge/envs/langchain/lib/python3.9/site-packages (from requests->google-search-results) (1.26.17)
Requirement already satisfied: certifi>=2017.4.17 in /home/mohtashimkhan/mambaforge/envs/langchain/lib/python3.9/site-packages (from requests->google-search-results) (2023.5.7)
```

```python
import os
from langchain_community.tools.google_scholar import GoogleScholarQueryRun
from langchain_community.utilities.google_scholar import GoogleScholarAPIWrapper
```

```python
os.environ["SERP_API_KEY"] = ""
tool = GoogleScholarQueryRun(api_wrapper=GoogleScholarAPIWrapper())
tool.run("LLM Models")
```

```output
'Title: Large language models (LLM) and ChatGPT: what will the impact on nuclear medicine be?\nAuthors: IL Alberts,K Shi\nSummary: IL Alberts, L Mercolli, T Pyka, G Prenosil, K Shi… - European journal of …, 2023 - Springer\nTotal-Citations: 28\n\nTitle: Dynamic Planning with a LLM\nAuthors: G Dagan,F Keller,A Lascarides\nSummary: G Dagan, F Keller, A Lascarides - arXiv preprint arXiv:2308.06391, 2023 - arxiv.org\nTotal-Citations: 3\n\nTitle: Openagi: When llm meets domain experts\nAuthors: Y Ge,W Hua,J Ji,J Tan,S Xu,Y Zhang\nSummary: Y Ge, W Hua, J Ji, J Tan, S Xu, Y Zhang - arXiv preprint arXiv:2304.04370, 2023 - arxiv.org\nTotal-Citations: 19\n\nTitle: Llm-planner: Few-shot grounded planning for embodied agents with large language models\nAuthors: CH Song\nSummary: CH Song, J Wu, C Washington… - Proceedings of the …, 2023 - openaccess.thecvf.com\nTotal-Citations: 28\n\nTitle: The science of detecting llm-generated texts\nAuthors: R Tang,YN Chuang,X Hu\nSummary: R Tang, YN Chuang, X Hu - arXiv preprint arXiv:2303.07205, 2023 - arxiv.org\nTotal-Citations: 23\n\nTitle: X-llm: Bootstrapping advanced large language models by treating multi-modalities as foreign languages\nAuthors: F Chen,M Han,J Shi\nSummary: F Chen, M Han, H Zhao, Q Zhang, J Shi, S Xu… - arXiv preprint arXiv …, 2023 - arxiv.org\nTotal-Citations: 12\n\nTitle: 3d-llm: Injecting the 3d world into large language models\nAuthors: Y Hong,H Zhen,P Chen,S Zheng,Y Du\nSummary: Y Hong, H Zhen, P Chen, S Zheng, Y Du… - arXiv preprint arXiv …, 2023 - arxiv.org\nTotal-Citations: 4\n\nTitle: The internal state of an llm knows when its lying\nAuthors: A Azaria,T Mitchell\nSummary: A Azaria, T Mitchell - arXiv preprint arXiv:2304.13734, 2023 - arxiv.org\nTotal-Citations: 18\n\nTitle: LLM-Pruner: On the Structural Pruning of Large Language Models\nAuthors: X Ma,G Fang,X Wang\nSummary: X Ma, G Fang, X Wang - arXiv preprint arXiv:2305.11627, 2023 - arxiv.org\nTotal-Citations: 15\n\nTitle: Large language models are few-shot testers: Exploring llm-based general bug reproduction\nAuthors: S Kang,J Yoon,S Yoo\nSummary: S Kang, J Yoon, S Yoo - 2023 IEEE/ACM 45th International …, 2023 - ieeexplore.ieee.org\nTotal-Citations: 17'
```
