# 字幕文件格式

[SubRip 文件格式](https://en.wikipedia.org/wiki/SubRip#SubRip_file_format) 在 `Matroska` 多媒体容器格式的网站上被描述为“也许是所有字幕格式中最基本的一个。” `SubRip (SubRip Text)` 文件的扩展名为 `.srt`，其中包含由空行分隔的分组中的格式化纯文本行。字幕按顺序编号，从 1 开始。使用的时间码格式为小时:分钟:秒,毫秒，时间单位固定为两个零填充数字，分数固定为三个零填充数字 (00:00:00,000)。使用逗号作为小数分隔符，因为该程序是在法国编写的。

如何从字幕（`.srt`）文件加载数据

请从[这里](https://www.opensubtitles.org/en/subtitles/5575150/star-wars-the-clone-wars-crisis-at-the-heart-en)下载[示例 .srt 文件](https://www.opensubtitles.org/en/subtitles/5575150/star-wars-the-clone-wars-crisis-at-the-heart-en)。

```python
%pip install --upgrade --quiet  pysrt
```

```python
from langchain_community.document_loaders import SRTLoader
```

```python
loader = SRTLoader(
    "example_data/Star_Wars_The_Clone_Wars_S06E07_Crisis_at_the_Heart.srt"
)
```

```python
docs = loader.load()
```

```python
docs[0].page_content[:100]
```

```output
'<i>Corruption discovered\nat the core of the Banking Clan!</i> <i>Reunited, Rush Clovis\nand Senator A'
```