# 多国语支持
欢迎来到Dgraph入门第四篇教程。

在前面的教程中，我们了解了Dgraph中的数据类型，索引，过滤和反向遍历。

在本教程中，我们将学习如何使用多语言字符串以及如何使用语言标签。

您可以在下面看到相应的视频。
[依然是Youtube](https://youtu.be/_lDE9QXHZC0)


## 字符串和语言
Dgraph中的字符串值采用UTF-8格式。 Dgraph还支持有多种语言的字符串谓词的值。多国语功能对于构建某此要求您以多种语言存储相同的信息的功能特别有用。

让我们进一步了解它们吧！

首先，我们从建立一个简单的食物评论图开始。这是图模型。

![模型](https://dgraph.io/docs//images/tutorials/4/a-graph-model.jpg)

上图具有三个实体结点：食物，评论和国家。

图中的节点代表这些实体。

在本教程的其余部分中，我们将代表食品的节点称为食品节点。表示评论的评论节点为评论节点，表示原产国的节点为国家节点。
这是它们之间的关系：
- 每个食品都通过评论边连接到其评论。
- 每个食品都通过原产地边连接到其原产国。
我们先为一些很棒的菜加一些评论！

在我们这样做之前如何加一点料呢？

让我们以原产国的母语为这些食物添加评论先吧。
```
{
  "set": [
    {
      "food_name": "Hamburger",
      "review": [
        {
          "comment": "Tastes very good"
        }
      ],
      "origin": [
        {
          "country": "United states of America"
        }
      ]
    },
    {
      "food_name": "Carrillada",
      "review": [
        {
          "comment": "Sabe muy sabroso"
        }
      ],
      "origin": [
        {
          "country": "Spain"
        }
      ]
    },
    {
      "food_name": "Pav Bhaji",
      "review": [
        {
          "comment": "स्वाद बहुत अच्छा है"
        }
      ],
      "origin": [
        {
          "country": "India"
        }
      ]
    },
    {
      "food_name": "Borscht",
      "review": [
        {
          "comment": "очень вкусно"
        }
      ],
      "origin": [
        {
          "country": "Russia"
        }
      ]
    },
    {
      "food_name": "mapo tofu",
      "review": [
        {
          "comment": "真好吃"
        }
      ],
      "origin": [
        {
          "country": "China"
        }
      ]
    }
  ]
}

```
注意：如果您不熟悉这些突变语法，请参考第一个教程以学习Dgraph中的突变操作基础。

这是我们的图！
![如图](https://dgraph.io/docs//images/tutorials/4/a-full-graph.png)

这个图有几项内容：
- 五个蓝色食物节点。
- 绿色节点代表这些食品的原产国。
- 食品的评论为粉红色。
您还可以看到Dgraph已经自动检测到谓词的数据类型了。 您也可以从“架构”标签中看到它们。

![全图](https://dgraph.io/docs//images/tutorials/4/c-schema.png)

注意：请查看前面的教程，以了解有关Dgraph中数据类型的更多信息。

让我们写一个查询来获取所有的食品，它们的评论和原产国。

转到查询选项卡，粘贴下面的查询，然后单击运行。
```
{
  food_review(func: has(food_name)) {
    food_name
      review {
        comment
      }
      origin {
        country
      }
  }
}
```
提示: 如果不太熟，可以看前面的遍历操作。
下面是只获得评论的查询操作
```
{
  food_review(func: has(food_name)) {
    food_name
      review {
        comment
      }
  }
}
```
像你想像的，下面的评论各种语言都有
![如图](https://dgraph.io/docs//images/tutorials/4/b-comments.png)
但是我们可以根据评论的语种来获取评论吗？ 我们可以写一个查询说：嘿Dgraph，您能给我只用中文写的评论吗？

这是可行的，但前提是您提供有关字符串数据语言的其他信息。 您可以使用语言标签来实现。 使用突变操作添加字符串数据时，可以使用language标签指定字符串谓词的语言。

让我们看看实际的语言标签怎么用吧！

听说寿司很好吃！ 那就用一种以上的语言为Sushi添加评论。 我们将以三种不同的语言撰写评论：英语，日语和俄语。

这是这样做的突变操作。
```
{
  "set": [
    {
      "food_name": "Sushi",
      "review": [
        {
          "comment": "Tastes very good",
          "comment@jp": "とても美味しい",
          "comment@ru": "очень вкусно"
        }
      ],
      "origin": [
        {
          "country": "Japan"
        }
      ]
    }
  ]
}
```
让我们仔细看看如何为不同语言的comment谓词分配具体的值。

我们使用语言标签（@ru，@jp）作为comment谓词的后缀。

在上面的突变操作中：

-- 我们使用@ru语言标签在俄语中添加了注释：“ comment@ru”：“оченьвкусно”。

- 我们使用@jp语言标记在日语中添加注释：“ comment@jp”：“とても美味しい”。

- 英文注释未加标签：“comment”：“口味很好”。

在上面的突变操作中，Dgraph为评论创建一个新节点，并将comment，comment@ru和comment@jp存储在同一节点内的不同谓词中。

注意：如果您不清楚诸如谓词之类的基本术语，请阅读第一篇教程。

让我们运行以上突变操作。

转到“突变”标签，粘贴突变语句，然后单击“运行”。
![如图](https://dgraph.io/docs//images/tutorials/4/d-lang-error.png)

我们又出错了！使用language标记要求您将@lang指令添加到架构(Schema)中。

请按照以下说明将@lang指令添加到comment谓词上。

- 转到“架构”选项卡。
- 单击comment谓词。
- 勾选lang指令。
- 单击更新按钮。
![如图](https://dgraph.io/docs//images/tutorials/4/e-update-lang.png)

让我们重新运行突变操作。

![如图](https://dgraph.io/docs//images/tutorials/4/f-mutation-success.png)

成功！

同样，请记住，使用上述突变操作，我们为Sushi只添加了一个评论，而不是三个不同的评论！

但是，如果您想添加三个不同的评论，请按以下步骤进行。

以以下格式添加评论会创建三个节点，每个节点有评论一个。但是，仅当您添加新评论时才执行此操作，而不是用不同语言代表同一评论。
```
"review": [
  {
    "comment": "Tastes very good"
  },
  {
    "comment@jp": "とても美味しい"
  },
  {
    "comment@ru": "очень вкусно"
  }
]
```
Dgraph允许将任何字符串用作语言标签。 但是，强烈建议仅对语言标签使用ISO标准代码。

通过遵循该标准，您无需将标签传达给您的团队或将其记录在某个地方。 单击[此处](https://www.w3schools.com/tags/ref_language_codes.asp)以查看语言标签的ISO标准代码列表。

在我们的下一部分中，将在查询中使用语言标签。

## 使用语言标签查询。
先仅获取Sushi的评论。

在上一篇文章中，我们了解了如何使用eq运算符和哈希索引来查询字符串谓词的值。

利用这些知识，我们首先为food_name谓词添加哈希索引。

![散列索引](https://dgraph.io/docs//images/tutorials/4/g-hash.png)

现在，转到“查询”选项卡，将查询粘贴到文本区域，然后单击“运行”。
```
{
  food_review(func: eq(food_name,"Sushi")) {
    food_name
      review {
        comment
      }
  }
}
```
![如图](https://dgraph.io/docs//images/tutorials/4/h-comment.png)
默认情况下，查询仅返回未标记的注释。

但是您可以使用language标签专门查询给定语言下的评论。

让我们查询一下日语的Sushi的评论吧。
```
{
  food_review(func: eq(food_name,"Sushi")) {
    food_name
    review {
      comment@jp
    }
  }
}
```
![如图](https://dgraph.io/docs//images/tutorials/4/i-japanese.png)
再试一个俄语的评论。
```
{
  food_review(func: eq(food_name,"Sushi")) {
    food_name
    review {
      comment@ru
    }
  }
}
```
![如图](https://dgraph.io/docs//images/tutorials/4/j-russian.png)
也可以把所有的语言评论都弄回来。
```
{
  food_review(func: eq(food_name,"Sushi")) {
    food_name
    review {
      comment@*
    }
  }
}
```
![如图](https://dgraph.io/docs//images/tutorials/4/k-star.png)
下面是使用各种关于语言标签的语法表格.
| 语法| 结果|
| -- | -- |
| comment |查找无标记的字符串； 如果不存在未加标签的评论，则不返回任何内容。|
| comment@.| 寻找未加标签的字符串（如果找不到），然后以任何语言返回评论。 但是，这仅返回一个值。|
|comment@jp|查找标记为@jp的评论。 如果未找到，查询将不返回任何内容。|
|comment@ru|查找标记为@ru的评论。 如果未找到，查询将不返回任何内容。|
|name@jp:.| 首先寻找标有@jp的评论。 如果找不到，找到未加标签的注释。 如果也找不到，用一个存在的语言返回的评论。|
|name@jp:ru|查找标记为@jp的评论，然后为@ru。 如果均未找到，则不返回任何内容。|
|name@jp:ru:.| 寻找标有@jp的评论，然后是@ru。 如果都找不到，找到未加标签的注释。 如果也找不到，返回其他任何注释（如果存在）。|
|name@*|返回所有语言标记，包括未标记的语言。|

如果你还记得的话，我们已经用俄语给 Borscht加了一个评论了。
![如图](https://dgraph.io/docs//images/tutorials/4/l-russian.png)
如果您注意到了，我们还没有使用语言标签@ru进行俄语评论。

因此，如果我们查询所有用俄语撰写的评论，那么针对Borscht的评论就不会进入列表。

只有Sushi的评论（用俄语撰写）才进入列表。

![俄语](https://dgraph.io/docs//images/tutorials/4/m-sushi.png)

到此，就是这次的课程！

`如果您用不同的语言表示相同的信息，请不要忘记添加语言标签！`

## 总结
在本教程中，我们学习了如何使用多语言字符串以及如何使用语言标签对其进行操作。

标签的使用不仅限于多语言字符串。 语言标签只是Dgraph标记数据功能的一个用例。

在下一个教程中，我们将继续探索Dgraph中的字符串类型。 我们将详细探讨字符串类型索引。

听起来不错吗？