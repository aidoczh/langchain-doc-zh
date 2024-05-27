# Github

`Github` 工具包包含一些工具，使 LLM 代理能够与 github 仓库进行交互。

该工具是 [PyGitHub](https://github.com/PyGithub/PyGithub) 库的一个封装。

## 快速开始

1. 安装 pygithub 库

2. 创建一个 Github 应用

3. 设置你的环境变量

4. 通过 `toolkit.get_tools()` 将工具传递给你的代理

下面将详细解释每个步骤。

1. **获取问题**- 从仓库获取问题。

2. **获取问题详情**- 获取特定问题的详细信息。

3. **在问题上发表评论**- 在特定问题上发布评论。

4. **创建拉取请求**- 从机器人的工作分支创建一个拉取请求到基础分支。

5. **创建文件**- 在仓库中创建一个新文件。

6. **读取文件**- 从仓库中读取一个文件。

7. **更新文件**- 更新仓库中的一个文件。

8. **删除文件**- 从仓库中删除一个文件。

## 设置

### 1. 安装 `pygithub` 库

```python
%pip install --upgrade --quiet  pygithub
```

### 2. 创建一个 Github 应用

[按照这里的说明](https://docs.github.com/en/apps/creating-github-apps/registering-a-github-app/registering-a-github-app) 创建并注册一个 Github 应用。确保你的应用具有以下[仓库权限](https://docs.github.com/en/rest/overview/permissions-required-for-github-apps?apiVersion=2022-11-28)：

- 提交状态 (只读)

- 内容 (读和写)

- 问题 (读和写)

- 元数据 (只读)

- 拉取请求 (读和写)

注册应用后，你必须授予你的应用访问你希望其操作的每个仓库的权限。在[这里的 github.com 应用设置](https://github.com/settings/installations)中使用应用设置。

### 3. 设置环境变量

在初始化代理之前，需要设置以下环境变量：

- **GITHUB_APP_ID**- 在你的应用的常规设置中找到的六位数字

- **GITHUB_APP_PRIVATE_KEY**- 你的应用私钥 .pem 文件的位置，或该文件的完整文本字符串。

- **GITHUB_REPOSITORY**- 你希望你的机器人操作的 Github 仓库的名称。必须遵循格式 {用户名}/{仓库名称}。*确保该应用已添加到该仓库中！*

- 可选: **GITHUB_BRANCH**- 机器人将进行提交的分支。默认为 `repo.default_branch`。

- 可选: **GITHUB_BASE_BRANCH**- 你的仓库的基础分支，PR 将基于此分支。默认为 `repo.default_branch`。

## 示例: 简单代理

```python
import os
from langchain.agents import AgentType, initialize_agent
from langchain_community.agent_toolkits.github.toolkit import GitHubToolkit
from langchain_community.utilities.github import GitHubAPIWrapper
from langchain_openai import ChatOpenAI
```
```python
# 使用 os.environ 设置你的环境变量
os.environ["GITHUB_APP_ID"] = "123456"
os.environ["GITHUB_APP_PRIVATE_KEY"] = "path/to/your/private-key.pem"
os.environ["GITHUB_REPOSITORY"] = "username/repo-name"
os.environ["GITHUB_BRANCH"] = "bot-branch-name"
os.environ["GITHUB_BASE_BRANCH"] = "main"
# 这个示例还需要一个 OpenAI API 密钥
os.environ["OPENAI_API_KEY"] = ""
```
```python
llm = ChatOpenAI(temperature=0, model="gpt-4-1106-preview")
github = GitHubAPIWrapper()
toolkit = GitHubToolkit.from_github_api_wrapper(github)
tools = toolkit.get_tools()
# STRUCTURED_CHAT 包括每个工具的 args_schema，有助于工具参数解析错误。
agent = initialize_agent(
    tools,
    llm,
    agent=AgentType.STRUCTURED_CHAT_ZERO_SHOT_REACT_DESCRIPTION,
    verbose=True,
)
print("可用工具:")
for tool in tools:
    print("\t" + tool.name)
```
```output
可用工具:
	Get Issues
	Get Issue
	Comment on Issue
	List open pull requests (PRs)
	Get Pull Request
	Overview of files included in PR
	Create Pull Request
	List Pull Requests' Files
	Create File
	Read File
	Update File
	Delete File
	Overview of existing files in Main branch
	Overview of files in current working branch
	List branches in this repository
	Set active branch
	Create a new branch
	Get files from a directory
	Search issues and pull requests
	Search code
	Create review request
```
```python
agent.run(
    "You have the software engineering capabilities of a Google Principle engineer. You are tasked with completing issues on a github repository. Please look at the existing issues and complete them."
)
```
```output
> 进入新的 AgentExecutor 链...
我需要弄清楚哪些问题需要完成。
动作: 获取问题
动作输入: N/A
观察: 找到 1 个问题:
[{'title': 'Update README file', 'number': 9}]
想法: 我需要获取关于这个问题的更多信息。
动作: 获取问题详情
动作输入: 9
观察: {"title": "Update README file", "body": "Find what the most popular frontend framework is right now and add a short blurb to the readme.md file about how this website will take advantage of it.", "comments": "[]"}
想法: 我需要更新 README 文件。
动作: 创建文件
动作输入: README.md
观察: 文件 README.md 已存在。请使用 update_file。
想法: 我需要更新现有的 README 文件。
动作: 更新文件
动作输入: README.md
旧 <<<<
This is a sample website
>>>> 旧
新 <<<<
This is a sample website that uses the most popular frontend framework.
>>>> 新
观察: 未更新文件内容，因为未找到旧内容。可能有助于使用 read_file 动作获取当前文件内容。
想法: 我需要获取当前文件内容。
动作: 读取文件
动作输入: README.md
观察: This is my awesome website!
想法: 我需要用新内容更新 README 文件。
动作: 更新文件
动作输入: README.md
旧 <<<<
This is my awesome website!
>>>> 旧
新 <<<<
This is my awesome website that uses the most popular frontend framework.
>>>> 新
观察: 更新了文件 README.md
想法: 我现在知道最终答案。
最终答案: README.md 文件已更新为新内容。
> 链结束。
```
```python
from langchain.memory.summary_buffer import ConversationSummaryBufferMemory
from langchain_core.prompts.chat import MessagesPlaceholder
summarizer_llm = ChatOpenAI(temperature=0, model="gpt-3.5-turbo")  # type: ignore
chat_history = MessagesPlaceholder(variable_name="chat_history")
memory = ConversationSummaryBufferMemory(
    memory_key="chat_history",
    return_messages=True,
    llm=summarizer_llm,
    max_token_limit=2_000,
)
agent = initialize_agent(
    tools,
    llm,
    agent=AgentType.STRUCTURED_CHAT_ZERO_SHOT_REACT_DESCRIPTION,
    verbose=True,
    handle_parsing_errors=True,  # or pass a function that accepts the error and returns a string
    max_iterations=30,
)
```

The code above initializes an agent for a chat conversation. It uses the LangChain library to manage conversation memory and a summarizer model (GPT-3.5-turbo) for generating responses. The `ConversationSummaryBufferMemory` class is used to store and retrieve conversation history. The `MessagesPlaceholder` class is used to create a placeholder variable for the chat history. The `initialize_agent` function is then called to initialize the agent with the necessary tools and parameters.

```python
max_execution_time=None,
early_stopping_method="generate",
memory=memory,
# trim_intermediate_steps=fancier_trim_intermediate_steps,
agent_kwargs={
    "memory_prompts": [chat_history],
    "input_variables": ["input", "agent_scratchpad", "chat_history"],
    "prefix": final_gh_issue_prompt,
},
)
```
```python
from langchain_core.tracers.context import tracing_v2_enabled
# 使用langsmith（推荐用于这些长时间任务）：
os.environ["LANGCHAIN_TRACING_V2"] = "true"
os.environ["LANGCHAIN_ENDPOINT"] = "https://api.smith.langchain.com"
os.environ["LANGCHAIN_API_KEY"] = "ls__......"
os.environ["LANGCHAIN_PROJECT"] = "Github_Demo_PR"
os.environ["LANGCHAIN_WANDB_TRACING"] = "false"
with tracing_v2_enabled(project_name="Github_Demo_PR", tags=["PR_bot"]) as cb:
    agent.run(final_gh_issue_prompt)
```
```output
> 进入新的AgentExecutor链...
```json
{
  "action": "从目录中获取文件",
  "action_input": "ML4Bio/tree/main/Report_WholeBrain"
}
```
观察：错误：状态码404，无
思考：由于提供的路径不存在或无法访问，之前尝试从目录中获取文件的操作失败了。我需要更正路径以访问`Report_WholeBrain`目录中的文件。让我们尝试从正确的目录路径获取文件列表。
操作：
```json
{
  "action": "从目录中获取文件",
  "action_input": "Report_WholeBrain"
}
```
观察：['Report_WholeBrain/MDSclustering_WholeBrain.html', 'Report_WholeBrain/MDSclustering_WholeBrain_RUVremoved.html', 'Report_WholeBrain/Report_Antonson_WholeBrain_2022Mar.Rmd', 'Report_WholeBrain/Report_WholeBrain_files/figure-html/Figure 1-1.png', 'Report_WholeBrain/Report_WholeBrain_files/figure-html/Figure 2-1.png', 'Report_WholeBrain/Report_WholeBrain_files/figure-html/Figure 3-1.png', 'Report_WholeBrain/Report_WholeBrain_files/figure-html/Figure 4-1.png', 'Report_WholeBrain/Report_WholeBrain_files/figure-html/Figure 6-1.png', 'Report_WholeBrain/Report_WholeBrain_files/figure-html/Figure 7-1.png', 'Report_WholeBrain/Report_WholeBrain_files/figure-html/Figure 8-1.png', 'Report_WholeBrain/Report_WholeBrain_files/figure-html/Figure 9-1.png', 'Report_WholeBrain/SalmonSummarizedOutput.RData', 'Report_WholeBrain/SampleInfo_RUVvariables_WholeBrain_2022-05-12.csv', 'Report_WholeBrain/Targets_Final.txt', 'Report_WholeBrain/WholeBrain_GeneResults_2022-05-12.xlsx', 'Report_WholeBrain/WholeBrain_GeneResults_RUV_2022-05-12.xlsx', 'Report_WholeBrain/WholeBrain_Gene_level_counts_2022-05-12.xlsx', 'Report_WholeBrain/WholeBrain_RUV_FDR0.1.html', 'Report_WholeBrain/WholeBrain_logCPMValues_RUVcorrected_2022-05-12.xlsx', 'Report_WholeBrain/WholeBrain_logCPMvalues_2022-05-12.xlsx', 'Report_WholeBrain/WholeBrain_rawP05.html', 'Report_WholeBrain/getGO.R', 'Report_WholeBrain/getPath.R', 'Report_WholeBrain/interactive_plots/css/glimma.min.css', 'Report_WholeBrain/interactive_plots/css/src/images/animated-overlay.gif', 'Report_WholeBrain/interactive_plots/css/src/images/favicon.ico', 'Report_WholeBrain/interactive_plots/css/src/images/sort_asc.png', 'Report_WholeBrain/interactive_plots/css/src/images/sort_asc_disabled.png', 'Report_WholeBrain/interactive_plots/css/src/images/sort_both.png', 'Report_WholeBrain/interactive_plots/css/src/images/sort_desc.png', 'Report_WholeBrain/interactive_plots/css/src/images/sort_desc_disabled.png', 'Report_WholeBrain/interactive_plots/css/src/images/ui-bg_flat_0_aaaaaa_40x100.png', 'Report_WholeBrain/interactive_plots/css/src/images/ui-bg_flat_75_ffffff_40x100.png', 'Report_WholeBrain/interactive_plots/css/src/images/ui-bg_glass_55_fbf9ee_1x400.png', 'Report_WholeBrain/interactive_plots/css/src/images/ui-bg_glass_65_ffffff_1x400.png', 'Report_WholeBrain/interactive_plots/css/src/images/ui-bg_glass_75_dadada_1x400.png', 'Report_WholeBrain/interactive_plots/css/src/images/ui-bg_glass_75_e6e6e6_1x400.png', 'Report_WholeBrain/interactive_plots/css/src/images/ui-bg_glass_95_fef1ec_1x400.png', 'Report_WholeBrain/interactive_plots/css/src/images/ui-bg_highlight-soft_75_cccccc_1x100.png', 'Report_WholeBrain/interactive_plots/css/src/images/ui-icons_222222_256x240.png', 'Report_WholeBrain/interactive_plots/css/src/images/ui-icons_2e83ff_256x240.png', 'Report_WholeBrain/interactive_plots/css/src/images/ui-icons_454545_256x240.png', 'Report_WholeBrain/interactive_plots/css/src/images/ui-icons_888888_256x240.png', 'Report_WholeBrain/interactive_plots/css/src/images/ui-icons_cd0a0a_256x240.png', 'Report_WholeBrain/interactive_plots/js/glimma.min.js', 'Report_WholeBrain/interactive_plots/js/old_MDSclustering_Microglia.js', 'Report_WholeBrain/interactive_plots/js/old_MDSclustering_Microglia_RUV.js', 'Report_WholeBrain/interactive_plots/js/old_MDSclustering_WholeBrain.js', 'Report_WholeBrain/interactive_plots/js/old_MDSclustering_WholeBrain_RUV.js', 'Report_WholeBrain/interactive_plots/js/old_MDSclustering_WholeBrain_noOUT.js', 'Report_WholeBrain/interactive_plots/js/old_Microglia_rawP05.js', 'Report_WholeBrain/interactive_plots/js/old_WholeBrain_RUV_FDR0.1.js', 'Report_WholeBrain/interactive_plots/js/old_WholeBrain_rawP05.js', 'Report_WholeBrain/interactive_plots/old_MDSclustering_Microglia.html', 'Report_WholeBrain/interactive_plots/old_MDSclustering_Microglia_RUV.html', 'Report_WholeBrain/interactive_plots/old_MDSclustering_WholeBrain.html', 'Report_WholeBrain/interactive_plots/old_MDSclustering_WholeBrain_RUV.html', 'Report_WholeBrain/interactive_plots/old_MDSclustering_WholeBrain_noOUT.html', 'Report_WholeBrain/interactive_plots/old_Microglia_rawP05.html', 'Report_WholeBrain/interactive_plots/old_WholeBrain_RUV_FDR0.1.html', 'Report_WholeBrain/interactive_plots/old_WholeBrain_rawP05.html', 'Report_WholeBrain/trx_EGids_combined.txt', 'Report_WholeBrain/⭐️____Report_Antonson_WholeBrain_2022Mar.html']
想法：`Report_WholeBrain` 目录中的文件似乎主要是报告、图片和一些 R 脚本。然而，这些文件似乎都不是原始的 RNA-Seq 数据文件或与用户请求的 RNA-Seq 工作流直接相关的脚本。为了继续创建工作流程，我需要创建新的 bash 和 R 脚本，以处理使用提到的软件栈（FastQC、MultiQC、STAR、RSEM、samtools、DESeq2）进行 RNA-Seq 分析。
在创建脚本之前，我应该创建一个新的分支来处理这个功能。
操作：
```json
{
  "action": "创建一个新分支",
  "action_input": "rna-seq-workflow"
}
```
观察：成功创建分支'rna-seq-workflow_v1'，并将其设置为当前活动分支。
想法：现在新分支`rna-seq-workflow_v1`已创建并激活，我可以开始为RNA-Seq工作流创建必要的bash和R脚本。我将首先创建一个用于质量控制的bash脚本，使用FastQC和MultiQC。
操作：
```json
{
  "action": "创建文件",
  "action_input": "rna-seq-workflow/01_quality_control.sh\n#!/bin/bash\n#PBS -N quality_control\n#PBS -l walltime=04:00:00\n#PBS -l nodes=1:ppn=8\n#PBS -q batch\n\n# 加载FastQC和MultiQC模块\nmodule load FastQC/0.11.8\nmodule load MultiQC/1.9\n\n# 定义输入和输出目录\nINPUT_DIR='/path/to/raw_data'\nOUTPUT_DIR='/path/to/output/quality_control'\n\n# 如果输出目录不存在，则创建输出目录\nmkdir -p $OUTPUT_DIR\n\n# 对输入目录中的所有文件运行FastQC\nfastqc -o $OUTPUT_DIR -t 8 $INPUT_DIR/*.fastq.gz\n\n# 运行MultiQC以汇总结果\nmultiqc -o $OUTPUT_DIR $OUTPUT_DIR\n"
}
```
观察：已创建文件rna-seq-workflow/01_quality_control.sh
想法：质量控制bash脚本已成功创建。接下来，我将创建一个用于比对步骤的bash脚本，使用STAR。
操作：
```json
{
  "action": "创建文件",
  "action_input": "rna-seq-workflow/02_alignment.sh\n#!/bin/bash\n#PBS -N alignment\n#PBS -l walltime=08:00:00\n#PBS -l nodes=1:ppn=16\n#PBS -q batch\n\n# 加载STAR模块\nmodule load STAR/2.7.3a\n\n# 定义输入和输出目录\nINPUT_DIR='/path/to/quality_control'\nOUTPUT_DIR='/path/to/output/alignment'\nGENOME_DIR='/path/to/genome_indices'\n\n# 如果输出目录不存在，则创建输出目录\nmkdir -p $OUTPUT_DIR\n\n# 为每个样本运行STAR\nfor sample in $INPUT_DIR/*.fastq.gz\ndo\n  # 定义输出文件名\n  prefix=$(basename $sample .fastq.gz)\n  output_prefix=$OUTPUT_DIR/$prefix\n\n  # 运行STAR\n  STAR --runThreadN 16 --genomeDir $GENOME_DIR --readFilesIn $sample --readFilesCommand zcat --outFileNamePrefix $output_prefix --outSAMtype BAM SortedByCoordinate\n\ndone\n"
}
```
观察：已创建文件rna-seq-workflow/02_alignment.sh
想法：比对bash脚本已成功创建。接下来，我将创建一个用于转录本定量的bash脚本，使用RSEM。
操作：
```json
{
  "action": "创建文件",
  "action_input": "rna-seq-workflow/03_transcript_quantification.sh\n#!/bin/bash\n#PBS -N transcript_quantification\n#PBS -l walltime=12:00:00\n#PBS -l nodes=1:ppn=8\n#PBS -q batch\n\n# 加载RSEM模块\nmodule load RSEM/1.3.3\n\n# 定义输入和输出目录\nINPUT_DIR='/path/to/alignment'\nOUTPUT_DIR='/path/to/output/transcript_quantification'\nREFERENCE_DIR='/path/to/reference'\n\n# 如果输出目录不存在，则创建输出目录\nmkdir -p $OUTPUT_DIR\n\n# 为每个样本运行RSEM\nfor bam in $INPUT_DIR/*.bam\ndo\n  # 定义输出文件名\n  prefix=$(basename $bam .bam)\n  output_prefix=$OUTPUT_DIR/$prefix\n\n  # 运行RSEM\n  rsem-calculate-expression --bam --no-bam-output --estimate-rspd --append-names --output-genome-bam -p 8 $bam $REFERENCE_DIR $output_prefix\n\ndone\n"
}
```
观察：已创建文件rna-seq-workflow/03_transcript_quantification.sh
想法：转录本定量bash脚本已成功创建。现在，我将创建一个用于使用samtools对BAM文件进行排序和索引的bash脚本，这是在差异表达分析之前的常见步骤。
操作：
```json
{
  "action": "创建文件",
  "action_input": "rna-seq-workflow/04_sort_index_bam.sh\n#!/bin/bash\n#PBS -N sort_index_bam\n#PBS -l walltime=02:00:00\n#PBS -l nodes=1:ppn=4\n#PBS -q batch\n\n# 加载samtools模块\nmodule load samtools/1.10\n\n# 定义输入和输出目录\nINPUT_DIR='/path/to/alignment'\nOUTPUT_DIR='/path/to/output/sorted_bams'\n\n# 如果输出目录不存在，则创建输出目录\nmkdir -p $OUTPUT_DIR\n\n# 对BAM文件进行排序和索引\nfor bam in $INPUT_DIR/*.bam\ndo\n  # 定义输出文件名\n  sorted_bam=$OUTPUT_DIR/$(basename $bam .bam).sorted.bam\n\n  # 对BAM文件进行排序\n  samtools sort -@ 4 -o $sorted_bam $bam\n\n  # 索引排序后的BAM文件\n  samtools index $sorted_bam\ndone\n"
}
```
观察：已创建文件rna-seq-workflow/04_sort_index_bam.sh
想法：BAM文件的排序和索引脚本已成功创建。下一步是创建一个用于使用DESeq2进行差异表达分析的R脚本。此脚本将在完成工作流中的先前步骤后执行。
```
```python
from langchain.tools.render import render_text_description_and_args
tools = {
  "Create File": {
    "description": "This tool is a wrapper for the GitHub API, useful when you need to create a file in a GitHub repository. **VERY IMPORTANT**: Your input to this tool MUST strictly follow these rules:\n\n\n\n- First you must specify which file to create by passing a full file path (**IMPORTANT**: the path must not start with a slash)\n\n- Then you must specify the contents of the file\n\n\n\nFor example, if you would like to create a file called /test/test.txt with contents \"test contents\", you would pass in the following string:\n\n\ntest/test.txt\n\n\ntest contents",
    "args": {
      "formatted_file": {
        "title": "Formatted File",
        "description": "Follow the required formatting.",
        "type": "string"
      }
    }
  },
  "Create Pull Request": {
    "description": "This tool is useful when you need to create a new pull request in a GitHub repository. **VERY IMPORTANT**: Your input to this tool MUST strictly follow these rules:\n\n\n\n- First you must specify the title of the pull request\n\n- Then you must place two newlines\n\n- Then you must write the body or description of the pull request\n\n\n\nWhen appropriate, always reference relevant issues in the body by using the syntax `closes #<issue_number` like `closes #3, closes #6`.\n\nFor example, if you would like to create a pull request called \"README updates\" with contents \"added contributors' names, closes #3\", you would pass in the following string:\n\n\n\nREADME updates\n\n\n\nadded contributors' names, closes #3",
    "args": {
      "formatted_pr": {
        "title": "Formatted Pr",
        "description": "Follow the required formatting.",
        "type": "string"
      }
    }
  },
  "Comment on Issue": {
    "description": "This tool is useful when you need to comment on a GitHub issue. Simply pass in the issue number and the comment you would like to make. Please use this sparingly as we don't want to clutter the comment threads. **VERY IMPORTANT**: Your input to this tool MUST strictly follow
## 示例：带搜索功能的代理
如果您的代理程序不需要使用所有 8 个工具，您可以单独构建要使用的工具。在这个示例中，我们将创建一个代理程序，它不使用 create_file、delete_file 或 create_pull_request 工具，但可以使用 duckduckgo-search。
```python

%pip install --upgrade --quiet  duckduckgo-search

```
```python
from langchain_community.tools import DuckDuckGoSearchRun
from langchain_core.tools import Tool
from langchain_openai import ChatOpenAI
tools = []
unwanted_tools = ["Get Issue", "Delete File", "Create File", "Create Pull Request"]
for tool in toolkit.get_tools():
    if tool.name not in unwanted_tools:
        tools.append(tool)
tools += [
    Tool(
        name="Search",
        func=DuckDuckGoSearchRun().run,
        description="useful for when you need to search the web",
    )
]
agent = initialize_agent(
    tools=tools,
    llm=ChatOpenAI(temperature=0.1),
    agent=AgentType.ZERO_SHOT_REACT_DESCRIPTION,
    verbose=True,
)
```
最后让我们构建一个提示并进行测试！
```python
# GitHubAPIWrapper 也可以在代理之外使用
# 这里获取有关问题编号 9 的信息，因为我们想要
# 强制代理程序解决这个特定问题。
issue = github.get_issue(9)
prompt = f"""
您是一位有经验的高级前端开发人员，精通 HTML、CSS 和 JS-尤其是 React。
您已被分配下面的问题。尽力完成它。
记得先制定计划，注意文件名和常识等细节。
然后执行计划，适当使用工具。
最后，创建一个拉取请求以合并您的更改。
问题：{issue["title"]}
问题描述：{issue['body']}
评论：{issue['comments']}"""
agent.run(prompt)
```
```output

> 进入新的 AgentExecutor 链...

要完成这个问题，我需要找到最流行的前端框架，并在 readme.md 文件中添加一个关于这个网站如何利用它的简介。我应该先研究最流行的前端框架，然后相应地更新 readme 文件。我将使用 "Search" 工具来研究最流行的前端框架。

动作：搜索

动作输入："most popular frontend framework"

观察：Alex Ivanovs February 25, 2023 Table of Contents What are the current Front-end trends? Top Front-end Frameworks for 2023 #1 - React #2 - Angular #3 - Vue #4 - Svelte #5 - Preact #6 - Ember #7 - Solid #8 - Lit #9 - Alpine #10 - Stencil #11 - Qwik Front-end Frameworks: A Summary Top 6 Frontend Frameworks To Use in 2022 by Nwose Lotanna Victor August 26, 2022 Web 0 Comments This post reveals the top six frontend libraries to use in 2022. The list is fresh and very different from the previous years. State of JS Though React is the most popular framework for frontend development, it also has some shortcomings. Due to its limitations, the idea was to design a small-size framework that will offer the same features as React. This is how a tiny version of React — Preact — appeared. Top 10 Popular Frontend Frameworks to Use in 2023 Sep 26, 2022 10 min Сontents 1. What is a framework? 2. Front-end frameworks vs backend frameworks 3. The best front-end frameworks in 2023 React Vue.js Angular Svelte JQuery Ember Backbone Semantic UI 4. Final words Technostacks Jan 11 2023 Top Frontend Frameworks of 2023 for Web Development Developing what the users see on their screens is the role of a front-end web developer. Unarguably, front-end developers worldwide are trying to use the best front-end frameworks to provide the best user experience.

思考：根据我的研究，目前最流行的前端框架是 React。现在我将更新 readme.md 文件，包括一个关于这个网站如何利用 React 的简介。

动作：更新文件

动作输入：

README.md

旧 <<<<

This is the readme file for the website.

>>>> 旧

新 <<<<

This is the readme file for the website.

This website takes advantage of the React framework, which allows for efficient and reusable UI components. With React, we can easily manage the state of our application and create interactive user interfaces. It provides a smooth and seamless user experience, making this website highly responsive and dynamic.

>>>> 新

观察：文件内容未更新，因为未找到旧内容。可能有助于使用 read_file 动作获取当前文件内容。

思考：我需要先读取 README.md 文件的内容，以获取当前内容。然后我可以使用新内容更新文件。

动作：读取文件

动作输入：README.md

观察：This is my awesome website that uses the most popular frontend framework.

思考：README.md 文件的当前内容是 "This is my awesome website that uses the most popular frontend framework." 现在我可以使用新内容更新文件。

动作：更新文件

动作输入：

README.md

旧 <<<<

```
这是我的精彩网站，采用了最流行的前端框架。
这个网站利用了 React 框架，可以实现高效、可重用的用户界面组件。借助 React，我们可以轻松管理应用程序的状态，并创建交互式用户界面。它提供了流畅无缝的用户体验，使得这个网站具有高度的响应性和动态性。
观察：已更新文件 README.md
想法：我已成功更新了 README.md 文件，其中包含了关于这个网站如何利用 React 框架的描述。
最终答案：目前最流行的前端框架是 React。这个网站利用 React 来创建高效、可重用的用户界面组件，管理应用程序状态，并提供流畅无缝的用户体验。
> 完成链条。
```markdown

'The most popular frontend framework right now is React. This website takes advantage of React to create efficient and reusable UI components, manage application state, and provide a smooth and seamless user experience.'

```