# ClearML

[ClearML](https://github.com/allegroai/clearml) 是一个机器学习/深度学习开发和生产套件，包含以下5个主要模块：

- `实验管理器` - 自动化实验跟踪、环境和结果

- `MLOps` - 用于机器学习/深度学习作业的编排、自动化和流水线解决方案（K8s / 云 / 金属）

- `数据管理` - 基于对象存储（S3 / GS / Azure / NAS）的完全可区分的数据管理和版本控制解决方案

- `模型服务` - 云就绪的可扩展模型服务解决方案！

    在不到5分钟内部署新的模型端点

    包括由Nvidia-Triton支持的优化GPU服务

    具备开箱即用的模型监控功能

- `火报告` - 创建和共享支持嵌入式在线内容的丰富的Markdown文档

为了正确跟踪您的langchain实验及其结果，您可以启用`ClearML`集成。我们使用`ClearML实验管理器`来整洁地跟踪和组织所有实验运行。

<a target="_blank" href="https://colab.research.google.com/github/langchain-ai/langchain/blob/master/docs/docs/integrations/providers/clearml_tracking.ipynb">

  <img src="https://colab.research.google.com/assets/colab-badge.svg" alt="Open In Colab"/>

</a>

## 安装和设置

```python
%pip install --upgrade --quiet  clearml
%pip install --upgrade --quiet  pandas
%pip install --upgrade --quiet  textstat
%pip install --upgrade --quiet  spacy
!python -m spacy download en_core_web_sm
```

### 获取API凭据

我们将在本笔记本中使用一些API，以下是这些API的列表以及获取它们的位置：

- ClearML: https://app.clear.ml/settings/workspace-configuration

- OpenAI: https://platform.openai.com/account/api-keys

- SerpAPI (google search): https://serpapi.com/dashboard

```python
import os
os.environ["CLEARML_API_ACCESS_KEY"] = ""
os.environ["CLEARML_API_SECRET_KEY"] = ""
os.environ["OPENAI_API_KEY"] = ""
os.environ["SERPAPI_API_KEY"] = ""
```

## 回调函数

```python
from langchain_community.callbacks import ClearMLCallbackHandler
```

```python
from langchain_core.callbacks import StdOutCallbackHandler
from langchain_openai import OpenAI
# 设置并使用ClearML回调函数
clearml_callback = ClearMLCallbackHandler(
    task_type="inference",
    project_name="langchain_callback_demo",
    task_name="llm",
    tags=["test"],
    # 根据您想要跟踪的详细程度更改以下参数
    visualize=True,
    complexity_metrics=True,
    stream_logs=True,
)
callbacks = [StdOutCallbackHandler(), clearml_callback]
# 准备好OpenAI模型
llm = OpenAI(temperature=0, callbacks=callbacks)
```

```output
clearml回调函数目前处于测试阶段，可能会根据`langchain`的更新而发生变化。请将任何问题报告给https://github.com/allegroai/clearml/issues，并使用标签`langchain`。
```

### 场景1：只有一个LLM

首先，让我们运行几次单个LLM，并在ClearML中捕获生成的提示-回答对话。

```python
# 场景1 - LLM
llm_result = llm.generate(["Tell me a joke", "Tell me a poem"] * 3)
# 在每次生成运行之后，使用flush确保所有指标、提示和其他输出都被正确保存
clearml_callback.flush_tracker(langchain_asset=llm, name="simple_sequential")
```

```output
{'action': 'on_llm_start', 'name': 'OpenAI', 'step': 3, 'starts': 2, 'ends': 1, 'errors': 0, 'text_ctr': 0, 'chain_starts': 0, 'chain_ends': 0, 'llm_starts': 2, 'llm_ends': 1, 'llm_streams': 0, 'tool_starts': 0, 'tool_ends': 0, 'agent_ends': 0, 'prompts': 'Tell me a joke'}
{'action': 'on_llm_start', 'name': 'OpenAI', 'step': 3, 'starts': 2, 'ends': 1, 'errors': 0, 'text_ctr': 0, 'chain_starts': 0, 'chain_ends': 0, 'llm_starts': 2, 'llm_ends': 1, 'llm_streams': 0, 'tool_starts': 0, 'tool_ends': 0, 'agent_ends': 0, 'prompts': 'Tell me a poem'}
{'action': 'on_llm_start', 'name': 'OpenAI', 'step': 3, 'starts': 2, 'ends': 1, 'errors': 0, 'text_ctr': 0, 'chain_starts': 0, 'chain_ends': 0, 'llm_starts': 2, 'llm_ends': 1, 'llm_streams': 0, 'tool_starts': 0, 'tool_ends': 0, 'agent_ends': 0, 'prompts': 'Tell me a joke'}
{'action': 'on_llm_start', 'name': 'OpenAI', 'step': 3, 'starts': 2, 'ends': 1, 'errors': 0, 'text_ctr': 0, 'chain_starts': 0, 'chain_ends': 0, 'llm_starts': 2, 'llm_ends': 1, 'llm_streams': 0, 'tool_starts': 0, 'tool_ends': 0, 'agent_ends': 0, 'prompts': 'Tell me a poem'}
{'action': 'on_llm_start', 'name': 'OpenAI', 'step': 3, 'starts': 2, 'ends': 1, 'errors': 0, 'text_ctr': 0, 'chain_starts': 0, 'chain_ends': 0, 'llm_starts': 2, 'llm_ends': 1, 'llm_streams': 0, 'tool_starts': 0, 'tool_ends': 0, 'agent_ends': 0, 'prompts': 'Tell me a joke'}
{'action': 'on_llm_start', 'name': 'OpenAI', 'step': 3, 'starts': 2, 'ends': 1, 'errors': 0, 'text_ctr': 0, 'chain_starts': 0, 'chain_ends': 0, 'llm_starts': 2, 'llm_ends': 1, 'llm_streams': 0, 'tool_starts': 0, 'tool_ends': 0, 'agent_ends': 0, 'prompts': 'Tell me a poem'}
{'action': 'on_llm_end', 'token_usage_prompt_tokens': 24, 'token_usage_completion_tokens': 138, 'token_usage_total_tokens': 162, 'model_name': 'text-davinci-003', 'step': 4, 'starts': 2, 'ends': 2, 'errors': 0, 'text_ctr': 0, 'chain_starts': 0, 'chain_ends': 0, 'llm_starts': 2, 'llm_ends': 2, 'llm_streams': 0, 'tool_starts': 0, 'tool_ends': 0, 'agent_ends': 0, 'text': '\n\nQ: What did the fish say when it hit the wall?\nA: Dam!', 'generation_info_finish_reason': 'stop', 'generation_info_logprobs': None, 'flesch_reading_ease': 109.04, 'flesch_kincaid_grade': 1.3, 'smog_index': 0.0, 'coleman_liau_index': -1.24, 'automated_readability_index': 0.3, 'dale_chall_readability_score': 5.5, 'difficult_words': 0, 'linsear_write_formula': 5.5, 'gunning_fog': 5.2, 'text_standard': '5th and 6th grade', 'fernandez_huerta': 133.58, 'szigriszt_pazos': 131.54, 'gutierrez_polini': 62.3, 'crawford': -0.2, 'gulpease_index': 79.8, 'osman': 116.91}
{'action': 'on_llm_end', 'token_usage_prompt_tokens': 24, 'token_usage_completion_tokens': 138, 'token_usage_total_tokens': 162, 'model_name': 'text-davinci-003', 'step': 4, 'starts': 2, 'ends': 2, 'errors': 0, 'text_ctr': 0, 'chain_starts': 0, 'chain_ends': 0, 'llm_starts': 2, 'llm_ends': 2, 'llm_streams': 0, 'tool_starts': 0, 'tool_ends': 0, 'agent_ends': 0, 'text': '\n\nRoses are red,\nViolets are blue,\nSugar is sweet,\nAnd so are you.', 'generation_info_finish_reason': 'stop', 'generation_info_logprobs': None, 'flesch_reading_ease': 83.66, 'flesch_kincaid_grade': 4.8, 'smog_index': 0.0, 'coleman_liau_index': 3.23, 'automated_readability_index': 3.9, 'dale_chall_readability_score': 6.71, 'difficult_words': 2, 'linsear_write_formula': 6.5, 'gunning_fog': 8.28, 'text_standard': '6th and 7th grade', 'fernandez_huerta': 115.58, 'szigriszt_pazos': 112.37, 'gutierrez_polini': 54.83, 'crawford': 1.4, 'gulpease_index': 72.1, 'osman': 100.17}
{'action': 'on_llm_end', 'token_usage_prompt_tokens': 24, 'token_usage_completion_tokens': 138, 'token_usage_total_tokens': 162, 'model_name': 'text-davinci-003', 'step': 4, 'starts': 2, 'ends': 2, 'errors': 0, 'text_ctr': 0, 'chain_starts': 0, 'chain_ends': 0, 'llm_starts': 2, 'llm_ends': 2, 'llm_streams': 0, 'tool_starts': 0, 'tool_ends': 0, 'agent_ends': 0, 'text': '\n\nQ: What did the fish say when it hit the wall?\nA: Dam!', 'generation_info_finish_reason': 'stop', 'generation_info_logprobs': None, 'flesch_reading_ease': 109.04, 'flesch_kincaid_grade': 1.3, 'smog_index': 0.0, 'coleman_liau_index': -1.24, 'automated_readability_index': 0.3, 'dale_chall_readability_score': 5.5, 'difficult_words': 0, 'linsear_write_formula': 5.5, 'gunning_fog': 5.2, 'text_standard': '5th and 6th grade', 'fernandez_huerta': 133.58, 'szigriszt_pazos': 131.54, 'gutierrez_polini': 62.3, 'crawford': -0.2, 'gulpease_index': 79.8, 'osman': 116.91}
{'action': 'on_llm_end', 'token_usage_prompt_tokens': 24, 'token_usage_completion_tokens': 138, 'token_usage_total_tokens': 162, 'model_name': 'text-davinci-003', 'step': 4, 'starts': 2, 'ends': 2, 'errors': 0, 'text_ctr': 0, 'chain_starts': 0, 'chain_ends': 0, 'llm_starts': 2, 'llm_ends': 2, 'llm_streams': 0, 'tool_starts': 0, 'tool_ends': 0, 'agent_ends': 0, 'text': '\n\nRoses are red,\nViolets are blue,\nSugar is sweet,\nAnd so are you.', 'generation_info_finish_reason': 'stop', 'generation_info_logprobs': None, 'flesch_reading_ease': 83.66, 'flesch_kincaid_grade': 4.8, 'smog_index': 0.0, 'coleman_liau_index': 3.23, 'automated_readability_index': 3.9, 'dale_chall_readability_score': 6.71, 'difficult_words': 2, 'linsear_write_formula': 6.5, 'gunning_fog': 8.28, 'text_standard': '6th and 7th grade', 'fernandez_huerta': 115.58, 'szigriszt_pazos': 112.37, 'gutierrez_polini': 54.83, 'crawford': 1.4, 'gulpease_index': 72.1, 'osman': 100.17}
{'action': 'on_llm_end', 'token_usage_prompt_tokens': 24, 'token_usage_completion_tokens': 138, 'token_usage_total_tokens': 162, 'model_name': 'text-davinci-003', 'step': 4, 'starts': 2, 'ends': 2, 'errors': 0, 'text_ctr': 0, 'chain_starts': 0, 'chain_ends': 0, 'llm_starts': 2, 'llm_ends': 2, 'llm_streams': 0, 'tool_starts': 0, 'tool_ends': 0, 'agent_ends': 0, 'text': '\n\nQ: What did the fish say when it hit the wall?\nA: Dam!', 'generation_info_finish_reason': 'stop', 'generation_info_logprobs': None, 'flesch_reading_ease': 109.04, 'flesch_kincaid_grade': 1.3, 'smog_index': 0.0, 'coleman_liau_index': -1.24, 'automated_readability_index': 0.3, 'dale_chall_readability_score': 5.5, 'difficult_words': 0, 'linsear_write_formula': 5.5, 'gunning_fog': 5.2, 'text_standard': '5th and 6th grade', 'fernandez_huerta': 133.58, 'szigriszt_pazos': 131.54, 'gutierrez_polini': 62.3, 'crawford': -0.2, 'gulpease_index': 79.8, 'osman': 116.91}
{'action': 'on_llm_end', 'token_usage_prompt_tokens': 24, 'token_usage_completion_tokens': 138, 'token_usage_total_tokens': 162, 'model_name': 'text-davinci-003', 'step': 4, 'starts': 2, 'ends': 2, 'errors': 0, 'text_ctr': 0, 'chain_starts': 0, 'chain_ends': 0, 'llm_starts': 2, 'llm_ends': 2, 'llm_streams': 0, 'tool_starts': 0, 'tool_ends': 0, 'agent_ends': 0, 'text': '\n\nRoses are red,\nViolets are blue,\nSugar is sweet,\nAnd so are you.', 'generation_info_finish_reason': 'stop', 'generation_info_logprobs': None, 'flesch_reading_ease': 83.66, 'flesch_kincaid_grade': 4.8, 'smog_index': 0.0, 'coleman_liau_index': 3.23, 'automated_readability_index': 3.9, 'dale_chall_readability_score': 6.71, 'difficult_words': 2, 'linsear_write_formula': 6.5, 'gunning_fog': 8.28, 'text_standard': '6th and 7th grade', 'fernandez_huerta': 115.58, 'szigriszt_pazos': 112.37, 'gutierrez_polini': 54.83, 'crawford': 1.4, 'gulpease_index': 72.1, 'osman': 100.17}
{'action_records':           action    name  step  starts  ends  errors  text_ctr  chain_starts  \
0   on_llm_start  OpenAI     1       1     0       0         0             0   
1   on_llm_start  OpenAI     1       1     0       0         0             0   
2   on_llm_start  OpenAI     1       1     0       0         0             0   
3   on_llm_start  OpenAI     1       1     0       0         0             0   
4   on_llm_start  OpenAI     1       1     0       0         0             0   
5   on_llm_start  OpenAI     1       1     0       0         0             0   
6     on_llm_end     NaN     2       1     1       0         0             0   
7     on_llm_end     NaN     2       1     1       0         0             0   
8     on_llm_end     NaN     2       1     1       0         0             0   
9     on_llm_end     NaN     2       1     1       0         0             0   
10    on_llm_end     NaN     2       1     1       0         0             0   
11    on_llm_end     NaN     2       1     1       0         0             0   
12  on_llm_start  OpenAI     3       2     1       0         0             0   
13  on_llm_start  OpenAI     3       2     1       0         0             0   
14  on_llm_start  OpenAI     3       2     1       0         0             0   
15  on_llm_start  OpenAI     3       2     1       0         0             0   
16  on_llm_start  OpenAI     3       2     1       0         0             0   
17  on_llm_start  OpenAI     3       2     1       0         0             0   
18    on_llm_end     NaN     4       2     2       0         0             0   
19    on_llm_end     NaN     4       2     2       0         0             0   
20    on_llm_end     NaN     4       2     2       0         0             0   
21    on_llm_end     NaN     4       2     2       0         0             0   
22    on_llm_end     NaN     4       2     2       0         0             0   
23    on_llm_end     NaN     4       2     2       0         0             0   
    chain_ends  llm_starts  ...  difficult_words  linsear_write_formula  \
0            0           1  ...              NaN                    NaN   
1            0           1  ...              NaN                    NaN   
2            0           1  ...              NaN                    NaN   
3            0           1  ...              NaN                    NaN   
4            0           1  ...              NaN                    NaN   
5            0           1  ...              NaN                    NaN   
6            0           1  ...              0.0                    5.5   
7            0           1  ...              2.0                    6.5   
8            0           1  ...              0.0                    5.5   
9            0           1  ...              2.0                    6.5   
10           0           1  ...              0.0                    5.5   
11           0           1  ...              2.0                    6.5   
12           0           2  ...              NaN                    NaN   
13           0           2  ...              NaN                    NaN   
14           0           2  ...              NaN                    NaN   
15           0           2  ...              NaN                    NaN   
16           0           2  ...              NaN                    NaN   
17           0           2  ...              NaN                    NaN   
18           0           2  ...              0.0                    5.5   
19           0           2  ...              2.0                    6.5   
20           0           2  ...              0.0                    5.5   
21           0           2  ...              2.0                    6.5   
22           0           2  ...              0.0                    5.5   
23           0           2  ...              2.0                    6.5   
    gunning_fog      text_standard  fernandez_huerta szigriszt_pazos  \
0          NaN                NaN               NaN             NaN   
1          NaN                NaN               NaN             NaN   
2          NaN                NaN               NaN             NaN   
3          NaN                NaN               NaN             NaN   
4          NaN                NaN               NaN             NaN   
5          NaN                NaN               NaN             NaN   
6         5.20  5th and 6th grade            133.58          131.54   
7         8.28  6th and 7th grade            115.58          112.37   
8         5.20  5th and 6th grade            133.58          131.54   
9         8.28  6th and 7th grade            115.58          112.37   
10        5.20  5th and 6th grade            133.58          131.54   
11        8.28  6th and 7th grade            115.58          112.37   
12         NaN                NaN               NaN             NaN   
13         NaN                NaN               NaN             NaN   
14         NaN                NaN               NaN             NaN   
15         NaN                NaN               NaN             NaN   
16         NaN                NaN               NaN             NaN   
17         NaN                NaN               NaN             NaN   
18        5.20  5th and 6th grade            133.58          131.54   
19        8.28  6th and 7th grade            115.58          112.37   
20        5.20  5th and 6th grade            133.58          131.54   
21        8.28  6th and 7th grade            115.58          112.37   
22        5.20  5th and 6th grade            133.58          131.54   
23        8.28  6th and 7th grade            115.58          112.37   
    gutierrez_polini  crawford  gulpease_index   osman  
0                NaN       NaN             NaN     NaN  
1                NaN       NaN             NaN     NaN  
2                NaN       NaN             NaN     NaN  
3                NaN       NaN             NaN     NaN  
4                NaN       NaN             NaN     NaN  
5                NaN       NaN             NaN     NaN  
6              62.30      -0.2            79.8  116.91  
7              54.83       1.4            72.1  100.17  
8              62.30      -0.2            79.8  116.91  
9              54.83       1.4            72.1  100.17  
10             62.30      -0.2            79.8  116.91  
11             54.83       1.4            72.1  100.17  
12               NaN       NaN             NaN     NaN  
13               NaN       NaN             NaN     NaN  
14               NaN       NaN             NaN     NaN  
15               NaN       NaN             NaN     NaN  
16               NaN       NaN             NaN     NaN  
17               NaN       NaN             NaN     NaN  
18             62.30      -0.2            79.8  116.91  
19             54.83       1.4            72.1  100.17  
20             62.30      -0.2            79.8  116.91  
21             54.83       1.4            72.1  100.17  
22             62.30      -0.2            79.8  116.91  
23             54.83       1.4            72.1  100.17  
[24 rows x 39 columns], 'session_analysis':    prompt_step         prompts    name  output_step  \
0            1  Tell me a joke  OpenAI            2   
1            1  Tell me a poem  OpenAI            2   
2            1  Tell me a joke  OpenAI            2   
3            1  Tell me a poem  OpenAI            2   
4            1  Tell me a joke  OpenAI            2   
5            1  Tell me a poem  OpenAI            2   
6            3  Tell me a joke  OpenAI            4   
7            3  Tell me a poem  OpenAI            4   
8            3  Tell me a joke  OpenAI            4   
9            3  Tell me a poem  OpenAI            4   
10           3  Tell me a joke  OpenAI            4   
11           3  Tell me a poem  OpenAI            4   
                                               output  \
0   \n\nQ: What did the fish say when it hit the w...   
1   \n\nRoses are red,\nViolets are blue,\nSugar i...   
2   \n\nQ: What did the fish say when it hit the w...   
3   \n\nRoses are red,\nViolets are blue,\nSugar i...   
4   \n\nQ: What did the fish say when it hit the w...   
5   \n\nRoses are red,\nViolets are blue,\nSugar i...   
6   \n\nQ: What did the fish say when it hit the w...   
7   \n\nRoses are red,\nViolets are blue,\nSugar i...   
8   \n\nQ: What did the fish say when it hit the w...   
9   \n\nRoses are red,\nViolets are blue,\nSugar i...   
10  \n\nQ: What did the fish say when it hit the w...   
11  \n\nRoses are red,\nViolets are blue,\nSugar i...   
    token_usage_total_tokens  token_usage_prompt_tokens  \
0                        162                         24   
1                        162                         24   
2                        162                         24   
3                        162                         24   
4                        162                         24   
5                        162                         24   
6                        162                         24   
7                        162                         24   
8                        162                         24   
9                        162                         24   
10                       162                         24   
11                       162                         24   
    token_usage_completion_tokens  flesch_reading_ease  flesch_kincaid_grade  \
0                             138               109.04                   1.3   
1                             138                83.66                   4.8   
2                             138               109.04                   1.3   
3                             138                83.66                   4.8   
4                             138               109.04                   1.3   
5                             138                83.66                   4.8   
6                             138               109.04                   1.3   
7                             138                83.66                   4.8   
8                             138               109.04                   1.3   
9                             138                83.66                   4.8   
10                            138               109.04                   1.3   
11                            138                83.66                   4.8   
    ...  difficult_words  linsear_write_formula  gunning_fog      text_standard  fernandez_huerta  szigriszt_pazos  gutierrez_polini  crawford  gulpease_index   osman  
0   ...                0                    5.5         5.20  5th and 6th grade            133.58           131.54             62.30      -0.2            79.8  116.91  
1   ...                2                    6.5         8.28  6th and 7th grade            115.58           112.37             54.83       1.4            72.1  100.17  
2   ...                0                    5.5         5.20  5th and 6th grade            133.58           131.54             62.30      -0.2            79.8  116.91  
3   ...                2                    6.5         8.28  6th and 7th grade            115.58           112.37             54.83       1.4            72.1  100.17  
4   ...                0                    5.5         5.20  5th and 6th grade            133.58           131.54             62.30      -0.2            79.8  116.91  
5   ...                2                    6.5         8.28  6th and 7th grade            115.58           112.37             54.83       1.4            72.1  100.17  
6   ...                0                    5.5         5.20  5th and 6th grade            133.58           131.54             62.30      -0.2            79.8  116.91  
7   ...                2                    6.5         8.28  6th and 7th grade            115.58           112.37             54.83       1.4            72.1  100.17  
8   ...                0                    5.5         5.20  5th and 6th grade            133.58           131.54             62.30      -0.2            79.8  116.91  
9   ...                2                    6.5         8.28  6th and 7th grade            115.58           112.37             54.83       1.4            72.1  100.17  
10  ...                0                    5.5         5.20  5th and 6th grade            133.58           131.54             62.30      -0.2            79.8  116.91  
11  ...                2                    6.5         8.28  6th and 7th grade            115.58           112.37             54.83       1.4            72.1  100.17  
[12 rows x 24 columns]}
2023-03-29 14:00:25,948 - clearml.Task - INFO - Completed model upload to https://files.clear.ml/langchain_callback_demo/llm.988bd727b0e94a29a3ac0ee526813545/models/simple_sequential
```

此时，您可以访问 https://app.clear.ml 并查看生成的 ClearML 任务。

在其他内容中，您会看到此笔记本与任何 git 信息一起保存。包含所使用参数的模型 JSON 保存为一个 artifact，还有控制台日志，在绘图部分，您会找到代表链路流程的表格。

最后，如果您启用了可视化功能，这些将以 HTML 文件的形式存储在调试样本下。

### 情景 2：使用工具创建代理

为了展示一个更高级的工作流程，让我们创建一个可以访问工具的代理。尽管 ClearML 跟踪结果的方式并没有不同，但由于与之前简单示例相比采取了其他类型的操作，表格看起来会略有不同。

现在您还可以看到使用 `finish=True` 关键字，这将完全关闭 ClearML 任务，而不仅仅是重置参数并提示进行新的对话。

```python
from langchain.agents import AgentType, initialize_agent, load_tools
# 情景 2 - 带工具的代理
tools = load_tools(["serpapi", "llm-math"], llm=llm, callbacks=callbacks)
agent = initialize_agent(
    tools,
    llm,
    agent=AgentType.ZERO_SHOT_REACT_DESCRIPTION,
    callbacks=callbacks,
)
agent.run("Who is the wife of the person who sang summer of 69?")
clearml_callback.flush_tracker(
    langchain_asset=agent, name="Agent with Tools", finish=True
)
```

```output
> 进入新的 AgentExecutor 链...
{'action': 'on_chain_start', 'name': 'AgentExecutor', 'step': 1, 'starts': 1, 'ends': 0, 'errors': 0, 'text_ctr': 0, 'chain_starts': 1, 'chain_ends': 0, 'llm_starts': 0, 'llm_ends': 0, 'llm_streams': 0, 'tool_starts': 0, 'tool_ends': 0, 'agent_ends': 0, 'input': 'Who is the wife of the person who sang summer of 69?'}
{'action': 'on_llm_start', 'name': 'OpenAI', 'step': 2, 'starts': 2, 'ends': 0, 'errors': 0, 'text_ctr': 0, 'chain_starts': 1, 'chain_ends': 0, 'llm_starts': 1, 'llm_ends': 0, 'llm_streams': 0, 'tool_starts': 0, 'tool_ends': 0, 'agent_ends': 0, 'prompts': 'Answer the following questions as best you can. You have access to the following tools:\n\nSearch: A search engine. Useful for when you need to answer questions about current events. Input should be a search query.\nCalculator: Useful for when you need to answer questions about math.\n\nUse the following format:\n\nQuestion: the input question you must answer\nThought: you should always think about what to do\nAction: the action to take, should be one of [Search, Calculator]\nAction Input: the input to the action\nObservation: the result of the action\n... (this Thought/Action/Action Input/Observation can repeat N times)\nThought: I now know the final answer\nFinal Answer: the final answer to the original input question\n\nBegin!\n\nQuestion: Who is the wife of the person who sang summer of 69?\nThought:'}
{'action': 'on_llm_end', 'token_usage_prompt_tokens': 189, 'token_usage_completion_tokens': 34, 'token_usage_total_tokens': 223, 'model_name': 'text-davinci-003', 'step': 3, 'starts': 2, 'ends': 1, 'errors': 0, 'text_ctr': 0, 'chain_starts': 1, 'chain_ends': 0, 'llm_starts': 1, 'llm_ends': 1, 'llm_streams': 0, 'tool_starts': 0, 'tool_ends': 0, 'agent_ends': 0, 'text': ' I need to find out who sang summer of 69 and then find out who their wife is.\nAction: Search\nAction Input: "Who sang summer of 69"', 'generation_info_finish_reason': 'stop', 'generation_info_logprobs': None, 'flesch_reading_ease': 91.61, 'flesch_kincaid_grade': 3.8, 'smog_index': 0.0, 'coleman_liau_index': 3.41, 'automated_readability_index': 3.5, 'dale_chall_readability_score': 6.06, 'difficult_words': 2, 'linsear_write_formula': 5.75, 'gunning_fog': 5.4, 'text_standard': '3rd and 4th grade', 'fernandez_huerta': 121.07, 'szigriszt_pazos': 119.5, 'gutierrez_polini': 54.91, 'crawford': 0.9, 'gulpease_index': 72.7, 'osman': 92.16}
 I need to find out who sang summer of 69 and then find out who their wife is.
Action: Search
Action Input: "Who sang summer of 69"{'action': 'on_agent_action', 'tool': 'Search', 'tool_input': 'Who sang summer of 69', 'log': ' I need to find out who sang summer of 69 and then find out who their wife is.\nAction: Search\nAction Input: "Who sang summer of 69"', 'step': 4, 'starts': 3, 'ends': 1, 'errors': 0, 'text_ctr': 0, 'chain_starts': 1, 'chain_ends': 0, 'llm_starts': 1, 'llm_ends': 1, 'llm_streams': 0, 'tool_starts': 1, 'tool_ends': 0, 'agent_ends': 0}
{'action': 'on_tool_start', 'input_str': 'Who sang summer of 69', 'name': 'Search', 'description': 'A search engine. Useful for when you need to answer questions about current events. Input should be a search query.', 'step': 5, 'starts': 4, 'ends': 1, 'errors': 0, 'text_ctr': 0, 'chain_starts': 1, 'chain_ends': 0, 'llm_starts': 1, 'llm_ends': 1, 'llm_streams': 0, 'tool_starts': 2, 'tool_ends': 0, 'agent_ends': 0}
```

观察: Bryan Adams - Summer Of 69 (官方音乐视频)。

思考:{'action': 'on_tool_end', 'output': 'Bryan Adams - Summer Of 69 (官方音乐视频)', 'step': 6, 'starts': 4, 'ends': 2, 'errors': 0, 'text_ctr': 0, 'chain_starts': 1, 'chain_ends': 0, 'llm_starts': 1, 'llm_ends': 1, 'llm_streams': 0, 'tool_starts': 2, 'tool_ends': 1, 'agent_ends': 0}

{'action': 'on_llm_start', 'name': 'OpenAI', 'step': 7, 'starts': 5, 'ends': 2, 'errors': 0, 'text_ctr': 0, 'chain_starts': 1, 'chain_ends': 0, 'llm_starts': 2, 'llm_ends': 1, 'llm_streams': 0, 'tool_starts': 2, 'tool_ends': 1, 'agent_ends': 0, 'prompts': '尽力回答以下问题。您可以使用以下工具：\n\n搜索：搜索引擎。在需要回答有关当前事件的问题时很有用。输入应该是一个搜索查询。\n计算器：在需要回答数学问题时很有用。\n\n使用以下格式：\n\n问题：您必须回答的问题\n思考：您应该考虑要做什么\n行动：采取的行动，应该是[搜索，计算器]中的一个\n行动输入：行动的输入\n观察：行动的结果\n... (这个思考/行动/行动输入/观察可以重复N次)\n思考：我现在知道最终答案\n最终答案：原始输入问题的最终答案\n\n开始！\n\n问题：谁是演唱 Summer of 69 的人的妻子？\n思考：我需要找出谁演唱了 Summer of 69，然后找出他们的妻子是谁。\n行动：搜索\n行动输入："谁演唱了 Summer of 69"\n观察：Bryan Adams - Summer Of 69 (官方音乐视频)。\n思考:'}

{'action': 'on_llm_end', 'token_usage_prompt_tokens': 242, 'token_usage_completion_tokens': 28, 'token_usage_total_tokens': 270, 'model_name': 'text-davinci-003', 'step': 8, 'starts': 5, 'ends': 3, 'errors': 0, 'text_ctr': 0, 'chain_starts': 1, 'chain_ends': 0, 'llm_starts': 2, 'llm_ends': 2, 'llm_streams': 0, 'tool_starts': 2, 'tool_ends': 1, 'agent_ends': 0, 'text': '我需要找出 Bryan Adams 结婚了没有。\n行动：搜索\n行动输入："Bryan Adams 结婚了吗"', 'generation_info_finish_reason': 'stop', 'generation_info_logprobs': None, 'flesch_reading_ease': 94.66, 'flesch_kincaid_grade': 2.7, 'smog_index': 0.0, 'coleman_liau_index': 4.73, 'automated_readability_index': 4.0, 'dale_chall_readability_score': 7.16, 'difficult_words': 2, 'linsear_write_formula': 4.25, 'gunning_fog': 4.2, 'text_standard': '4th and 5th grade', 'fernandez_huerta': 124.13, 'szigriszt_pazos': 119.2, 'gutierrez_polini': 52.26, 'crawford': 0.7, 'gulpease_index': 74.7, 'osman': 84.2}

我需要找出 Bryan Adams 结婚了没有。

行动：搜索

行动输入："Bryan Adams 结婚了吗"{'action': 'on_agent_action', 'tool': 'Search', 'tool_input': 'Bryan Adams 结婚了吗', 'log': '我需要找出 Bryan Adams 结婚了没有。\n行动：搜索\n行动输入："Bryan Adams 结婚了吗"', 'step': 9, 'starts': 6, 'ends': 3, 'errors': 0, 'text_ctr': 0, 'chain_starts': 1, 'chain_ends': 0, 'llm_starts': 2, 'llm_ends': 2, 'llm_streams': 0, 'tool_starts': 3, 'tool_ends': 1, 'agent_ends': 0}

{'action': 'on_tool_start', 'input_str': 'Bryan Adams 结婚了吗', 'name': '搜索', 'description': '搜索引擎。在需要回答有关当前事件的问题时很有用。输入应该是一个搜索查询。', 'step': 10, 'starts': 7, 'ends': 3, 'errors': 0, 'text_ctr': 0, 'chain_starts': 1, 'chain_ends': 0, 'llm_starts': 2, 'llm_ends': 2, 'llm_streams': 0, 'tool_starts': 4, 'tool_ends': 1, 'agent_ends': 0}

观察: Bryan Adams 从未结婚。在1990年代，他与丹麦模特 Cecilie Thomsen 有过一段感情。2011年，Bryan 和 Alicia Grimaldi，他...

思考:{'action': 'on_tool_end', 'output': 'Bryan Adams 从未结婚。在1990年代，他与丹麦模特 Cecilie Thomsen 有过一段感情。2011年，Bryan 和 Alicia Grimaldi，他...', 'step': 11, 'starts': 7, 'ends': 4, 'errors': 0, 'text_ctr': 0, 'chain_starts': 1, 'chain_ends': 0, 'llm_starts': 2, 'llm_ends': 2, 'llm_streams': 0, 'tool_starts': 4, 'tool_ends': 2, 'agent_ends': 0}

### OpenAI 自然语言处理模型完成了一个任务

在这个任务中，OpenAI 的自然语言处理模型开始了一个包含多个步骤的流程。在第 12 步，模型开始了一个新的任务，其中包括 8 个起始步骤和 4 个结束步骤。在这个过程中，模型没有出现错误，也没有生成文本内容。

在这个任务中，模型被要求回答以下问题，并可以使用搜索引擎和计算器等工具来辅助回答问题。模型首先被问到：“谁是演唱《夏日 69》的人的妻子？”模型通过搜索发现是 Bryan Adams，并继续搜索得知 Bryan Adams 从未结过婚。最终，模型得出结论：Bryan Adams 从未结过婚。

这个任务的最终答案是：Bryan Adams 从未结过婚。

### 任务记录

- 模型名称：text-davinci-003

- 完成步骤：13

- 总起始步骤数：8

- 总结束步骤数：5

- 错误数：0

- 文本生成次数：0

- 链条起始次数：1

- 链条结束次数：0

- LLM（大型语言模型）起始次数：3

- LLM（大型语言模型）结束次数：3

- 工具起始次数：4

- 工具结束次数：2

- 代理结束次数：0

### 最终答案

Bryan Adams 从未结过婚。

> 完成链条。

```markdown
69             1           0           3  ...             NaN    NaN    NaN   
70             1           1           3  ...             NaN    NaN    NaN   
    tool  tool_input                                                log  \
0    NaN         NaN                                                NaN   
1    NaN         NaN                                                NaN   
2    NaN         NaN                                                NaN   
3    NaN         NaN                                                NaN   
4    NaN         NaN                                                NaN   
..   ...         ...                                                ...   
66   NaN         NaN                                                NaN   
67   NaN         NaN                                                NaN   
68   NaN         NaN                                                NaN   
69   NaN         NaN   现在我知道最终答案。\n最终答案：Bryan Adams从未结过婚。   
70   NaN         NaN                                                NaN   
    input_str  description                                             output  \
0         NaN          NaN                                                NaN   
1         NaN          NaN                                                NaN   
2         NaN          NaN                                                NaN   
3         NaN          NaN                                                NaN   
4         NaN          NaN                                                NaN   
..        ...          ...                                                ...   
66        NaN          NaN  Bryan Adams从未结过婚。在1990年代，他与维多利亚的秘密模特儿Cecil...   
67        NaN          NaN                                                NaN   
68        NaN          NaN                                                NaN   
69        NaN          NaN                Bryan Adams从未结过婚。   
70        NaN          NaN                                                NaN   
                                outputs  
0                                   NaN  
1                                   NaN  
2                                   NaN  
3                                   NaN  
4                                   NaN  
..                                  ...  
66                                  NaN  
67                                  NaN  
68                                  NaN  
69                                  NaN  
70  Bryan Adams从未结过婚。  
[71 rows x 47 columns], 'session_analysis':    prompt_step                                            prompts    name  \
0            2  尽力回答以下问题...  OpenAI   
1            7  尽力回答以下问题...  OpenAI   
2           12  尽力回答以下问题...  OpenAI   
   output_step                                             output  \
0            3   我需要找出谁唱了《69的夏天》以及...   
1            8   我需要找出Bryan Adams是否结婚...   
2           13   现在我知道最终答案。\n最终答案：Bryan Adams从未结过婚。   
   token_usage_total_tokens  token_usage_prompt_tokens  \
0                       223                        189   
1                       270                        242   
2                       332                        314   
   token_usage_completion_tokens  flesch_reading_ease  flesch_kincaid_grade  \
0                             34                91.61                   3.8   
1                             28                94.66                   2.7   
2                             18                81.29                   3.7   
   ...  difficult_words  linsear_write_formula  gunning_fog  \
0  ...                2                   5.75          5.4   
1  ...                2                   4.25          4.2   
2  ...                1                   2.50          2.8   
       text_standard  fernandez_huerta  szigriszt_pazos  gutierrez_polini  \
0  3rd and 4th grade            121.07           119.50             54.91   
1  4th and 5th grade            124.13           119.20             52.26   
2  3rd and 4th grade            115.70           110.84             49.79   
  crawford  gulpease_index  osman  
0      0.9            72.7  92.16  
1      0.7            74.7  84.20  
2      0.7            85.4  83.14  
[3 rows x 24 columns]}
``````output
无法更新任务 988bd727b0e94a29a3ac0ee526813545 中最后创建的模型，任务状态“已完成”无法被更新
### 提示和下一步操作
- 确保始终为 `clearml_callback.flush_tracker` 函数使用唯一的 `name` 参数。如果没有，用于运行的模型参数将覆盖先前的运行！
- 如果使用 `clearml_callback.flush_tracker(..., finish=True)` 关闭 ClearML 回调，则无法再使用该回调。如果要继续记录，请创建一个新的回调。
- 查看 ClearML 生态系统的其他部分，其中包括数据版本管理器、远程执行代理、自动化流水线等等！
```