# E2B数据分析

[E2B的云环境](https://e2b.dev)是LLM的优秀运行沙盒。

E2B的数据分析沙盒允许在沙盒环境中安全地执行代码。这非常适合构建诸如代码解释器或类似ChatGPT中的高级数据分析工具。

E2B数据分析沙盒允许您：

- 运行Python代码

- 通过matplotlib生成图表

- 在运行时动态安装Python包

- 在运行时动态安装系统包

- 运行shell命令

- 上传和下载文件

我们将创建一个简单的OpenAI代理，使用E2B的数据分析沙盒来使用Python对上传的文件执行分析。

获取您的OpenAI API密钥和[E2B API密钥在这里](https://e2b.dev/docs/getting-started/api-key)，并将它们设置为环境变量。

您可以在[这里找到完整的API文档](https://e2b.dev/docs)。

您需要安装`e2b`才能开始：

```python
%pip install --upgrade --quiet  langchain e2b
```

```python
from langchain_community.tools import E2BDataAnalysisTool
```

```python
import os
from langchain.agents import AgentType, initialize_agent
from langchain_openai import ChatOpenAI
os.environ["E2B_API_KEY"] = "<E2B_API_KEY>"
os.environ["OPENAI_API_KEY"] = "<OPENAI_API_KEY>"
```

当创建`E2BDataAnalysisTool`实例时，您可以传递回调函数来监听沙盒输出。例如，在与LLM的流式输出结合时，这非常有用，特别是在创建更具响应性的UI时。

```python
# 生成的图表是matplotlib创建的图表，当调用`plt.show()`时会生成
def save_artifact(artifact):
    print("生成了新的matplotlib图表：", artifact.name)
    # 将图表下载为`bytes`，并由用户自行决定如何显示（例如在前端显示）
    file = artifact.download()
    basename = os.path.basename(artifact.name)
    # 将图表保存到`charts`目录
    with open(f"./charts/{basename}", "wb") as f:
        f.write(file)
e2b_data_analysis_tool = E2BDataAnalysisTool(
    # 将环境变量传递给沙盒
    env_vars={"MY_SECRET": "secret_value"},
    on_stdout=lambda stdout: print("stdout:", stdout),
    on_stderr=lambda stderr: print("stderr:", stderr),
    on_artifact=save_artifact,
)
```

上传一个示例CSV数据文件到沙盒，以便我们可以使用代理对其进行分析。您可以使用例如[此文件](https://storage.googleapis.com/e2b-examples/netflix.csv)关于Netflix电视节目。

```python
with open("./netflix.csv") as f:
    remote_path = e2b_data_analysis_tool.upload_file(
        file=f,
        description="包含Netflix电视节目的数据，包括标题、类别、导演、发布日期、演员阵容、年龄评级等信息。",
    )
    print(remote_path)
```

```output
name='netflix.csv' remote_path='/home/user/netflix.csv' description='包含Netflix电视节目的数据，包括标题、类别、导演、发布日期、演员阵容、年龄评级等信息。'
```

创建一个`Tool`对象并初始化Langchain代理。

```python
tools = [e2b_data_analysis_tool.as_tool()]
llm = ChatOpenAI(model="gpt-4", temperature=0)
agent = initialize_agent(
    tools,
    llm,
    agent=AgentType.OPENAI_FUNCTIONS,
    verbose=True,
    handle_parsing_errors=True,
)
```

现在我们可以向代理询问关于之前上传的CSV文件的问题。

```python
agent.run(
    "Netflix上在2000年至2010年间发布的5部最长电影是哪些？创建一个显示它们时长的图表。"
)
```

```output
> 进入新的AgentExecutor链...
调用：`e2b_data_analysis`，参数为`{'python_code': "import pandas as pd\n\n# 加载数据\nnetflix_data = pd.read_csv('/home/user/netflix.csv')\n\n# 将'release_year'列转换为整数\nnetflix_data['release_year'] = netflix_data['release_year'].astype(int)\n\n# 为2000年至2010年间发布的电影筛选数据\nfiltered_data = netflix_data[(netflix_data['release_year'] >= 2000) & (netflix_data['release_year'] <= 2010) & (netflix_data['type'] == 'Movie')]\n\n# 删除'duration'不可用的行\nfiltered_data = filtered_data[filtered_data['duration'].notna()]\n\n# 将'duration'列转换为整数\nfiltered_data['duration'] = filtered_data['duration'].str.replace(' min','').astype(int)\n\n# 获取前5部最长电影\nlongest_movies = filtered_data.nlargest(5, 'duration')\n\n# 创建一个条形图\nimport matplotlib.pyplot as plt\n\nplt.figure(figsize=(10,5))\nplt.barh(longest_movies['title'], longest_movies['duration'], color='skyblue')\nplt.xlabel('时长（分钟）')\nplt.title('Netflix上最长的5部电影（2000-2010年）')\nplt.gca().invert_yaxis()\nplt.savefig('/home/user/longest_movies.png')\n\nlongest_movies[['title', 'duration']]"}`
stdout:                              title  duration
stdout: 1019                        Lagaan       224
stdout: 4573                  Jodhaa Akbar       214
stdout: 2731      Kabhi Khushi Kabhie Gham       209
stdout: 2632  No Direction Home: Bob Dylan       208
stdout: 2126          What's Your Raashee?       203
{'stdout': "                             title  duration\n1019                        Lagaan       224\n4573                  Jodhaa Akbar       214\n2731      Kabhi Khushi Kabhie Gham       209\n2632  No Direction Home: Bob Dylan       208\n2126          What's Your Raashee?       203", 'stderr': ''}
Netflix上在2000年至2010年间发布的5部最长电影是：
1. Lagaan - 224分钟
2. Jodhaa Akbar - 214分钟
3. Kabhi Khushi Kabhie Gham - 209分钟
4. No Direction Home: Bob Dylan - 208分钟
5. What's Your Raashee? - 203分钟
以下是显示它们时长的图表：
![最长电影](sandbox:/home/user/longest_movies.png)
> 完成链。
Netflix上发布的2000年至2010年间最长的5部电影分别是：
1. 《甘地传》- 224分钟
2. 《荣耀之战》- 214分钟
3. 《家族的荣耀》- 209分钟
4. 《无所适从：鲍勃·迪伦》- 208分钟
5. 《你的星座是什么？》- 203分钟
以下是显示它们时长的图表：
![Longest Movies](sandbox:/home/user/longest_movies.png)
此外，您还可以像这样从沙盒中下载任何文件：
```python
# 路径是沙盒中的远程路径
files_in_bytes = e2b_data_analysis_tool.download_file("/home/user/netflix.csv")
```
最后，您可以通过`run_command`在沙盒内运行任何shell命令。
```python
# 安装SQLite
e2b_data_analysis_tool.run_command("sudo apt update")
e2b_data_analysis_tool.install_system_packages("sqlite3")
# 检查SQLite版本
output = e2b_data_analysis_tool.run_command("sqlite3 --version")
print("version: ", output["stdout"])
print("error: ", output["stderr"])
print("exit code: ", output["exit_code"])
```
当您的代理完成时，请不要忘记关闭沙盒。
```python
e2b_data_analysis_tool.close()
```