# URL

本示例介绍了如何从一系列URL中加载`HTML`文档，并将其转换为我们可以在下游使用的`Document`格式。

## 无结构URL加载器

您需要安装`unstructured`库：

```python
!pip install -U unstructured
```

```python
from langchain_community.document_loaders import UnstructuredURLLoader
```

```python
urls = [
    "https://www.understandingwar.org/backgrounder/russian-offensive-campaign-assessment-february-8-2023",
    "https://www.understandingwar.org/backgrounder/russian-offensive-campaign-assessment-february-9-2023",
]
```

在headers=headers中传入ssl_verify=False以解决ssl_verification错误。

```python
loader = UnstructuredURLLoader(urls=urls)
```

```python
data = loader.load()
```

## Selenium URL加载器

这部分介绍了如何使用`SeleniumURLLoader`从URL列表中加载HTML文档。

使用`Selenium`可以加载需要JavaScript渲染的页面。

要使用`SeleniumURLLoader`，您需要安装`selenium`和`unstructured`。

```python
!pip install -U selenium unstructured
```

```python
from langchain_community.document_loaders import SeleniumURLLoader
```

```python
urls = [
    "https://www.youtube.com/watch?v=dQw4w9WgXcQ",
    "https://goo.gl/maps/NDSHwePEyaHMFGwh8",
]
```

```python
loader = SeleniumURLLoader(urls=urls)
```

```python
data = loader.load()
```

## Playwright URL加载器

这部分介绍了如何使用`PlaywrightURLLoader`从URL列表中加载HTML文档。

[Playwright](https://playwright.dev/)为现代Web应用程序提供可靠的端到端测试。

与Selenium的情况类似，`Playwright`允许我们加载和渲染JavaScript页面。

要使用`PlaywrightURLLoader`，您需要安装`playwright`和`unstructured`。此外，您还需要安装`Playwright Chromium`浏览器：

```python
!pip install -U playwright unstructured
```

```python
!playwright install
```

```python
from langchain_community.document_loaders import PlaywrightURLLoader
```

```python
urls = [
    "https://www.youtube.com/watch?v=dQw4w9WgXcQ",
    "https://goo.gl/maps/NDSHwePEyaHMFGwh8",
]
```

```python
loader = PlaywrightURLLoader(urls=urls, remove_selectors=["header", "footer"])
```

```python
data = loader.load()
```