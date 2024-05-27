# 黑板

[Blackboard Learn](https://en.wikipedia.org/wiki/Blackboard_Learn)（之前称为 Blackboard 学习管理系统）是由 Blackboard 公司开发的基于网络的虚拟学习环境和学习管理系统。该软件具有课程管理、可定制的开放式架构和可扩展的设计，可以与学生信息系统和认证协议集成。它可以安装在本地服务器上，由 Blackboard ASP Solutions 托管，也可以作为由 Amazon Web Services 托管的软件即服务提供。其主要目的是在传统面对面课程中添加在线元素，并开发几乎没有或没有面对面会议的完全在线课程。

这里介绍了如何从 [Blackboard Learn](https://www.anthology.com/products/teaching-and-learning/learning-effectiveness/blackboard-learn) 实例中加载数据。

此加载器不兼容所有的 `Blackboard` 课程。它只兼容使用新的 `Blackboard` 接口的课程。要使用此加载器，您必须拥有 BbRouter cookie。您可以通过登录课程，然后从浏览器的开发者工具中复制 BbRouter cookie 的值来获取此 cookie。

```python
from langchain_community.document_loaders import BlackboardLoader
loader = BlackboardLoader(
    blackboard_course_url="https://blackboard.example.com/webapps/blackboard/execute/announcement?method=search&context=course_entry&course_id=_123456_1",
    bbrouter="expires:12345...",
    load_all_recursively=True,
)
documents = loader.load()
```