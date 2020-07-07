# 入门-快速入门指南
`注意这是一个快速入门指南。您可以在此处找到入门教程系列。`
```
常用的名词
schema 架构
mutation 突变 或者 突变操作
predicate 谓词
```
## Dgraph
Dgraph是从底层开始设计到产品的，一个开放源代码，可扩展，分布式，高可用性和快速的图数据库。

Dgraph群集由不同的节点（Zero，Alpha和Ratel）组成，并且每个节点都有不同的用途。

- Dgraph Zero结点控制Dgraph群集，将服务器分配给一个组，并在服务器组之间，做数据均衡。

- Dgraph Alpha结点包含了 包含谓词和索引。谓词可以是与节点关联的属性，也可以是两个节点之间的关系。索引是可以与谓词关联的词法分析器，以使用针对谓词的各种的过滤功能。

- Ratel 提供UI服务以运行查询，突变(Mutation)和更改架构(schema)。

运行一个基本的功能，您也需要至少一个Dgraph Zero和一个Dgraph Alpha结点才能开始。

## 这是一个四步教程，可帮助您快速开始。

这是运行Dgraph的快速入门指南。

`提示本指南适用于Dgraph的功能强大的查询语言GraphQL+-，它是Facebook创建的查询语言GraphQL的变体。您可以从 dgraph.io/graphql 中找到有关GraphQL的入门说明。`

### 步骤1：运行Dgraph
有几种安装和运行Dgraph的方法，所有这些方法都可以在“下载”页面中找到。

启动和运行Dgraph的最简单方法是使用dgraph/standalone 的docker映像。如果尚未安装Docker，请按照说明进行安装。

此独立映像仅用于快速入门目的。不建议在生产环境中使用。

`docker run --rm -it -p 8080：8080 -p 9080：9080 -p 8000：8000 -v〜/dgraph：/ dgraph dgraph/standalone：v20.03.0`

这将开始在其中运行Dgraph的Alpha，Zero和Ratel结点在单个容器中。您会发现Dgraph数据存储在个人主目录的名为dgraph的文件夹中。

`提示 通常，您需要通过lru_mb标志来设置Dgraph 的alpha结点可以使用的内存。但这只是Dgraph alpha的提示，实际使用情况会更高。建议将lru_mb设置为可用R内存的三分之一。对于standalone设置，是默认成这样的。`

### 步骤2：执行变异
`提示 一旦Dgraph运行，您可以从http：// localhost：8000访问Ratel。它可以通过浏览器未完成查询，变异和可视化。您可以在命令行中通过curl在命令行中完成，或者在Ratel中粘贴变异数据来运行以下变异和查询。`

### 数据集
数据集是电影的图数据，其中，图节点是导演，演员，类型，或电影的实体。

### 在图中存储数据
更改Dgraph中存储的数据就是一种突变(Mutation)。截至目前，Dgraph支持两种数据的突变：RDF和JSON。以下RDF突变存储了有关“星球大战”系列的前三个发行版和其中一部“星际迷航”电影的信息。通过curl或Ratel UI的mutate tab 运行RDF突变，会将数据存储在Dgraph中。
```
curl -H "Content-Type: application/rdf" "localhost:8080/mutate?commitNow=true" -XPOST -d $'
{
  set {
   _:luke <name> "Luke Skywalker" .
   _:luke <dgraph.type> "Person" .
   _:leia <name> "Princess Leia" .
   _:leia <dgraph.type> "Person" .
   _:han <name> "Han Solo" .
   _:han <dgraph.type> "Person" .
   _:lucas <name> "George Lucas" .
   _:lucas <dgraph.type> "Person" .
   _:irvin <name> "Irvin Kernshner" .
   _:irvin <dgraph.type> "Person" .
   _:richard <name> "Richard Marquand" .
   _:richard <dgraph.type> "Person" .

   _:sw1 <name> "Star Wars: Episode IV - A New Hope" .
   _:sw1 <release_date> "1977-05-25" .
   _:sw1 <revenue> "775000000" .
   _:sw1 <running_time> "121" .
   _:sw1 <starring> _:luke .
   _:sw1 <starring> _:leia .
   _:sw1 <starring> _:han .
   _:sw1 <director> _:lucas .
   _:sw1 <dgraph.type> "Film" .

   _:sw2 <name> "Star Wars: Episode V - The Empire Strikes Back" .
   _:sw2 <release_date> "1980-05-21" .
   _:sw2 <revenue> "534000000" .
   _:sw2 <running_time> "124" .
   _:sw2 <starring> _:luke .
   _:sw2 <starring> _:leia .
   _:sw2 <starring> _:han .
   _:sw2 <director> _:irvin .
   _:sw2 <dgraph.type> "Film" .

   _:sw3 <name> "Star Wars: Episode VI - Return of the Jedi" .
   _:sw3 <release_date> "1983-05-25" .
   _:sw3 <revenue> "572000000" .
   _:sw3 <running_time> "131" .
   _:sw3 <starring> _:luke .
   _:sw3 <starring> _:leia .
   _:sw3 <starring> _:han .
   _:sw3 <director> _:richard .
   _:sw3 <dgraph.type> "Film" .

   _:st1 <name> "Star Trek: The Motion Picture" .
   _:st1 <release_date> "1979-12-07" .
   _:st1 <revenue> "139000000" .
   _:st1 <running_time> "132" .
   _:st1 <dgraph.type> "Film" .
  }
}
' | python -m json.tool | less

```

