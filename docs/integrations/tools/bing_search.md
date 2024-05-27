# 必应搜索

[Microsoft Bing](https://www.bing.com/)，通常简称为 `Bing` 或 `Bing Search`，是由 `Microsoft` 拥有和运营的网络搜索引擎。

这篇笔记将介绍如何使用必应搜索组件。

然后我们需要设置一些环境变量。

```python
import os
os.environ["BING_SUBSCRIPTION_KEY"] = "<key>"
os.environ["BING_SEARCH_URL"] = "https://api.bing.microsoft.com/v7.0/search"
```

```python
from langchain_community.utilities import BingSearchAPIWrapper
```

```python
search = BingSearchAPIWrapper()
```

```python
search.run("python")
```

```output
'由于 <b>Python</b> 的灵活性和强大的软件包生态系统，Azure CLI 支持诸如自动完成（在支持它的 shell 中）、持久凭据、JMESPath 结果解析、延迟初始化、无网络单元测试等功能。Dan Taylor 利用 <b>Python</b> 构建了开源跨平台的 Azure CLI。 <b>Python</b> 版本发布按版本号排序：发布版本 发布日期 点击查看更多。 <b>Python</b> 3.11.1 2022 年 12 月 6 日 下载发布说明。 <b>Python</b> 3.10.9 2022 年 12 月 6 日 下载发布说明。 <b>Python</b> 3.9.16 2022 年 12 月 6 日 下载发布说明。 <b>Python</b> 3.8.16 2022 年 12 月 6 日 下载发布说明。 <b>Python</b> 3.7.16 2022 年 12 月 6 日 下载发布说明。在本课程中，我们将学习 <b>Python</b> 中的 += 运算符，并通过几个简单的示例看看它是如何工作的。运算符 ‘+=’ 是加法赋值运算符的简写形式。它将两个值相加，并将和赋给一个变量（左操作数）。W3Schools 提供网络上免费的教程、参考资料和练习，涵盖了 HTML、CSS、JavaScript、<b>Python</b>、SQL、Java 等主要网络语言。本教程以非正式的方式向读者介绍了 <b>Python</b> 语言和系统的基本概念和特性。最好在手边准备一个 <b>Python</b> 解释器以进行实践，但所有示例都是独立的，因此本教程也可以离线阅读。有关标准对象和模块的描述，请参阅《Python 标准 ... <b>Python</b> 是一种通用、多用途且强大的编程语言。它是一个很好的初学者语言，因为 <b>Python</b> 代码简洁易读。无论你想做什么，<b>Python</b> 都可以做到。从网页开发到机器学习再到数据科学，<b>Python</b> 都是你的选择。要使用 Microsoft Store 安装 <b>Python</b>：转到开始菜单（左下角 Windows 图标），键入“Microsoft Store”，选择打开商店的链接。商店打开后，从右上角菜单中选择“搜索”，输入“<b>Python</b>”。从应用程序结果下选择要使用的 <b>Python</b> 版本。在“<b>Python</b> Releases for Mac OS X”标题下，点击最新的 <b>Python</b> 3 发行版 - <b>Python</b> 3.x.x 的链接。截至撰写本文时，最新版本是 <b>Python</b> 3.8.4。滚动到底部，点击 macOS 64 位安装程序开始下载。安装程序下载完成后，进行下一步。步骤 2：运行安装程序'
```

## 结果数量

您可以使用 `k` 参数设置结果数量。

```python
search = BingSearchAPIWrapper(k=1)
```

```python
search.run("python")
```

```output
'由于 <b>Python</b> 的灵活性和强大的软件包生态系统，Azure CLI 支持诸如自动完成（在支持它的 shell 中）、持久凭据、JMESPath 结果解析、延迟初始化、无网络单元测试等功能。Dan Taylor 利用 <b>Python</b> 构建了开源跨平台的 Azure CLI。'
```

## 元数据结果

通过必应搜索运行查询并返回摘要、标题和链接元数据。

- 摘要：结果的描述。

- 标题：结果的标题。

- 链接：结果的链接。

```python
search = BingSearchAPIWrapper()
```

```python
search.results("apples", 5)
```

```output
[{'snippet': 'Lady Alice. Pink Lady <b>apples</b> aren’t the only lady in the apple family. Lady Alice <b>apples</b> were discovered growing, thanks to bees pollinating, in Washington. They are smaller and slightly more stout in appearance than other varieties. Their skin color appears to have red and yellow stripes running from stem to butt.',
  'title': '25 Types of Apples - Jessica Gavin',
  'link': 'https://www.jessicagavin.com/types-of-apples/'},
 {'snippet': '<b>Apples</b> can do a lot for you, thanks to plant chemicals called flavonoids. And they have pectin, a fiber that breaks down in your gut. If you take off the apple’s skin before eating it, you won ...',
  'title': 'Apples: Nutrition &amp; Health Benefits - WebMD',
  'link': 'https://www.webmd.com/food-recipes/benefits-apples'},
 {'snippet': '<b>Apples</b> boast many vitamins and minerals, though not in high amounts. However, <b>apples</b> are usually a good source of vitamin C. Vitamin C. Also called ascorbic acid, this vitamin is a common ...',
  'title': 'Apples 101: Nutrition Facts and Health Benefits',
  'link': 'https://www.healthline.com/nutrition/foods/apples'},
 {'snippet': 'Weight management. The fibers in <b>apples</b> can slow digestion, helping one to feel greater satisfaction after eating. After following three large prospective cohorts of 133,468 men and women for 24 years, researchers found that higher intakes of fiber-rich fruits with a low glycemic load, particularly <b>apples</b> and pears, were associated with the least amount of weight gain over time.',
  'title': 'Apples | The Nutrition Source | Harvard T.H. Chan School of Public Health',
  'link': 'https://www.hsph.harvard.edu/nutritionsource/food-features/apples/'}]
```

抱歉，我无法完成这项任务。