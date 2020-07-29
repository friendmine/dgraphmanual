[参考英文](https://dgraph.io/tour//intro/1/)

## 欢迎
你好，此教程针对 Dgraph 20.03.3

### 欢迎来到Dgraph。

该交互式教程将帮助您使用Dgraph并开始运行，并使用Dgraph自己的图查询语言GraphQL+-编写图查询。

本教程分为多个模块。 您可以随时单击左上角的“ A graph of Dgraph”访问目录，并可以使用此窗格底部的向右和向左箭头来回移动。

在本教程中，您可以运行和编辑查询并与可视化进行交互。

本教程将指导您从第一个图查询到图专家都将为之骄傲的查询。

除了本教程之外，还有许多参考资料可帮助您使用Dgraph。

准备开始时，请单击向右箭头。

### 运行Dgraph
让我们在计算机上运行一个Dgraph实例；这样您就可以自行安装Dgraph和加载数据。

本教程中的所有数据将存储在您安装的实例中，所有查询也在本地计算机上查找Dgraph运行。如果在容器中运行它，您将拥有一个新的Dgraph，它也能用作沙箱供您学习和实验。

有几种安装和运行Dgraph的方法，所有这些方法都可以在[“下载”](https://dgraph.io/downloads)页面中找到。对于本教程，我们在Docker的容器中运行Dgraph。

首先，获取最新版本的Docker。

然后，使用以下命令提取最新的Dgraph版本：

`docker pull dgraph/dgraph:v20.03.0`

我们创建一个文件夹，用于在容器外部存储Dgraph数据：

`mkdir -p ~/dgraph`

现在，要在Docker中运行Dgraph，如下：

```
# Run Dgraph zero
docker run -it -p 5080:5080 -p 6080:6080 -p 8080:8080 \
  -p 9080:9080 -p 8000:8000 -v ~/dgraph:/dgraph --name dgraph \
  dgraph/dgraph:v20.03.0 dgraph zero

# In another terminal, now run Dgraph alpha
docker exec -it dgraph dgraph alpha --lru_mb 2048 --zero localhost:5080

# And in another, run ratel (Dgraph UI)
docker exec -it dgraph dgraph-ratel
```

Dgraph Alpha现在将运行在端口8080上监听HTTP请求，而Ratel将在端口8000上监听。还有更多选择，但这就是我们开始所需要的。

### 加载架构 LoadSchema
一旦Dgraph启动并运行，请按右侧面板上的run。 这会将架构加载到Dgraph中，供我们在教程的第一步中使用。 现在不必太担心语法，我们将在以后的课程中介绍所有内容，只需在响应面板中检查操作是否成功即可。

注意
请注意，仅能通过Alter操作来进行模式更改操作。 所有客户端都可以进行Mutate，Query和Alter的操作。

### 载入资料
现在已经加载了架构，是时候加载一些数据了。 再次按下运行按钮，并检查响应面板是否指示数据已成功加载。

暂时不用担心语法，稍后将进行解释。

您可以使用JSON格式加载相同的示例数据。 您可以通过我们的客户端，cURL或Ratel UI进行操作。

请参阅JSON：

```
{
        "set": [
            {
                "uid": "_:michael",
                "name": "Michael",
                "dgraph.type": "Person",
                "age": 39,
                "owns_pet": {
                    "uid": "_:rammy",
                    "dgraph.type": "Animal",
                    "name": "Rammy the sheep"
                },
                "friend": [
                    {
                        "uid": "_:amit"
                    },
                    {
                        "uid": "_:sarah"
                    },
                    {
                        "uid": "_:sang"
                    },
                    {
                        "uid": "_:catalina"
                    },
                    {
                        "uid": "_:artyom"
                    }
                ]
            },
            {
                "uid": "_:amit",
                "name@hi": "अमित",
                "name@bn": "অমিত",
                "name@en": "Amit",
                "dgraph.type": "Person",
                "age": 35,
                "friend": [
                    {
                        "uid": "_:michael"
                    },
                    {
                        "uid": "_:sang"
                    },
                    {
                        "uid": "_:artyom"
                    }
                ]
            },
            {
                "uid": "_:luke",
                "name@pl": "Łukasz",
                "name@en": "Luke",
                "dgraph.type": "Person",
                "age": 77
            },
            {
                "uid": "_:artyom",
                "name@ru": "Артём",
                "name@en": "Artyom",
                "dgraph.type": "Person",
                "age": 35
            },
            {
                "uid": "_:sarah",
                "name": "Sarah",
                "dgraph.type": "Person",
                "age": 55
            },
            {
                "uid": "_:sang",
                "name@ko": "상현",
                "name@en": "Sang Hyun",
                "dgraph.type": "Person",
                "age": 24,
                "owns_pet": {
                    "uid": "_:goldie",
                    "dgraph.type": "Animal",
                    "name": "Goldie"
                },
                "friend": [
                    {
                        "uid": "_:amit"
                    },
                    {
                        "uid": "_:catalina"
                    },
                    {
                        "uid": "_:hyung"
                    }
                ]
            },
            {
                "uid": "_:hyung",
                "name@en": "Hyung Sin",
                "name@ko": "형신",
                "dgraph.type": "Person",
                "friend": {
                    "uid": "_:sang"
                }
            },
            {
                "uid": "_:catalina",
                "name": "Catalina",
                "dgraph.type": "Person",
                "age": 19,
                "owns_pet": {
                    "uid": "_:perro",
                    "dgraph.type": "Animal",
                    "name": "Perro"
                },
                "friend": {
                    "uid": "_:sang"
                }
            }
        ]
    }
```
提示：JSON示例可能最终可以帮助您更好地理解RDF中的格式。

### 突变简介 Mutation
在上一个练习中，您向Dgraph添加了一些数据。在Dgraph中添加或删除数据称为突变操作(Mutation, 开始翻译这个词的时候，有异变，突变的意思，没有普通数据库的CRUD操作，所以想了很久还是觉得突变更有意思，就像是突然间图变化了一下)。我们有两种类型的突变标准格式：RDF（资源描述框架）N-Quad和JSON（JavaScript对象表示法）。 RDF是图或本体系统(Ontology)中广泛使用的标准。

在Dgraph中，突变操作由两种模式组成：无UID引用或显式UID引用。

下面的简介对于以后的练习很重要，此处介绍的许多概念将在接下来的几章中重复进行。 （例如，[添加数据-突变数据](https://dgraph.io/tour/schema/2/)和练习：[整合现有数据](https://dgraph.io/tour/schema/6/)。）

### 空白UID/Blank UID
在突变操作中，任何没有UID显式引用的定义都可以视为空白UID引用，也称为空白节点。空白节点的结构包括：

下划线+冒号+唯一名称（标识符）

例如，在上一节中，您看到了空白节点_：michael和_：amit。这是正确的空白节点语法。但是，在某些情况下，解析器会将某些拼写错误视为空白节点，但不建议使用它。下面的例子：
```
<_：uid43>或<_：uid4e030f>
<someTypoMistake>
```
Dgraph的[数据导出](https://docs.dgraph.io/deploy/#export-database)使用显示的第一种形式的空白节点。

在JSON中，空白节点格式相似。空白节点是通过JSON对象的“ uid”键定义的：
```
{
  "uid": "_:diggy",
  "name": "diggy",
  "dgraph.type": "Food",
  "food": "pizza"
}
```

空白节点在[突变文档](https://docs.dgraph.io/mutations/#blank-nodes-and-uid)中有更详细的介绍。

```
注意
在Dgraph的批量加载器中，任何拼写错误都将被视为空白的UID引用。因为批量加载器是一种用于从头开始填充数据的工具。与Live Loader不同。
```

### 显式的UID
要对现有节点进行突变操作，您可以直接在突变中引用该节点的UID。语法很简单：

```
<THE-UID> # or 
<0x4e030f> <somePredicate> "some new data" .
```
在JSON中：
```
{
  "uid": "0x4e030f",
  "somePredicate": "some new data"
}
```
知道现有节点的UID的一种方法是简单地查询索引谓词，例如：
```
{
  MyUser（func：eq（name，“ Jack Torrance”））{
    uid
  }
}
```

注意
您只能引用现有的UID。如果使用未分配的UID编写突变操作，则Dgraph将返回错误。对于新节点，请对Dgraph使用的空白节点来分配新的UID。

### Graphs
我们已经开始使用Dgraph，但首先要考虑的是：什么是图，它与数据库有什么关系？

图是描述对象及其之间的关系。许多人应该都听说过友谊图或社交网络图，所以让我们从这里开始。

```
注意
好的，您必须在这里发挥您的想象力。我们仍在改进教程，并且还没有可视化相关的东西。通过启动Dgraph，您将使可视化组件正常运行。很快，它将与本教程集成。如果需要可视化，则必须在另一个选项卡中运行它并剪切并粘贴查询。
```
按运行查询。 （您将在此处看到JSON输出。但是在Dgraph UI中运行相同的查询可以同时可视化的查看结果。）在结果图中，圆圈表示人和宠物，线条表示它们之间的连接或带标签的关系。看看是否可以找到Michael和他的宠物羊Rammy。

图天然就适合可视化，您会发现自己一直在用图和可视化方式在思考。

在图中，对象（或实体）称为节点，而关系称为边或谓词。

图形不仅适用于社交网络。也可以有其他例子：

- 互连数据，例如需要联接的SQL表
- 高级搜索
- 推荐引擎
- 模式检测
- 网络，例如计算机，道路和电信
- 流程，例如业务流程和生物流程
- 事件及其之间的因果关系或其他联系
- 公司或市场的结构

就是这些图以及更多应用程序促使我们进入了图数据库这一行业。

### 图数据库
图数据库是为存储和查询图而优化的数据库。 在关系方面的操作上，图数据库比SQL数据库要快得多。

SQL数据库不太擅长类似图的数据操作，因为边意味着连接表Join；表越大，边越多，就意味着更多的联接Join，需要加载和处理的数据就越多。

在图数据库中，边是基本结构，因此沿着边处理是单个查找，这样该操作会非常快。

Dgraph是一个图数据库。 它是针对存储图数据及快速查询优化的。 不仅如此，它还可以在许多机器上处理分布式的图。

使用Dgraph，您可以将具有数百万条边的图存储在单台计算机上，也可以存储需要整个数据中心才能进行处理的海量图。

### 恭喜啦
恭喜，您已经完成了本教程的第一个模块。

到现在为止，您应该已经跟随教程启动并运行了Dgraph，并且对图有所了解。

让我们再研究一些图表并开始编写查询。