`提示 要通过curl使用文件来运行RDF/JSON的突变，可以使用curl选项 --data-binary @/path/to/mutation.rdf 而不是 -d $''。 --data-binary选项会跳过curl的默认网址编码。`

### 步骤3：变更架构
更改架构以在某些数据上添加索引，以便查询可以使用术语匹配，过滤和排序。
```
curl "localhost:8080/alter" -XPOST -d $'
  name: string @index(term) .
  release_date: datetime @index(year) .
  revenue: float .
  running_time: int .

  type Person {
    name
  }

  type Film {
    name
    release_date
    revenue
    running_time
    starring
    director
  }
' | python -m json.tool | less
```
`提示 要从Ratel UI提交架构，请转到“架构”页面，单击“批量编辑”，然后粘贴架构。`

### 步骤4：执行查询
获取所有电影
运行此查询以获取所有电影。 该查询列出了所有具有星标关系(Edge)的电影。
```
curl -H "Content-Type: application/graphql+-" "localhost:8080/query" -XPOST -d $'
{
 me(func: has(starring)) {
   name
  }
}
' | python -m json.tool | less
```

`提示 您还可以从Ratel UI的查询标签页中运行GraphQL+- 查询。`

### 获取“1980年”之后发行的所有电影
运行此查询以获取“1980年”之后发行的“星球大战”电影。 请在用户界面中尝试，将查看结果的图。
```
curl -H "Content-Type: application/graphql+-" "localhost:8080/query" -XPOST -d $'
{
  me(func:allofterms(name, "Star Wars")) @filter(ge(release_date, "1980")) {
    name
    release_date
    revenue
    running_time
    director {
     name
    }
    starring {
     name
    }
  }
}
' | python -m json.tool | less
```
结果
```
{
  "data":{
    "me":[
      {
        "name":"Star Wars: Episode V - The Empire Strikes Back",
        "release_date":"1980-05-21T00:00:00Z",
        "revenue":534000000.0,
        "running_time":124,
        "director":[
          {
            "name":"Irvin Kernshner"
          }
        ],
        "starring":[
          {
            "name":"Han Solo"
          },
          {
            "name":"Luke Skywalker"
          },
          {
            "name":"Princess Leia"
          }
        ]
      },
      {
        "name":"Star Wars: Episode VI - Return of the Jedi",
        "release_date":"1983-05-25T00:00:00Z",
        "revenue":572000000.0,
        "running_time":131,
        "director":[
          {
            "name":"Richard Marquand"
          }
        ],
        "starring":[
          {
            "name":"Han Solo"
          },
          {
            "name":"Luke Skywalker"
          },
          {
            "name":"Princess Leia"
          }
        ]
      }
    ]
  }
}
```

到这儿就完事了！ 在这四个步骤中，我们设置了Dgraph，添加了一些数据，设置了一个架构并对该数据 进行了查询操作。