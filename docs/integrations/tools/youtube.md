# YouTube

>[YouTube Search](https://github.com/joetats/youtube_search) 包可以搜索 `YouTube` 视频，而无需使用它们受到严格限制的 API。

>

>它利用 `YouTube` 主页上的表单，并对生成的页面进行抓取。

这个笔记本展示了如何使用一个工具来搜索 YouTube。

改编自 [https://github.com/venuv/langchain_yt_tools](https://github.com/venuv/langchain_yt_tools)

```python
%pip install --upgrade --quiet  youtube_search
```

```python
from langchain_community.tools import YouTubeSearchTool
```

```python
tool = YouTubeSearchTool()
```

```python
tool.run("lex friedman")
```

```output
"['/watch?v=VcVfceTsD0A&pp=ygUMbGV4IGZyaWVkbWFu', '/watch?v=gPfriiHBBek&pp=ygUMbGV4IGZyaWVkbWFu']"
```

你也可以指定返回的结果数量

```python
tool.run("lex friedman,5")
```

```output
"['/watch?v=VcVfceTsD0A&pp=ygUMbGV4IGZyaWVkbWFu', '/watch?v=YVJ8gTnDC4Y&pp=ygUMbGV4IGZyaWVkbWFu', '/watch?v=Udh22kuLebg&pp=ygUMbGV4IGZyaWVkbWFu', '/watch?v=gPfriiHBBek&pp=ygUMbGV4IGZyaWVkbWFu', '/watch?v=L_Guz73e6fw&pp=ygUMbGV4IGZyaWVkbWFu']"
```