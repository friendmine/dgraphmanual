## 更大的数据集
现在，我们从Dgraph和GraphQL+-开始。让我们提前一点儿。

Dgraph可以执行查询聚合，地理查询，字符串查询等。但是，让我们从最初的小型数据集出发，尝试大的，更大的东西。

在我们的github存储库中，您将找到有关电影，导演和演员的[数据集](https://github.com/dgraph-io/tutorial/tree/master/resources/1million.rdf.gz)。

从该链接下载并保存到〜/dgraph目录，或者在终端中运行以下命令

```
cd ~/dgraph
wget "https://github.com/dgraph-io/tutorial/blob/master/resources/1million.rdf.gz?raw=true" -O 1million.rdf.gz -q

OR (We have curl in Dgraph's containers)

curl -L -o 1million.rdf.gz "https://github.com/dgraph-io/tutorial/blob/master/resources/1million.rdf.gz?raw=true"
```

使用运行按钮运行架构突变操作，然后从终端将数据集加载到Dgraph中。您可能需要使用--lru_mb 4096重新启动Alpha，才能处理更大的数据集。

```
docker exec -it dgraph dgraph live -f 1million.rdf.gz --alpha localhost:9080 --zero localhost:5080 -c 1
```

数据集中大约有一百万个三元组。 Dgraph会准确地报告加载了多少个三元组以及花费了多长时间。

这是一个庞大的电影数据库，但不会给Dgraph带来麻烦。但是，它足够大到可以让我们练习使用更复杂的查询。

该数据集是较大的[电影数据集](https://github.com/dgraph-io/benchmarks/blob/master/data/21million.rdf.gz)的有一百万个三元组的子集，其中包含大约230000个演员和大约86000个电影。我们制作了一个子集以使其能够快速加载，同时又具有足够的复杂性以产生有趣的结果。但是，由于它是其中的一个子集，您可能会发现自己最喜欢的演员不在那儿，或者某些演员没有相应的某些电影。

在本教程的后面，您可能需要返回此页面（或发出[架构查询](https://dgraph.io/tour/basic/3)）以检查索引或架构类型。请记住，我们无法将函数应用于未建立索引的边，并且我们将了解到某些函数需要特定的索引。

### 电影架构
通过Dgraph查询，架构类型和可视化将帮助我们了解电影数据集的架构。

正如您将在以下页面中看到的那样，我们可以从多个角度查看数据。例如将导演，电影，流派甚至地点作为出发点。

让我们从导演的角度来看一下。凯思琳·比格洛（Kathryn Bigelow）执导了90年代初期的经典《点破 Point Break》。我们将从她开始，来了解该架构。

通过查看类型架构，我们可以知道导演作为“Person类型”有一个名字name，并且与他们通过director.film边与导演的电影相连。

通过扩展以下内容，将查询扩展到更深的层次

```
director.film (first: 1) {     
    name
    initial_release_date
    genre { name }
    starring { Performance }
 }
 ```
我们可以看到电影具有名称name，还具有流派genre和主演starring边，它们关联到该电影所归类的类型以及电影中的人物。

还可以通过研究主演边，您将了解到通过该边到达的节点代表了演员在电影中作为特定角色的出演。

从Tom Hanks开始尝试相同的查询。他既是演员又是导演，因此您将看到电影和演员的演出链接。

请记住，还有一个架构查询，可用于查询架构中的类型和索引。只是

```
schema {}
```
可以查询有关架构的所有信息

如果您愿意，请进行更多学习。到目前为止的课程，您可以找到特定导演的所有电影，演员的电影或电影演员。