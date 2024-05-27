# 谷歌地点

本文介绍如何使用谷歌地点 API。

```python
%pip install --upgrade --quiet  googlemaps
```

```python
import os
os.environ["GPLACES_API_KEY"] = ""
```

```python
from langchain_community.tools import GooglePlacesTool
```

```python
places = GooglePlacesTool()
```

```python
places.run("al fornos")
```

```output
"1. Delfina Restaurant\n地址：3621 18th St, San Francisco, CA 94110, 美国\n电话：(415) 552-4055\n网站：https://www.delfinasf.com/\n\n\n2. Piccolo Forno\n地址：725 Columbus Ave, San Francisco, CA 94133, 美国\n电话：(415) 757-0087\n网站：https://piccolo-forno-sf.com/\n\n\n3. L'Osteria del Forno\n地址：519 Columbus Ave, San Francisco, CA 94133, 美国\n电话：(415) 982-1124\n网站：未知\n\n\n4. Il Fornaio\n地址：1265 Battery St, San Francisco, CA 94111, 美国\n电话：(415) 986-0100\n网站：https://www.ilfornaio.com/\n\n"
```