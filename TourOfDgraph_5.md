## 多个命名查询块
```
注意
从现在开始，要运行查询，您需要加载那个百万级的电影数据集。
```

查询可以并发的(多个查询 issued as multiples)。

对于标记为q1，...，qn的查询，可作为多个查询块发出，JSON结果将包含每个查询的标记为应答的块q1，...，qn。

如果一个块用var标记，则该查询不返回任何结果。

以这种方式发出的查询是独立的。

此处的可视化视是基于uid在两个查询的并集，但JSON结果包含每个查询的独立应答。

因此，在这种情况下，我们会展现两个不同查询之间的联合的可视化。

查询变量是GraphQL+- 的最强大功能之一。 查询变量允许将一个块中的响应内容用于另一个块中，从而将两个查询结合在一起。 您将在之后了解查询变量。

### 查询变量
结果可以存储在变量中，并可以在查询的其他地方使用。

变量由声明方式如下：
```
var_name as some_block { ... }
```

其中var_name是任何唯一的变量名，some_block可以是整个查询或是边匹配查询的内部块。

一旦定义后，变量可用于：

 - 定义块的子查询中（不是定义的父级）
 - 在另一个查询中
变量在定义时不会影响查询的语义。

查询变量通常是使用成uid(<variable-name>)这种样子。 变量将根据对它们进行定义的块对查询中所有匹配的uid求值。 特别要注意的是，变量是uid列表，而不是与该块匹配的图，并且该变量的值是针对整个查询的所有uid的块，而不是与任何一个分支匹配的uid。

定义变量但不使用它是错误的。

让我们查看一下查询变量的选项。

子块中的查询变量
子块中的查询变量允许查询将与同级匹配的应答向下传递给子对象进行过滤。

例如，在简·坎皮恩（Jane Campion）的所有电影中，演员的集合； 我们的挑战是要找到哪对在非简·坎皮恩（Jane Campion）执导的电影中一起表演的演员。

这里的查询将使用所有Jane Campion电影JC_films的集合以及任何Jane Campion电影JC_actors中所有演员的子集。

@cascade指令可确保答案与查询的所有部分匹配-否则，将返回没有匹配共同角色的参与者。

该查询返回参与者及其自身以及其他参与者的匹配项。

可以使用跨越多个查询的变量来编写相同的查询，但是图遍历是相同的。

### 另一个查询块中的查询变量 I
让我们花点时间考虑一下最后一个查询。 变量JC_actor会有Jane Campion电影中的所有演员。 无论我们在哪里使用它，都是全集。

这就是正确使用Dgraph变量的关键：了解它们是全局的使用的，因为它们会评估可与查询中的该边缘匹配的所有节点，而不是局部的，针对评估每部Jane Campion电影有不同的结果。

我们还可以在另一个查询块中使用查询变量，既可以作为根过滤器匹配的节点，也可以在内部过滤器中使用。

彼得·杰克逊（Peter Jackson）经常出现在自己的电影中，大多是在背景中或只是瞥见而已，但他在那里。 该查询会显示彼得·杰克逊执导并出演的所有电影。

### 另一个查询块中的查询变量II
在其他查询块中使用的查询变量也允许重新组织结果。

### 练习：链接查询的查询变量
我们看到了变量的两种常见用法：过滤和重新组织结果。 另一个方法是将两个查询链接起来，基于合并的结果来获得一些新结果。

让我们看一个连接两个查询的例子。

对于两位导演，可以找到曾与两位导演合作过的演员（不一定是同一部电影）。 许多导演不会有共同的演员，所以一定要从某些导演入手（下面的答案是彼得·杰克逊和马丁·斯科塞斯，他们的演员很少。）

作为一项可选挑战，对于每个演员，列出他们与任一导演制作的电影。
```
{
  var(func: allofterms(name@en, "Peter Jackson")) {
    F_PJ as director.film {
      starring{
        A_PJ as performance.actor
      }
    }
  }

   var(func: allofterms(name@en, "Martin Scorsese")) {
    F_MS as director.film {
      starring{
        A_MS as performance.actor
      }
    }
  }

  actors(func: uid(A_PJ)) @filter(uid(A_MS)) @cascade {
    actor: name@en
    actor.film {
      performance.film @filter (uid(F_PJ, F_MS)) {
      	name@en
      }
    }
  }
}
```
语法 uid(F_PJ, F_MS)的语义是F_PJ与F_MS的合集， 是与uid(F_PJ) OR uid(F_MS) 等效的。

### 值变量-最小值和最大值
我们刚刚研究了存储查询块中uid结果的变量。 值变量存储与其匹配的值。

