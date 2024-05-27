# HTML转文本

>[html2text](https://github.com/Alir3z4/html2text/) 是一个Python包，可以将`HTML`页面转换为干净、易读的普通`ASCII文本`。

这些ASCII文本也恰好是有效的`Markdown`（一种文本到HTML的格式）。

```python
%pip install --upgrade --quiet  html2text
```

```python
from langchain_community.document_loaders import AsyncHtmlLoader
urls = ["https://www.espn.com", "https://lilianweng.github.io/posts/2023-06-23-agent/"]
loader = AsyncHtmlLoader(urls)
docs = loader.load()
```

```output
Fetching pages: 100%|############| 2/2 [00:00<00:00, 10.75it/s]
```

```python
from langchain_community.document_transformers import Html2TextTransformer
```

```python
urls = ["https://www.espn.com", "https://lilianweng.github.io/posts/2023-06-23-agent/"]
html2text = Html2TextTransformer()
docs_transformed = html2text.transform_documents(docs)
```

```python
docs_transformed[0].page_content[1000:2000]
```

```output
"  * ESPNFC\n\n  * X Games\n\n  * SEC Network\n\n## ESPN Apps\n\n  * ESPN\n\n  * ESPN Fantasy\n\n## Follow ESPN\n\n  * Facebook\n\n  * Twitter\n\n  * Instagram\n\n  * Snapchat\n\n  * YouTube\n\n  * The ESPN Daily Podcast\n\n2023 FIFA Women's World Cup\n\n## Follow live: Canada takes on Nigeria in group stage of Women's World Cup\n\n2m\n\nEPA/Morgan Hancock\n\n## TOP HEADLINES\n\n  * Snyder fined $60M over findings in investigation\n  * NFL owners approve $6.05B sale of Commanders\n  * Jags assistant comes out as gay in NFL milestone\n  * O's alone atop East after topping slumping Rays\n  * ACC's Phillips: Never condoned hazing at NU\n\n  * Vikings WR Addison cited for driving 140 mph\n  * 'Taking his time': Patient QB Rodgers wows Jets\n  * Reyna got U.S. assurances after Berhalter rehire\n  * NFL Future Power Rankings\n\n## USWNT AT THE WORLD CUP\n\n### USA VS. VIETNAM: 9 P.M. ET FRIDAY\n\n## How do you defend against Alex Morgan? Former opponents sound off\n\nThe U.S. forward is unstoppable at this level, scoring 121 goals and adding 49"
```

```python
docs_transformed[1].page_content[1000:2000]
```

```output
"t's brain,\ncomplemented by several key components:\n\n  * **Planning**\n    * Subgoal and decomposition: The agent breaks down large tasks into smaller, manageable subgoals, enabling efficient handling of complex tasks.\n    * Reflection and refinement: The agent can do self-criticism and self-reflection over past actions, learn from mistakes and refine them for future steps, thereby improving the quality of final results.\n  * **Memory**\n    * Short-term memory: I would consider all the in-context learning (See Prompt Engineering) as utilizing short-term memory of the model to learn.\n    * Long-term memory: This provides the agent with the capability to retain and recall (infinite) information over extended periods, often by leveraging an external vector store and fast retrieval.\n  * **Tool use**\n    * The agent learns to call external APIs for extra information that is missing from the model weights (often hard to change after pre-training), including current information, code execution c"
```