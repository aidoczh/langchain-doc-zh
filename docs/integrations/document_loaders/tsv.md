# TSV

> [制表符分隔值（TSV）](https://en.wikipedia.org/wiki/Tab-separated_values)文件是一种简单的基于文本的文件格式，用于存储表格数据。[3] 记录之间由换行符分隔，记录内的值由制表符分隔。

## `UnstructuredTSVLoader`

您还可以使用 `UnstructuredTSVLoader` 加载表格。使用 `UnstructuredTSVLoader` 的一个优势是，如果在“elements”模式下使用它，表格的 HTML 表示将在元数据中可用。

```python
from langchain_community.document_loaders.tsv import UnstructuredTSVLoader
```

```python
loader = UnstructuredTSVLoader(
    file_path="example_data/mlb_teams_2012.csv", mode="elements"
)
docs = loader.load()
```

```python
print(docs[0].metadata["text_as_html"])
```

```output
<table border="1" class="dataframe">
  <tbody>
    <tr>
      <td>Nationals,     81.34, 98</td>
    </tr>
    <tr>
      <td>Reds,          82.20, 97</td>
    </tr>
    <tr>
      <td>Yankees,      197.96, 95</td>
    </tr>
    <tr>
      <td>Giants,       117.62, 94</td>
    </tr>
    <tr>
      <td>Braves,        83.31, 94</td>
    </tr>
    <tr>
      <td>Athletics,     55.37, 94</td>
    </tr>
    <tr>
      <td>Rangers,      120.51, 93</td>
    </tr>
    <tr>
      <td>Orioles,       81.43, 93</td>
    </tr>
    <tr>
      <td>Rays,          64.17, 90</td>
    </tr>
    <tr>
      <td>Angels,       154.49, 89</td>
    </tr>
    <tr>
      <td>Tigers,       132.30, 88</td>
    </tr>
    <tr>
      <td>Cardinals,    110.30, 88</td>
    </tr>
    <tr>
      <td>Dodgers,       95.14, 86</td>
    </tr>
    <tr>
      <td>White Sox,     96.92, 85</td>
    </tr>
    <tr>
      <td>Brewers,       97.65, 83</td>
    </tr>
    <tr>
      <td>Phillies,     174.54, 81</td>
    </tr>
    <tr>
      <td>Diamondbacks,  74.28, 81</td>
    </tr>
    <tr>
      <td>Pirates,       63.43, 79</td>
    </tr>
    <tr>
      <td>Padres,        55.24, 76</td>
    </tr>
    <tr>
      <td>Mariners,      81.97, 75</td>
    </tr>
    <tr>
      <td>Mets,          93.35, 74</td>
    </tr>
    <tr>
      <td>Blue Jays,     75.48, 73</td>
    </tr>
    <tr>
      <td>Royals,        60.91, 72</td>
    </tr>
    <tr>
      <td>Marlins,      118.07, 69</td>
    </tr>
    <tr>
      <td>Red Sox,      173.18, 69</td>
    </tr>
    <tr>
      <td>Indians,       78.43, 68</td>
    </tr>
    <tr>
      <td>Twins,         94.08, 66</td>
    </tr>
    <tr>
      <td>Rockies,       78.06, 64</td>
    </tr>
    <tr>
      <td>Cubs,          88.19, 61</td>
    </tr>
    <tr>
      <td>Astros,        60.65, 55</td>
    </tr>
  </tbody>
</table>
```