值变量与查询变量的工作方式不同。 值变量是上下文相关的-实际上，它们是UID到值的映射，我们在读取和过滤值变量时可以使用它。 使用 val(<variable-name>)提取值变量中的值。

在定义时，因为上下文在适当的UID内，所以值变量的作用类似于相应的值。 在封闭块中，值变量是UID到值的映射，必须聚集在一起。

值变量的聚集Aggregation不依赖于索引-无需索引，因为已经在查询中找到了值。

最小值min和最大值max可应用于int，float，string，bool和date类型。

### 练习：值变量-求和和平均值 sum and avg
求和和平均值只能应用于包含int和float数据的值变量。

找出这两个数字哪个更大，史蒂芬·斯皮尔伯格（Steven Spielberg）的电影数量或他导演的电影中的演员平均数量。

该查询可以在单个块中完成，但是如果相应的UID可用于值映射，则您将在这里看到在其他块中如何使用值。

```
{
  ID as var(func: allofterms(name@en, "Steven Spielberg")) {
    director.film {
      num_actors as count(starring)
    }
    average as avg(val(num_actors))
  }

  avs(func: uid(ID)) @normalize {
    name : name@en
    average_actors : val(average)
    num_films : count(director.film)
  }
}
```
查询生效后，请尝试使用错误的UID来更改第二个块，然后查看会发生什么情况。 例如，更改avs以查询其他导演，而不使用ID(变量ID的定义也需要删除)，否则将会出现被定义但不使用的错误。

### 值变量：过滤和排序
如果块的UID提供的上下文正确，则值变量也可以用于过滤和排序。

在这里，ID将是名称为Steven的所有导演的UID，而平均值是从这些UID到每个导演的平均值的映射。 在该上下文中评估var(average)的过滤，排序和结果，以获得每个对应的值。

可以使用值变量代替UID变量，uid(<value-variable>) 会求值到映射中的UID。 例如，avs查询块可以编写为：

```
avs(func: uid(average), orderdesc: val(average)) @filter(ge(val(average), 40)) @normalize {
```

### 值变量：数学函数
除了最小，最大，平均和求和外，Dgraph还支持许多可应用于值变量的函数。 这些需要包含在math(...)中并存储在变量中。

完整列表是：

|OPERATOR	|ACCEPTED TYPE	| NOTES
|--|--|--|
|+ - * / %|	int and float| 	
|min max	|All types except geo and bool	|
|< > <= >= == !=|	All, except geo and bool|	boolean result
|floor ceil``ln exp sqrt|	int and float	|
|since|	date|	number of seconds (float) from the time specified
|pow(a, b)	|int and float|	a^b
|logbase(a,b)|	int and float	|log(a) to the base b
|cond(a, b, c)	|a must be a boolean|	selects b if a is true else c

请注意，这些是基于值的函数，而不是用于汇聚值的映射的函数。 这些函数的结果是值变量，可以用于汇聚的。

### 练习：最新电影
编写查询以查找每个导演最近发行的电影，并按发行日期排序结果。

如果您需要一些提示，请尝试：
为了解决这个问题，您需要

 - 找出如何查询所有导演-导演做了哪些工作使他们成为导演？
 - 查找每个导演的最新电影上映日期
 - 按最新日期对结果进行排序
 - 返回导演姓名，以及最新电影的详细信息
 - 您将需要一个查询以获取导演及其最新电影，而另一个查询则需要对其进行排序以首先获取最新电影。
 - 如果需要额外的帮助，请尝试使用since来算出电影上映后的天数（或某些电影上映后的天数）

```
{ 
  # Get all directors
  var(func: has(director.film)) @cascade {
    director.film {
      date as initial_release_date
    }
    # Store maxDate as a variable
    maxDate as max(val(date))
    daysSince as math(since(maxDate)/(24*60*60))
  }

  # Order by maxDate
  me(func: uid(maxDate), orderdesc: val(maxDate), first: 10) {
    name@en
    days : val(daysSince)

    # For each director, sort by release date and get latest movie.
    director.film(orderdesc: initial_release_date, first: 1) {
      name@en
      initial_release_date
    }
  }
}
```

### 分组
分组查询，根据给定的一组属性（对元素进行分组）汇总查询结果。 例如，包含该块的查询

```
director.film @groupby(genre) {
  a as count(uid)
}
```
找到导演的电影边的节点，根据流派将其分为几组，然后计算每组中有多少个节点。

在groupby块中，仅允许聚合，并且count只能应用于uid。

结果是分组的边和聚合的值变量。

在这种情况下，值变量是从流派到值的映射。

通常，有必要将groupby与另一个查询一起使用以获取除聚合以外的其他值。

恭喜你！
您已完成本课程。

接下来，进行字符串和地理检索。 或使用索引转到另一个主题。