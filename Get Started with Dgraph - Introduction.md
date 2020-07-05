# 欢迎来到Dgraph入门

Dgraph是一个开源的，支持事物的，分布式的，原生的Graph数据库。 这是使用Dgraph的入门系列的第一个教程。

在本教程中，我们将学习如何在Dgraph上构建以下图模型，

![简单图](https://dgraph.io/docs//images/tutorials/1/gs-1.JPG)

在此过程中，我们将了解：

+ 使用dgraph/standalone 的docker映像运行Dgraph。
+ 使用Dgraph的UI Ratel运行以下基本操作，
+ - 创建一个节点。
+ - 在两个节点之间创建一条边。
+ - 查询节点。

[您可以在下面看到相应的视频](https://youtu.be/u73ovhDCPQQ)
很遗憾，是基于Youtube的。只能大家自己想办法了。

# 运行Dgraph

运行dgraph/standalone docker映像是Dgraph入门的最快方法。 此standalone映像仅用于快速入门目的。 不建议在生产环境中使用。

确保Docker已在您的计算机上安装并运行。

现在，只需运行以下命令，即可启动并运行Dgraph了。
```
docker run --rm -it -p 8000:8000 -p 8080:8080 -p 9080:9080 dgraph/standalone:latest
```

# 结点与边 Nodes 与 Edges
在本部分中，我们将构建一个简单的图，其中包含两个节点和一条连接它们的边。

![如图](https://dgraph.io/docs//images/tutorials/1/gs-1.JPG)

在图数据库中，概念或实体表示为节点。 它可能是销售，交易，地点或个人，所有这些实体都在图数据库中表示为节点。

一条边表示两个节点之间的关系。 上图中的两个节点代表着 人：Karthic和Jessica。 您还可以看到这些节点具有两个关联的属性：名称和年龄。 节点的这些属性在Dgraph中称为谓词。

Karthic 与（Jessica）是跟随(Follows)的关系。 他们之间的边(Follows)代表了他们之间的关系。 连接两个节点的边在Dgraph中也称为谓词(predicates)，在Dgraph中，使用它来从一个点指向另一个节点，而不是字符串或整数。

dgraph/standalone 映像是附带着名为Ratel的Dgraph UI。 只需从浏览器访问http:// localhost:8000，便可以访问它。
![如图](https://dgraph.io/docs//images/tutorials/1/gs-2.png)
我们选择Ratel的最新的稳定版(Latest Stable)
![如图](https://dgraph.io/docs//images/tutorials/1/gs-3.png)

## Ratel中的突变(Mutations)

Dgraph中的创建，更新和删除操作称为突变(Mutations), 我的理解就是图发生了变化。

通过Ratel，可以更轻松地运行查询和变异。 在本教程系列中，我们将探索其更多功能。

让我们转到“突变”标签，然后将以下突变粘贴到文本区域。 暂时不要执行！
```
{
  "set": [
    {
      "name": "Karthic",
      "age": 28
    },
    {
      "name": "Jessica",
      "age": 31
    }
  ]
}
```
上面的突变操作会创建两个节点，每一个对应于与“set”关联的JSON值。 但是，它不会在这些节点之间创建边。

对该突变进行一点修改即可修改该突变，可以在它们之间创建一条边。
```
{
  "set": [
    {
      "name": "Karthic",
      "age": 28,
      "follows": {
        "name": "Jessica",
        "age": 31
      }
    }
  ]
}
```
![如图](https://dgraph.io/docs//images/tutorials/1/explain-query.JPG)
我拉执行一下这个突变，点击 Run看看吧。
![如图](https://dgraph.io/docs//images/tutorials/1/mutate-example.gif)

您可以在响应中看到已经创建了两个UID（Universal IDentifier, 唯一ID）。 响应中的“uids”字段中的两个值对应于为“ Karthic”和“ Jessica”的两个节点。

## 使用has函数查询
现在，让我们运行一个查询来可视化一下刚刚创建的节点。 我们将使用Dgraph的has函数。 表达式是 has（name）, 它返回所有与谓词 name 关联的节点。
```
{
  people(func: has(name)) {
    name
    age
  }
}
```
这次转到“查询”标签（Query），输入上面的查询语句。 然后，单击屏幕右上方的“运行” Run。
![如图](https://dgraph.io/docs//images/tutorials/1/query-1.png)

Ratel将会以图形可视化的形式呈现结果。

只需单击它们中的任何一个，就会注意到，节点已经被分配了UID，这些ID与我们在突变响应中看到的UID是一样的。

您还可以在右侧的JSON选项卡中查看返回的JSON结果。
![如图](https://dgraph.io/docs//images/tutorials/1/query-2.png)

## 理解一下查询
![如图](https://dgraph.io/docs//images/tutorials/1/explain-query-2.JPG)

查询的第一部分是用户定义的函数名称。 在查询中，我们将其命名为people。 但是，您可以使用任何其他名称。

func参数必须与Dgraph的内置函数相匹配。 Dgraph提供了各种内置函数。 has是其中之一。 查看查询语言指南，可以了解有关Dgraph中其他内置函数的更多信息。

查询的内部字段类似于SQL select语句中的列名称或GraphQL查询！

您可以轻松指定要返回的谓词(Node的属性)。
```
{
  people(func: has(name)) {
    name
  }
}
```
类似的，你可以用has函数得到所有的有age属性的结点
```
{
  people(func: has(age)) {
    name
  }
}
```
## 灵活的架构schema
Dgraph没有强制要先创建一个结构或架构。 相反，您可以先开始输入数据并根据需要添加约束。

让我们看一下这种突变操作。
```
{
  "set": [
    {
      "name": "Balaji",
      "age": 23,
      "country": "India"
    },
    {
      "name": "Daniel",
      "age": 25,
      "city": "San Diego"
    }
  ]
}
```
我们创建了两个节点，而第一个节点具有谓词，name, age, country，第二个节点具有name，age和city。

最初是不需要架构。 Dgraph会在您的突变中出现时创建新的谓词。 这种灵活性可能是很有用的，但是如果您希望强制突变操作遵循给定的模式，那么我们将在下一个教程中探讨具体的可用的选项。

## 总结
在本教程中，我们学习了Dgraph的基础知识，包括如何运行数据库，添加新节点和谓词以及查询。

在结束之前，这里有一些有关下一个教程的快速介绍。 您知道给定节点的UID也可以获取它们吗？ 它们还可以用于在即存节点之间创建边！

听起来不错吧？