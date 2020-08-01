## 你好，世界 Hello World!
```
注意
要运行以下查询，您应该已在[简介](https://dgraph.io/tour/intro/3/)中加载了数据。
```

让我们先看一下GraphQL+-中的hello world查询吧。

每个查询都有一个名称，并且结果用相同的名称标记返回。

搜索条件func: ...来匹配节点。函数eq可以满足您的要求，比如匹配名称等于“ Michael”的节点。结果是匹配的节点和这些节点列出的边。

Dgraph使用唯一的内部ID（即其UID）标识每个节点。 UID不是边，但可以在查询中返回。如果您已经知道节点的UID，则可以使用func: uid(<uid-number>)对其进行检索。

可以尝试的操作：更改查询以选择其他人。您可以再次看看[样本数据](https://dgraph.io/tour/intro/4)。


在数据中尝试找到一个名称
```
{
  find_someone(func: eq(name, "...name in here...")) {
    name
    age
  }
}
```
或查看数据中的UID并尝试

```
{
  find_someone(func: uid(...uid number here...)) {
    name
    age
  }
}
```
您可以编辑查询并尝试任何想要的操作。当有这样的答案时，请将其剪切并粘贴到查询面板中，然后编辑并运行它们。

本教程针对如何处理查询操作，因此仅关注查询。您还可以了解有关[查询](https://docs.dgraph.io/get-started/#step-3-run-queries)或使用Dgraph进行[编程](https://docs.dgraph.io/clients/)的信息。

### 图形作为结果
上一个查询为我们提供了一些数据，但结果不是很有趣，也与图无关。

在Dgraph和GraphQL+-中，查询返回图，而不是表或数据列表。

对图执行查询，结果是查询的图的子集，或者是基于查询图的某种操作或计算。

让我们改进查询以获取Michael和他的朋友网络。

还可以尝试的：迈克尔还拥有一只宠物，找到了它，并找到了它的名字。 owns_pet边缘在有用哦。
```
{
  michaels_pet(func: eq(name@., "Michael")) {
    name
    age
    friend {
      name@.
    }
    owns_pet {
      name
    }
  }
}
```
### 数据类型，架构和类型系统
现在该讨论数据类型和节点

类型系统
从1.1版开始，Dgraph支持类型系统。目前，类型系统是很基本的，但已经可以用于对节点进行分类并根据其类型查询它们。在扩展查询期间也可以使用类型系统。

因此为了扩展或删除节点。您需要正确定义类型。您可以在此处详细了解类型系统。

数据类型和架构(Schema)
在上一个查询中存在一些魔术，因为一些查询到的边返回了一个值（名称和年龄），而另一些则是其他具有其他属性的节点（friend和owns_pet）的边。

事实是，有一种模式或者说架构Schema告诉我们如何解释边。

这次可视化用处不大。看一下JSON，以了解示例的架构。

图中有两种节点，我们称它们为节点和值（或文字）。在该示例中，代表人的节点有一个名称边，其值为字符串值，有一个年龄的边，其值为int值。值不能包含任何边。

这是可用于在Dgraph中存储值的类型。

|图类型|说明|
|--|--|
|default|默认类型（它是一个中性类型）|
|int|有符号64位整数|
|float|双精度浮点数|
|string|字符串|
|bool|布尔|
|dateTime| RFC3339时间格式，带有可选的时区，例如：2006-01-02T15：04：05.999999999 + 10：00或2006-01-02T15：04：05.999999999|
|geo|使用go-geom存储的几何形状|
|password|密码加密值|


  节点也有个uid类型。该模式告诉我们，朋友边是从一个节点到另一个节点，而不是一个值。

本教程后面的内容中有许多关于[模式](https://dgraph.io/tour/schema/1)的内容。以后，我们对图的结构及其中的事物类型会了解更多。


### 语言支持
Dgraph支持的是UTF-8编码格式的字符串文本和查询。

字符串值谓词可以使用注释来标所用语言。

如: 阿米特（Amit）的名字以英语"Amit"@en，印地语"अमित"@hi和孟加拉语"অমিত"@bn来存储。迈克尔的名字用英语存储。 Artyom的以英语和俄语"Артём"@ru存储。Sang Hyun以英语和韩语"상현"@ko存储。

查询可以通过指定要搜索的语言和要返回的语言来搜索带语言标记的文本。语法@lang1:...:langN，使用的是以下规则指定返回语言的优先顺序：

 - 最多返回一个结果
 - 如果首选语言中的结果存在，则返回这些语言中最左侧（在首选项列表中）
 - 如果首选语言中没有结果，则不会返回任何结果，除非首选项列表以"."终止，在这种情况下，将返回没有指定语言标签的那个值，或者都没有，则返回“某种”语言的值回来。
  
可以尝试将name@ko:ru 更改为name@ko:ru:. 重新查询一下朋友的名字。

### 查询描述图
Dgraph查询结果是图。 实际上，结果结构是与查询结构匹配。

查询中的大括号edge_name { ... }表示嵌套的块，其中块内的边与通过该块的边找到的节点匹配。 我们可以从节点的边继续嵌套查询。

虽然不是严格要求，但查询缩进是一种很好的样式。

将下面的JSON结果与查询的结构进行比较一下，例如，friend边与多个节点匹配，并且每个节点都以JSON表示。

可以尝试的方法：迈克尔的朋友的宠物或迈克尔的朋友的朋友如何？

查询迈克尔的朋友的宠物
```
{
  michael_friends_pets(func: eq(name@., "Michael")) {
    name
    age
    friend {
      name@.
      owns_pet {
        name
      }
    }
  }
}
```
查询迈克尔的朋友的朋友
```
{
  michael_friends_friends(func: allofterms(name@., "Michael")) {
    name
    age
    friend {
      name@.
      friend {
        name@.
      }
    }
  }
}
```

基本上是将查询表述为遍历图，然后沿边到达所需数据。 JSON输出中的uid会解释为图而不是树。 例如，查看迈克尔的朋友的朋友中的循环。

### 函数和过滤
会根据应用于节点出站边的过滤函数来过滤节点。

到目前为止，在查询中仅将过滤器应用于顶级节点，但是过滤器可以应用于查询中的任何节点。

注意，在查询中根结点的过滤与内部块的过滤之间的语法是有差异。

有很多过滤函数，其中一些是

 - allOfTerms(edge_name, "term1 ... termN"): 匹配具有输出字符串的边的edge_name的节点，该字符串包含所有列出的术语。

 - anyOfTerms(edge_name, "term1 ... termN"): 与allOfTerms一样，但只需要至少匹配一个术语。

等于和不等于可以应用于以下类型的边：int，float，string和date

- eq（edge_name，value）：等于
- ge（edge_name，value）：大于或等于
- le（edge_name，value）：小于或等于
- gt（edge_name，value）：大于
- lt（edge_name，value）：小于

此外还有正则表达式，全文搜索和地理搜索，但这些都是较大的主题，在本教程中将有各自的章节。

函数仅适用于已建立索引的谓词，但在过滤器中使用比较函数（eq，ge，gt，le，lt）的情况除外-这是关于[模式](https://dgraph.io/tour/schema/1)的课程的一部分。

### AND, OR and NOT
逻辑连接符 AND, OR 与 NOT 可以连接多个函数到一个过滤器中.

### 排序（升序或降序）
可以使用orderasc和orderdesc对结果进行排序。

虽然可视化效果与未排序的外观相同，但是JSON结果是有序的。

### 分页（first最先，offset偏移和after之后）
查询有成千上万的结果并不少见。

但是您可能只想选择前k个答案，对结果进行分页显示或限制较大的结果。

在GraphQL+-中，这是通过first，offset和after与排序结合完成的。

 - first: N仅返回前N个结果
 - offset: N跳过前N个结果
 - offset: N 返回uid之后的结果
默认情况下，查询答案是按uid排序的。 您可以通过[显式指定顺序](https://dgraph.io/tour/basic/8)来更改排序。

当我们只知道返回的最后一个uid，而不记录先前检索到的结果数时，after就很有用。

### Count
要统计扇出(outgoing)边的数量，可以用count函数.

### Dgraph搜索如何工作
鉴于您到目前为止所看到的内容，您可能已经了解了这一点，但是还是值得再了解一遍。

Dgraph中的图可能很大，因此开始搜索所有节点的效率是不高的。 Dgraph需要一个开始搜索的地方，即根节点root node。

从根本上讲，我们使用func: 和一个函数来查找初始节点集。到目前为止，我们已经使用eq和allofterms进行字符串搜索，但是我们还可以搜索其他值，例如日期，数字以及计数过滤器。

Dgraph需要在建立索引的基础上，以这种方式来搜索，因为如果没有索引来提高搜索效率，Dgraph将不得不遍历整个数据库以找到匹配的值。

从根过滤器中匹配的节点开始，Dgraph会沿着边来获得满足查询的其余部分。根内部块上的过滤器仅应用于随后找到的边可以到达的节点。

func: 仅接受单个函数，不接受过滤器中的AND，OR和NOT连接形式。因此，当需要对根函数返回的元素进行进一步过滤时，需要使用语法query_name(func: foo(...)) @filter(... AND ...) {...}。

可以尝试的方法：查找所有20岁到30岁间且拥有至少2个朋友的人。
```
{
  lots_of_friends(func: ge(count(friend), 2)) @filter(ge(age, 20) AND lt(age, 30)) {
    name@.
    age
    friend {
        name@.
    }
  }
}
```
### has
函数has（edge_name）会返回具有给定名称的扇出边的节点。

### Alias别名
输出图可以使用别名为输出的边设置名称。

### 级联cascade
@cascade指令会删除查询中所有没有匹配所有条件的边的节点。

另一个用途是删除块内的过滤后不返回结果的节点。

在下面的查询中，Dgraph返回所有Michael的朋友，无论他们是否拥有宠物。
```
{
  michael_friends_with_pets(func: allofterms(name@., "Michael")) {
    name
    age
    friend {
      name@.
      owns_pet
    }
  }
}
```
通过@cascade指令，可以过滤没有宠物的朋友
```
{
  michael_friends_with_pets(func: allofterms(name@., "Michael")) @cascade {
    name
    age
    friend {
      name@.
      owns_pet
    }
  }
}
```
可以在[说明文档](https://dgraph.io/docs/query-language/#cascade-directive)中尽一步了解此指令 。

### 归一化 Normalize
@normalize指令

 - 仅返回列出有别名的边，并且

 - 展平结果以消除嵌套

```
提示
别名可以与原始边相同。
```
### 注释Comments
查询是可以包含注释的。

一行中＃号之后的所有内容均为注释，在查询处理中将被忽略。

这对于调试查询和需要在线解释查询部分的教程很有帮助，在本教程后面的更复杂查询中，我们将会增加注释。

### 构面：边属性 Facets：Edge attributes
Dgraph支持构面（边上的键值对），作为对RDF三元组的扩展。即，构面将属性添加到边而不是节点上。例如，两个节点之间的朋友边可能具有亲密友谊的布尔属性。面也可以用作边的权重。

您可以在我们的文档中找到有关Facets的更多详细信息：https://docs.dgraph.io/query-language/#facets-edge-attributes

阅读有关Facet的所有文档，您将获得以下示例：

 - 标量谓词上的面
 - 面的别名
 - UID谓词上的面
 - 筛选面
 - 使用构面排序
 - 将面的值分配给变量
 - 面和变量的传播
 - 面和统计汇总
有关使用JSON的面(Facets)操作的信息，请参见：https://docs.dgraph.io/mutations/#facets

在运行查询之前，请在您的终端上运行此突变：

```
curl -sH "Content-Type: application/rdf" "localhost:8080/mutate?commitNow=true" -XPOST -d $'
{
  set {

    # -- Facets on scalar predicates
    _:alice <name> "Alice" .
    _:alice <mobile> "040123456" (since=2006-01-02T15:04:05) .
    _:alice <car> "MA0123" (since=2006-02-02T13:01:09, first=true) .

    _:bob <name> "Bob" .
    _:bob <car> "MA0134" (since=2006-02-02T13:01:09) .

    _:charlie <name> "Charlie" .
    _:dave <name> "Dave" .


    # -- Facets on UID predicates
    _:alice <friend> _:bob (close=true, relative=false) .
    _:alice <friend> _:charlie (close=false, relative=true) .
    _:alice <friend> _:dave (close=true, relative=true) .


    # -- Facets for variable propagation
    _:movie1 <name> "Movie 1" .
    _:movie2 <name> "Movie 2" .
    _:movie3 <name> "Movie 3" .

    _:alice <rated> _:movie1 (rating=3) .
    _:alice <rated> _:movie2 (rating=2) .
    _:alice <rated> _:movie3 (rating=5) .

    _:bob <rated> _:movie1 (rating=5) .
    _:bob <rated> _:movie2 (rating=5) .
    _:bob <rated> _:movie3 (rating=5) .

    _:charlie <rated> _:movie1 (rating=2) .
    _:charlie <rated> _:movie2 (rating=5) .
    _:charlie <rated> _:movie3 (rating=1) .
  }
}' | python -m json.tool
```
将多个属性添加到第三个单个节点
有时，您需要两个具有不同谓词的Edge指向同一节点。此功能可用于“解释节点的关系”。

假设您有一个现有的Node作为目标，其UID为“0x0f43”。因此，您需要像这样设置RDF。

```
_:MyNode <predicate1> <0x0f43> ( foo=bar, and=1 ) .
_:MyNode <predicate2> <0x0f43> ( bar=bar, and=2 ) .
_:MyNode <predicate3> <0x0f43> ( bar=bar, and=3 ) .
```
更多示例：
```
_:MyNode <Morning> <0x0f43> ( foo=true, and=1 , since=2006-02-02T13:01:09 ) .
_:MyNode <Evening> <0x0f43> ( bar=false, and=2, since=2006-02-02T13:01:09 ) .
_:MyNode <Night>   <0x22>   ( bar=true, and=3,  since=2006-02-02T13:01:09 ) .
```
如果您在上面进行了相同的突变操作，但是使用了相同的谓词而不是唯一的谓词。您将逐行覆盖Edge属性。因此，您必须使用不同的谓词。

恭喜啦
您已完成本课程。

您可以使用本课中的材料来查询图，过滤输出以及对结果进行排序和分页。

可以再次查看部分内容的列表，返回并查看所有不清楚的内容，或继续下一课。
