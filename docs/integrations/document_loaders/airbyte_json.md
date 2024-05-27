# Airbyte JSON（已弃用）

注意：`AirbyteJSONLoader`已被弃用。请改用[`AirbyteLoader`](/docs/integrations/document_loaders/airbyte)。

>[Airbyte](https://github.com/airbytehq/airbyte)是一个用于从API、数据库和文件到数据仓库和数据湖的ELT管道的数据集成平台。它拥有最大的ELT连接器目录，可连接到数据仓库和数据库。

本文介绍如何将Airbyte中的任何数据源加载到本地的JSON文件中，以便作为文档读取。

先决条件：

已安装 Docker Desktop

步骤：

1) 从 GitHub 克隆 Airbyte - `git clone https://github.com/airbytehq/airbyte.git`

2) 切换到 Airbyte 目录 - `cd airbyte`

3) 启动 Airbyte - `docker compose up`

4) 在浏览器中，访问 http://localhost:8000。系统会要求输入用户名和密码。默认情况下，用户名为 `airbyte`，密码为 `password`。

5) 设置任何您希望的数据源。

6) 将目的地设置为本地 JSON，指定目标路径 - 假设为 `/json_data`。设置手动同步。

7) 运行连接。

7) 要查看创建了哪些文件，可以导航到：`file:///tmp/airbyte_local`

8) 找到您的数据并复制路径。该路径应保存在下面的文件变量中。它应以 `/tmp/airbyte_local` 开头。

```python
from langchain_community.document_loaders import AirbyteJSONLoader
```

```python
!ls /tmp/airbyte_local/json_data/
```

```output
_airbyte_raw_pokemon.jsonl
```

```python
loader = AirbyteJSONLoader("/tmp/airbyte_local/json_data/_airbyte_raw_pokemon.jsonl")
```

```python
data = loader.load()
```

```python
print(data[0].page_content[:500])
```

```output
abilities: 
ability: 
name: blaze
url: https://pokeapi.co/api/v2/ability/66/
is_hidden: False
slot: 1
ability: 
name: solar-power
url: https://pokeapi.co/api/v2/ability/94/
is_hidden: True
slot: 3
base_experience: 267
forms: 
name: charizard
url: https://pokeapi.co/api/v2/pokemon-form/6/
game_indices: 
game_index: 180
version: 
name: red
url: https://pokeapi.co/api/v2/version/1/
game_index: 180
version: 
name: blue
url: https://pokeapi.co/api/v2/version/2/
game_index: 180
version: 
n
```