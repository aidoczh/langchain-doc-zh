# MediaWiki Dump
[MediaWiki XML Dumps](https://www.mediawiki.org/wiki/Manual:Importing_XML_dumps) 包含了维基的内容（包括所有修订版本的维基页面），但不包含与网站相关的数据。XML dump 并不是维基数据库的完整备份，它不包含用户账户、图片、编辑日志等。
本文介绍了如何将 MediaWiki XML dump 文件加载为我们可以在下游使用的文档格式。
它使用了 `mediawiki-utilities` 中的 `mwxml` 进行 dump，使用了 `earwig` 中的 `mwparserfromhell` 进行解析 MediaWiki 的 wikicode。
可以使用 `dumpBackup.php` 或者在 Wiki 的 `Special:Statistics` 页面获取 dump 文件。
```python
# mediawiki-utilities 支持未合并分支中的 XML schema 0.11
%pip install --upgrade --quiet git+https://github.com/mediawiki-utilities/python-mwtypes@updates_schema_0.11
# mediawiki-utilities mwxml 存在一个 bug，修复 PR 正在等待合并
%pip install --upgrade --quiet git+https://github.com/gdedrouas/python-mwxml@xml_format_0.11
%pip install --upgrade --quiet mwparserfromhell
```
```python
from langchain_community.document_loaders import MWDumpLoader
```
```python
loader = MWDumpLoader(
    file_path="example_data/testmw_pages_current.xml",
    encoding="utf8",
    # namespaces = [0,2,3] 可选列表，只加载特定的命名空间。默认加载所有命名空间。
    skip_redirects=True,  # 如果为 True，则跳过只重定向到其他页面的页面；如果为 False，则不跳过
    stop_on_error=False,  # 如果为 True，则跳过导致解析错误的页面；如果为 False，则不跳过
)
documents = loader.load()
print(f"您的数据中有 {len(documents)} 个文档")
```
```output
您的数据中有 177 个文档
```
```python
documents[:5]
```
```output
[Document(page_content='\t\n\t\n\tArtist\n\tReleased\n\tRecorded\n\tLength\n\tLabel\n\tProducer', metadata={'source': 'Album'}),
 Document(page_content='{| class="article-table plainlinks" style="width:100%;"\n|- style="font-size:18px;"\n! style="padding:0px;" | Template documentation\n|-\n| Note: portions of the template sample may not be visible without values provided.\n|-\n| View or edit this documentation. (About template documentation)\n|-\n| Editors can experiment in this template\'s [ sandbox] and [ test case] pages.\n|}Category:Documentation templates', metadata={'source': 'Documentation'}),
 Document(page_content='Description\nThis template is used to insert descriptions on template pages.\n\nSyntax\nAdd <noinclude></noinclude> at the end of the template page.\n\nAdd <noinclude></noinclude> to transclude an alternative page from the /doc subpage.\n\nUsage\n\nOn the Template page\nThis is the normal format when used:\n\nTEMPLATE CODE\n<includeonly>Any categories to be inserted into articles by the template</includeonly>\n<noinclude>{{Documentation}}</noinclude>\n\nIf your template is not a completed div or table, you may need to close the tags just before {{Documentation}} is inserted (within the noinclude tags).\n\nA line break right before {{Documentation}} can also be useful as it helps prevent the documentation template "running into" previous code.\n\nOn the documentation page\nThe documentation page is usually located on the /doc subpage for a template, but a different page can be specified with the first parameter of the template (see Syntax).\n\nNormally, you will want to write something like the following on the documentation page:\n\n==Description==\nThis template is used to do something.\n\n==Syntax==\nType <code>{{t|templatename}}</code> somewhere.\n\n==Samples==\n<code><nowiki>{{templatename|input}}</nowiki></code> \n\nresults in...\n\n{{templatename|input}}\n\n<includeonly>Any categories for the template itself</includeonly>\n<noinclude>[[Category:Template documentation]]</noinclude>\n\nUse any or all of the above description/syntax/sample output sections. You may also want to add "see also" or other sections.\n\nNote that the above example also uses the Template:T template.\n\nCategory:Documentation templatesCategory:Template documentation', metadata={'source': 'Documentation/doc'}),
 Document(page_content='Description\nA template link with a variable number of parameters (0-20).\n\nSyntax\n \n\nSource\nImproved version not needing t/piece subtemplate developed on Templates wiki see the list of authors. Copied here via CC-By-SA 3.0 license.\n\nExample\n\nCategory:General wiki templates\nCategory:Template documentation', metadata={'source': 'T/doc'}),
 Document(page_content='\t\n\t\t    \n\t\n\t\t    Aliases\n\t    Relatives\n\t    Affiliation\n        Occupation\n    \n            Biographical information\n        Marital status\n    \tDate of birth\n        Place of birth\n        Date of death\n        Place of death\n    \n            Physical description\n        Species\n        Gender\n        Height\n        Weight\n        Eye color\n\t\n           Appearances\n       Portrayed by\n       Appears in\n       Debut\n    ', metadata={'source': 'Character'})]
```