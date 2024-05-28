# langchain-doc-zh
## 一、项目简介

LangChain是使用非常广的大模型编排工具，可以低代码的做大模型各种应用，有点类似在数据分析处理里面Pandas的地位。所以我有了一些想把一些工具的文档翻译成中文的想法。希望对于大家有一些帮助。 由于文档较多，人力和能力都有限，有可能很多地方有问题，如果发现，请给我反馈，我会修改优化。希望能抛砖引玉，更多人加入到翻译优秀AI工具文档中，对于国内广大Langchain使用者有一点帮助。未来我也会陆续翻译更多的优秀AI工具的文档。

### 1、在线文档

已经部署了在线文档 请看这个地址 http://www.aidoczh.com/langchain/v0.2/docs/introduction/

![](https://github.com/aidoczh/langchain-doc-zh/blob/main/static/img/screenshot.png)

![](https://github.com/aidoczh/langchain-doc-zh/blob/main/static/img/screen1.png)

![](https://github.com/aidoczh/langchain-doc-zh/blob/main/static/img/jupyter_screen.png)

## 二、项目内容

### 1、已经翻译的内容

（1）翻译了LangChain的python V0.2版的文档 https://python.langchain.com/v0.2/

（2）翻译了LangChain的CookBook https://github.com/langchain-ai/langchain/blob/master/cookbook/

### 2、后续待翻译的内容

（1）LangChain的API Reference



## 三、项目启动

### 1、需要安装yarn

```
# npm的版本大于等于8.15.0 yarn版本大于等于1.22.22
npm install -g yarn 
```

### 2、 Clone 项目到本地

```
git clone https://github.com/aidoczh/langchain-doc-zh.git
```

### 3、安装相关配置工具包

```
cd langchain-doc-zh
yarn
```

![](https://github.com/aidoczh/langchain-doc-zh/blob/main/static/img/yarn_install.png)

### 4、测试启动程序

```
yarn run start
```

![](https://github.com/aidoczh/langchain-doc-zh/blob/main/static/img/yarn_run_start.png)

### 5、代码编译

```
yarn run docusaurus build
```

### 6、 程序启动

```
npm run serve
```

![](https://github.com/aidoczh/langchain-doc-zh/blob/main/static/img/npm_run_serve.png)

### 7、文档查看

```
http://localhost:3000/v0.2/
```



## 四、公众号

我的公众号是数智笔记，欢迎关注。

![](https://github.com/aidoczh/langchain-doc-zh/blob/main/static/img/qrcode.jpg)
