# GitBook

[GitBook](https://docs.gitbook.com/) 是一个现代文档平台，团队可以在上面记录从产品到内部知识库和 API 等各种内容。

这个笔记本展示了如何从任何 `GitBook` 中提取页面数据。

```python
from langchain_community.document_loaders import GitbookLoader
```

### 从单个 GitBook 页面加载

```python
loader = GitbookLoader("https://docs.gitbook.com")
```

```python
page_data = loader.load()
```

```python
page_data
```

```output
[Document(page_content='Introduction to GitBook\nGitBook is a modern documentation platform where teams can document everything from products to internal knowledge bases and APIs.\nWe want to help \nteams to work more efficiently\n by creating a simple yet powerful platform for them to \nshare their knowledge\n.\nOur mission is to make a \nuser-friendly\n and \ncollaborative\n product for everyone to create, edit and share knowledge through documentation.\nPublish your documentation in 5 easy steps\nImport\n\nMove your existing content to GitBook with ease.\nGit Sync\n\nBenefit from our bi-directional synchronisation with GitHub and GitLab.\nOrganise your content\n\nCreate pages and spaces and organize them into collections\nCollaborate\n\nInvite other users and collaborate asynchronously with ease.\nPublish your docs\n\nShare your documentation with selected users or with everyone.\nNext\n - Getting started\nOverview\nLast modified \n3mo ago', lookup_str='', metadata={'source': 'https://docs.gitbook.com', 'title': 'Introduction to GitBook'}, lookup_index=0)]
```

### 从给定 GitBook 中的所有路径加载

为了使此操作生效，GitbookLoader 需要以根路径（在本例中为 `https://docs.gitbook.com`）进行初始化，并将 `load_all_paths` 设置为 `True`。

```python
loader = GitbookLoader("https://docs.gitbook.com", load_all_paths=True)
all_pages_data = loader.load()
```

```output
Fetching text from https://docs.gitbook.com/
Fetching text from https://docs.gitbook.com/getting-started/overview
Fetching text from https://docs.gitbook.com/getting-started/import
Fetching text from https://docs.gitbook.com/getting-started/git-sync
Fetching text from https://docs.gitbook.com/getting-started/content-structure
Fetching text from https://docs.gitbook.com/getting-started/collaboration
Fetching text from https://docs.gitbook.com/getting-started/publishing
Fetching text from https://docs.gitbook.com/tour/quick-find
Fetching text from https://docs.gitbook.com/tour/editor
Fetching text from https://docs.gitbook.com/tour/customization
Fetching text from https://docs.gitbook.com/tour/member-management
Fetching text from https://docs.gitbook.com/tour/pdf-export
Fetching text from https://docs.gitbook.com/tour/activity-history
Fetching text from https://docs.gitbook.com/tour/insights
Fetching text from https://docs.gitbook.com/tour/notifications
Fetching text from https://docs.gitbook.com/tour/internationalization
Fetching text from https://docs.gitbook.com/tour/keyboard-shortcuts
Fetching text from https://docs.gitbook.com/tour/seo
Fetching text from https://docs.gitbook.com/advanced-guides/custom-domain
Fetching text from https://docs.gitbook.com/advanced-guides/advanced-sharing-and-security
Fetching text from https://docs.gitbook.com/advanced-guides/integrations
Fetching text from https://docs.gitbook.com/billing-and-admin/account-settings
Fetching text from https://docs.gitbook.com/billing-and-admin/plans
Fetching text from https://docs.gitbook.com/troubleshooting/faqs
Fetching text from https://docs.gitbook.com/troubleshooting/hard-refresh
Fetching text from https://docs.gitbook.com/troubleshooting/report-bugs
Fetching text from https://docs.gitbook.com/troubleshooting/connectivity-issues
Fetching text from https://docs.gitbook.com/troubleshooting/support
```

```python
print(f"fetched {len(all_pages_data)} documents.")
# show second document
all_pages_data[2]
```

```output
fetched 28 documents.
```

```output
Document(page_content="Import\nFind out how to easily migrate your existing documentation and which formats are supported.\nThe import function allows you to migrate and unify existing documentation in GitBook. You can choose to import single or multiple pages although limits apply. \nPermissions\nAll members with editor permission or above can use the import feature.\nSupported formats\nGitBook supports imports from websites or files that are:\nMarkdown (.md or .markdown)\nHTML (.html)\nMicrosoft Word (.docx).\nWe also support import from:\nConfluence\nNotion\nGitHub Wiki\nQuip\nDropbox Paper\nGoogle Docs\nYou can also upload a ZIP\n \ncontaining HTML or Markdown files when \nimporting multiple pages.\nNote: this feature is in beta.\nFeel free to suggest import sources we don't support yet and \nlet us know\n if you have any issues.\nImport panel\nWhen you create a new space, you'll have the option to import content straight away:\nThe new page menu\nImport a page or subpage by selecting \nImport Page\n from the New Page menu, or \nImport Subpage\n in the page action menu, found in the table of contents:\nImport from the page action menu\nWhen you choose your input source, instructions will explain how to proceed.\nAlthough GitBook supports importing content from different kinds of sources, the end result might be different from your source due to differences in product features and document format.\nLimits\nGitBook currently has the following limits for imported content:\nThe maximum number of pages that can be uploaded in a single import is \n20.\nThe maximum number of files (images etc.) that can be uploaded in a single import is \n20.\nGetting started - \nPrevious\nOverview\nNext\n - Getting started\nGit Sync\nLast modified \n4mo ago", lookup_str='', metadata={'source': 'https://docs.gitbook.com/getting-started/import', 'title': 'Import'}, lookup_index=0)
```

### 什么是无监督学习？

无监督学习是机器学习的一种方法，其目标是从未标记的数据中发现隐藏的模式和结构。与监督学习不同，无监督学习不需要标记的数据，而是通过对数据进行聚类、降维或异常检测等技术来发现数据中的潜在关系。无监督学习可以帮助我们理解数据的特征和分布，从而为后续的数据分析和决策提供有价值的信息。

### 无监督学习的应用领域

无监督学习在各个领域都有广泛的应用。以下是一些常见的应用领域：

- 聚类分析：将数据分成不同的组别，每个组别内的数据具有相似的特征。聚类分析可以用于市场细分、社交网络分析等。

- 降维：将高维数据转换为低维表示，以便更好地理解和可视化数据。降维可以用于图像处理、推荐系统等。

- 异常检测：识别与正常模式不同的异常数据点。异常检测可以用于网络安全、信用卡欺诈检测等。

- 关联规则挖掘：发现数据中的频繁项集和关联规则。关联规则挖掘可以用于市场篮子分析、推荐系统等。

### 无监督学习算法

无监督学习有许多不同的算法，用于实现不同的任务。以下是一些常见的无监督学习算法：

- K-means 聚类算法：将数据分成 K 个不同的组别，每个组别内的数据具有相似的特征。

- 主成分分析（PCA）：将高维数据转换为低维表示，以保留最重要的特征。

- 高斯混合模型（GMM）：将数据建模为多个高斯分布的混合。

- 孤立森林（Isolation Forest）：通过构建随机树来识别异常数据点。

- 关联规则挖掘算法：发现数据中的频繁项集和关联规则。

### 总结

无监督学习是一种从未标记的数据中发现模式和结构的机器学习方法。它在聚类分析、降维、异常检测和关联规则挖掘等领域有广泛的应用。无监督学习算法包括 K-means 聚类算法、主成分分析、高斯混合模型、孤立森林和关联规则挖掘算法等。通过无监督学习，我们可以更好地理解和利用未标记的数据。

参考文献：

[20] Bishop, C. M. (2006). Pattern recognition and machine learning. Springer.

```