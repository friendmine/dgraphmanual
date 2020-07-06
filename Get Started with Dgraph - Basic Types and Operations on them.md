欢迎来到Dgraph入门的第三篇教程。

在上一个入门指南中，我们了解了使用UID的CRUD操作。 我们还了解了遍历和递归遍历。

在本教程中，我们将学习Dgraph的基本类型以及如何查询它们。 具体来说，我们将了解：
- Dgraph中的基本数据类型。
- 查询谓词的值。
- 索引。
- 过滤节点。
- 反向递归查询。
您可以在下面看到相应的视频。
[依然是Youtube](https://youtu.be/f401or0hg5E)

让我们首先构建一个简单的博客应用程序的图模型。 这是我们应用程序的图模型

![图模型](https://dgraph.io/docs//images/tutorials/3/a-main-graph.JPG)

上图具有三个实体：作者，博客文章和标签。 图中的节点代表这些实体。 在本教程的其余部分中，我们将代表博客的节点称为博客节点。 并将标签的节点称为标签节点，依此类推。

您可以从图模型中看到这些实体之间的关系：
- 每个作者都有一个或多个博客文章。   

已发表这种边将博客与其作者联系起来。 这些边从作者节点开始，并指向博客节点。

- 每个Blog帖子都有一个或多个标签。

带标签的边将博客与其标签相关联。 这些边从博客节点出现，并指向标签节点。

来建个图吧。

到Ratel，单击“突变”选项卡，粘贴以下突变操作，然后单击“运行”。
```
{
 "set": [
  {
   "author_name": "John Campbell",
   "rating": 4.1,
   "published": [
    {
     "title": "Dgraph's recap of GraphQL Conf - Berlin 2019",
     "url": "https://blog.dgraph.io/post/graphql-conf-19/",
     "content": "We took part in the recently held GraphQL conference in Berlin. The experience was fascinating, and we were amazed by the high voltage enthusiasm in the GraphQL community. Now, we couldn’t help ourselves from sharing this with Dgraph’s community! This is the story of the GraphQL conference in Berlin.",
     "likes": 100,
     "dislikes": 4,
     "publish_time": "2018-06-25T02:30:00",
     "tagged": [
      {
       "uid": "_:graphql",
       "tag_name": "graphql"
      },
      {
       "uid": "_:devrel",
       "tag_name": "devrel"
      }
     ]
    },
    {
     "title": "Dgraph Labs wants you!",
     "url": "https://blog.dgraph.io/post/hiring-19/",
     "content": "We recently announced our successful Series A fundraise and, since then, many people have shown interest to join our team. We are very grateful to have so many people interested in joining our team! We also realized that the job openings were neither really up to date nor covered all of the roles that we are looking for. This is why we decided to spend some time rewriting them and the result is these six new job openings!.",
     "likes": 60,
     "dislikes": 2,
     "publish_time": "2018-08-25T03:45:00",
     "tagged": [
      {
       "uid": "_:hiring",
       "tag_name": "hiring"
      },
      {
       "uid": "_:careers",
       "tag_name": "careers"
      }
     ]
    }
   ]
  },
  {
   "author_name": "John Travis",
   "rating": 4.5,
   "published": [
    {
     "title": "How Dgraph Labs Raised Series A",
     "url": "https://blog.dgraph.io/post/how-dgraph-labs-raised-series-a/",
     "content": "I’m really excited to announce that Dgraph has raised $11.5M in Series A funding. This round is led by Redpoint Ventures, with investment from our previous lead, Bain Capital Ventures, and participation from all our existing investors – Blackbird, Grok and AirTree. With this round, Satish Dharmaraj joins Dgraph’s board of directors, which includes Salil Deshpande from Bain and myself. Their guidance is exactly what we need as we transition from building a product to bringing it to market. So, thanks to all our investors!.",
     "likes": 139,
     "dislikes": 6,
     "publish_time": "2019-07-11T01:45:00",
     "tagged": [
      {
       "uid": "_:annoucement",
       "tag_name": "annoucement"
      },
      {
       "uid": "_:funding",
       "tag_name": "funding"
      }
     ]
    },
    {
     "title": "Celebrating 10,000 GitHub Stars",
     "url": "https://blog.dgraph.io/post/10k-github-stars/",
     "content": "Dgraph is celebrating the milestone of reaching 10,000 GitHub stars 🎉. This wouldn’t have happened without all of you, so we want to thank the awesome community for being with us all the way along. This milestone comes at an exciting time for Dgraph.",
     "likes": 33,
     "dislikes": 12,
     "publish_time": "2017-03-11T01:45:00",
     "tagged": [
      {
       "uid": "_:devrel"
      },
      {
       "uid": "_:annoucement"
      }
     ]
    }
   ]
  },
  {
   "author_name": "Katie Perry",
   "rating": 3.9,
   "published": [
    {
     "title": "Migrating data from SQL to Dgraph!",
     "url": "https://blog.dgraph.io/post/migrating-from-sql-to-dgraph/",
     "content": "Dgraph is rapidly gaining reputation as an easy to use database to build apps upon. Many new users of Dgraph have existing relational databases that they want to migrate from. In particular, we get asked a lot about how to migrate data from MySQL to Dgraph. In this article, we present a tool that makes this migration really easy: all a user needs to do is write a small 3 lines configuration file and type in 2 commands. In essence, this tool bridges one of the best technologies of the 20th century with one of the best ones of the 21st (if you ask us).",
     "likes": 20,
     "dislikes": 1,
     "publish_time": "2018-08-25T01:44:00",
     "tagged": [
      {
       "uid": "_:tutorial",
       "tag_name": "tutorial"
      }
     ]
    },
    {
     "title": "Building a To-Do List React App with Dgraph",
     "url": "https://blog.dgraph.io/post/building-todo-list-react-dgraph/",
     "content": "In this tutorial we will build a To-Do List application using React JavaScript library and Dgraph as a backend database. We will use dgraph-js-http — a library designed to greatly simplify the life of JavaScript developers when accessing Dgraph databases.",
     "likes": 97,
     "dislikes": 5,
     "publish_time": "2019-02-11T03:33:00",
     "tagged": [
      {
       "uid": "_:tutorial"
      },
      {
       "uid": "_:devrel"
      },
      {
       "uid": "_:javascript",
       "tag_name": "javascript"
      }
     ]
    }
   ]
  }
 ]
}
```
现在我们的图准备好了
![如图](https://dgraph.io/docs//images/tutorials/3/l-fullgraph-2.png)

我们的图现在有下面的内容：
- 三个蓝色作者节点。
- 每个作者都有两个博客，共有六个，由绿色节点表示。
- 博客文章的标签为粉红色。您可以看到有8个唯一标签，并且某些博客共享一个公用标签。
 
## 谓词的数据类型
Dgraph自动检测其谓词的数据类型。您可以使用Ratel UI查看自动检测到的数据类型。

单击左侧的架构选项卡，然后检查“类型”列。您会看到谓词名称及其相应的数据类型。

![评级博客评级](https://dgraph.io/docs//images/tutorials/3/a-initial.png)

这些数据类型包括string，float以及int和uid。除了它们，Dgraph还提供了另外三种基本数据类型：geo，dateTime和bool。

uid类型可以用于表示两个节点之间的谓词。换句话说，它们可以用于表示连接两个节点的边。

您可能已经注意到，published和tagged两个谓词的类型为uid数组（[uid]）。 UID数组表示UID的集合。这用于表示一对多的关系。

例如，我们知道一位作者可以发布多个博客。因此，从一个给定的作者节点可能会出现一个以上的publishedf边。每个指向作者的不同博客文章。

Dgraph的v1.1版本引入了类型系统的功能。此功能可以通过对一个或多个谓词进行分组来创建自定义数据类型。但是在本教程中，我们将只关注基本数据类型。

另外，请注意，索引列中没有任何条目。我们将在很快讨论到索引。

## 查询谓词的值
首先，让我们查询一下所有作者和他的评分。
```
{
  authors_and_ratings(func: has(author_name)) {
    uid
    author_name
    rating
  }
}
```
![如图](https://dgraph.io/docs//images/tutorials/3/a-find-rating-2.png)

如果您对查询的语法结构有任何疑问，请参考教程的第一节。

我们的数据集中共有3位作者。 现在，让我们找到最出色的作者。 我们要查询评分为4.0或更高的作者。

为了实现它，我们需要一种选择满足某些条件（例如，rating> 4.0）的节点的方法。 您可以使用Dgraph的内置比较功能来实现。 这是Dgraph中可用的比较功能列表。

|比较函数名|功能|
| -- | -- |
| eq | 等于 |
| lt | 小于 |
| le | 小于等于 |
| gt | 大于 |
| ge | 大于等于 |

Dgraph中共有五个比较函数。 您可以在查询中将它们中的任何一个与func关键字一起使用。

比较函数有两个参数。 一个是谓词名称，另一个是要比较的值。 这里有一些例子。
| 示例 | 描述 |
| -- | -- |
| func: eq(age, 60) | 年龄等于60 |
| func: gt(likes, 100) | 喜欢超过100个 |
| func: le(dislikes, 10) | 不喜欢小于10个 | 

现在，猜猜我们应该使用那个比较函数来选择评分为4.0或更高的作者节点。

如果您认为它应该大于或等于（ge）函数，那么您是对的！

让我们试一下。
```
{
  best_authors(func: ge(rating, 4.0)) {
    uid
    author_name
    rating
  }
}
```
![如图](https://dgraph.io/docs//images/tutorials/3/b-index-missing.png)

出错了！rating谓词的索引缺失。除非您为其添加索引，否则您无法查询谓词的值。

让我们进一步了解Dgraph中的索引以及如何添加索引吧。

## 在Dgraph中建立索引   
索引可以用于加快对谓词的查询。需要时必须将它们显式添加到谓词中。也就是说，仅当您需要查询谓词的值时，需要建立索尼。

另外，也无需在开始时就添加索引。您可以随时添加它们。

Dgraph提供了不同类型的索引。索引的类型选择取决于谓词的数据类型。

下面包含所有的数据类型和可应用于它们的索引类型的表。

|数据类型|可用的索引类型|
| -- | -- |
|int |int|
|float| float|
|string|hash, exact, term, fulltext, trigram|
|bool|bool|
|geo|geo|
|dateTime|year,month,day,hour| 

只有string和dateTime数据类型才可以选择多个索引类型。

在rating谓词上创建一个索引吧。 Ratel UI使得添加索引变得非常简单。

步骤如下：
- 转到左侧的架构标签。
- 从列表中单击rating谓词。
- 选中右侧“属性” UI中的索引选项, 然后更新一下。
  
![如图](https://dgraph.io/docs//images/tutorials/3/c-add-schema.png)
好了，已经成功加好索引了，再回到前面的查询吧。
![如图](https://dgraph.io/docs//images/tutorials/3/d-rating-query.png)

我们成功查询了评分为4.0或更高的作者节点。

我们还如何获取这些作者的博客呢？

我们已经知道published的边从author节点指向博客节点。 因此，获取author节点的blog很简单。 我们只需要从author节点开始遍历published的边。

```
{
  authors_and_ratings(func: ge(rating, 4.0)) {
    uid
    author_name
    rating
    published {
      title
      content
      dislikes
    }
  }
}
```
![如图](https://dgraph.io/docs//images/tutorials/3/e-rating-blog.png)

要是忘了怎么遍历查询，去看一下前面的文章吧。
同样的，我们可以扩展前面的查询，来获得blog的tags。
```
{
  authors_and_ratings(func: ge(rating, 4.0)) {
    uid
    author_name
    rating
    published {
      title
      content
      dislikes
      tagged {
        tag_name
      }
    }
  }
}
```
![如图](https://dgraph.io/docs//images/tutorials/3/m-four-blogs.png)

注意：作者节点为蓝色，博客文章为绿色，标签为粉色。

我们在结果中有两名作者，四篇博客及其标记。

如果您仔细观察一下结果，就会发现有一条博客有12个不喜欢。
![如图](https://dgraph.io/docs//images/tutorials/3/i-dislikes-2.png)

让我们过滤且仅获取热门博客文章。 我们仅查询不喜欢次数少于10的博客。

为此，我们需要将以下语句表示为对Dgraph的查询：遍历published的边，但仅返回那些不喜欢少于10的博客.

我们还可以在遍历期间过滤节点吗？ 可以！ 我们在下一部分中学习如何做到这一点。

## 过滤遍历
我们可以使用@filter指令过滤遍历的结果。

您可以将Dgraph的任意比较函数与@filter指令一起使用。

您应该使用lt比较函数来过滤不喜欢次数少于10的博客帖子。

这是查询。
```
{
  authors_and_ratings(func: ge(rating, 4.0)) {
    author_name
    rating

    published @filter(lt(dislikes, 10)) {
      title
      likes
      dislikes
      tagged {
        tag_name
      }
    }
  }
}
```
![如图](https://dgraph.io/docs//images/tutorials/3/j-dislike-index-2.png)
糟糕！ 我们忘记为dislikes这个谓语添加索引！

转到“架构”选项卡，找到dislikes这一谓语，然后从UI添加索引。
![如图](https://dgraph.io/docs//images/tutorials/3/g-dislike-index-3.png)
注意：请注意，dislikes谓语是整数类型。

让我们重新运行查询。
![如图](https://dgraph.io/docs//images/tutorials/3/n-three-blogs.png)

现在，结果中只有三个博客。 有12个不喜欢的博客被过滤了。

请注意，博客与一系列标签是相关联的。

让我们运行以下查询，并找到数据库中的所有标签。
```
{
  all_tags(func: has(tag_name)) {
    tag_name
  }
}
```
![如图](https://dgraph.io/docs//images/tutorials/3/o-tags.png)

我们在数据库中获得了所有标签。 我最喜欢的标签是devrel。 你的是啥呢？

在我们的下一部分中，让我们找到所有标记为devrel的博客。

## 查询字符串谓词
tag_name谓词表示标签的名称。 它是字符串类型。 这是获取所有标记为devrel的博客的步骤。
- 查找将tag_name谓词的值设置为devrel的根节点。 我们可以使用eq比较函数来实现。
- 在运行查询之前，请不要忘记在tag_name谓词中添加索引。
- 从节点开始沿tagged的边进行devrel标记的遍历。
首先，向tag_name谓词添加索引。 转到Ratel，从列表中单击tag_name谓词。
![如图](https://dgraph.io/docs//images/tutorials/3/p-string-index-2.png)
您会看到有五个索引选项可以应用于任何字符串谓词上。 fulltext，term和trigram是高级字符串索引。 我们将在下一集中详细讨论它们。

字符串类型索引和比较函数的使用存在一些限制。

例如，只有exact的索引与le，ge，lt和gt内置函数兼容。 如果使用任何其他索引设置字符串谓词并运行上述函数，则查询将失败。

虽然，这五个字符串类型索引中的任何一个都与eq函数兼容。 但是hash索引通常是与eq函数一起使用时性能最高的。

让我们将hash索引添加到tag_name谓词中。
![如图](https://dgraph.io/docs//images/tutorials/3/m-hash.png)
让我们用eq这个比较器来获取设定了tag_name是"devrel"的根结点
```
{
  devrel_tag(func: eq(tag_name,"devrel")) {
    tag_name
  }
}
```
![如图](https://dgraph.io/docs//images/tutorials/3/q-devrel-2.png)
我们终于有了我们想要的节点！

我们知道blog节点通过tagged的边连接到其tag节点的。

您是否认为从节点遍历devrel 的 tag 能为我们提供博客？

试试吧！
```
{
  devrel_tag(func: eq(tag_name,"devrel")) {
    tag_name
      tagged {
        title
        content
    }
  }
}
```
该查询似乎有点问题！ 它没有向我们返回博客！ 不用担心，这是意料之中的。

让我们再次观察图模型。
![如图](https://dgraph.io/docs//images/tutorials/3/a-main-graph.JPG)
我们知道Dgraph中的边具有方向性。 您可以看到tagged的边从博客节点指向tag节点。

对于Dgraph而言，沿边方向移动是很自然的。 因此，您可以通过blog的tagged边到tag。

但是要以另一种方式遍历，则需要与边方向相反。 您仍然可以通过在查询中添加波浪号（〜）来实现。 必须在要遍历的边的名称的开头添加波浪号（〜）。

让我们在tagged边的开始处添加波浪号（〜），然后开始进行反向边遍历。
```
{
  devrel_tag(func: eq(tag_name,"devrel")) {
    tag_name

    ~tagged {
      title
      content
    }
  }
}
```
![如图](https://dgraph.io/docs//images/tutorials/3/r-reverse-2.png)
又出错了。
反向遍历也要求在谓词有索引啊。
重新回到Ratel，添加一个 reverse索引到边上中。
![如图](https://dgraph.io/docs//images/tutorials/3/r-reverse-1.png)

再回来重新运行一下吧。
```
{
  devrel_tag(func: eq(tag_name, "devrel")) {
    tag_name

    ~tagged {
      title
      content
    }
  }
}
```
![如图](https://dgraph.io/docs//images/tutorials/3/s-devrel-blogs.png)
![如图](https://dgraph.io/docs//images/tutorials/3/s-devrel-blogs-2.png)

现在，我们获得了所有标记为devrel的博客。

同样，您可以扩展查询以找到这些博客的作者。 它要求您反向遍历published的谓词。

让我们将反向索引添加到published的边。
![如图](https://dgraph.io/docs//images/tutorials/3/t-reverse-published.png)
再运行一下下面的查询
```
{
  devrel_tag(func: eq(tag_name,"devrel")) {
    tag_name

    ~tagged {
      title
      content

      ~published {
        author_name
      }
    }
  }
}
```
![如图](https://dgraph.io/docs//images/tutorials/3/u-author-reverse-1.png)
![如图](https://dgraph.io/docs//images/tutorials/3/u-author-reverse-2.png)
在之前的查询中，我们只是以相反的顺序遍历了整个图形。 从tag节点开始，我们遍历到了author节点。

## 总结
在本教程中，我们了解了基本类型，索引，过滤和反向边遍历。

在总结之前，先来看看我们的下一个教程。

您知道Dgraph提供高级文本搜索功能吗？ 地理位置的查询功能如何？

听起来不错吗？
