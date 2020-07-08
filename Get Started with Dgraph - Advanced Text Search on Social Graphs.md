欢迎来到Dgraph入门第六篇教程。

在前面的一教程中，我们通过对Twitter进行建模来了解如何在Dgraph中构建社交图。我们使用了Hash和exact索引查询了这些推文，并且做了基于关键字的搜索来用术语(term)索引查找您喜欢的推文。

在本教程中，我们继续上次的内容，来了解一下Dgraph中的高级文本搜索功能。

具体来说，我们将重点介绍两个高级功能：

- 使用全文搜索搜索推文。
- 使用正则表达式搜索来搜索主题标签。

`该教程的相应视频还没有发布。`

在深入探讨之前，让我们快速回顾一下如何在Dgraph中对Twitter进行建模。

![推特模型](https://dgraph.io/docs//images/tutorials/5/a-graph-model.jpg)

在上一份教程中，我们以三个真实的推文为样本数据集，并使用上图作为模型将其存储在Dgraph中。

如果您还没将上一教程中的推文存储到Dgraph中，请这次用下面的使用示例数据集。

首先复制下面的突变操作，转到突变选项卡，然后单击运行。
```
{
  "set": [
    {
      "user_handle": "hackintoshrao",
      "user_name": "Karthic Rao",
      "uid": "_:hackintoshrao",
      "authored": [
        {
          "tweet": "Test tweet for the fifth episode of getting started series with @dgraphlabs. Wait for the video of the fourth one by @francesc the coming Wednesday!\n#GraphDB #GraphQL",
          "tagged_with": [
            {
              "uid": "_:graphql",
              "hashtag": "GraphQL"
            },
            {
              "uid": "_:graphdb",
              "hashtag": "GraphDB"
            }
          ],
          "mentioned": [
            {
              "uid": "_:francesc"
            },
            {
              "uid": "_:dgraphlabs"
            }
          ]
        }
      ]
    },
    {
      "user_handle": "francesc",
      "user_name": "Francesc Campoy",
      "uid": "_:francesc",
      "authored": [
        {
          "tweet": "So many good talks at #graphqlconf, next year I'll make sure to be *at least* in the audience!\nAlso huge thanks to the live tweeting by @dgraphlabs for alleviating the FOMO😊\n#GraphDB ♥️ #GraphQL",
          "tagged_with": [
            {
              "uid": "_:graphql"
            },
            {
              "uid": "_:graphdb"
            },
            {
              "hashtag": "graphqlconf"
            }
          ],
          "mentioned": [
            {
              "uid": "_:dgraphlabs"
            }
          ]
        }
      ]
    },
    {
      "user_handle": "dgraphlabs",
      "user_name": "Dgraph Labs",
      "uid": "_:dgraphlabs",
      "authored": [
        {
          "tweet": "Let's Go and catch @francesc at @Gopherpalooza today, as he scans into Go source code by building its Graph in Dgraph!\nBe there, as he Goes through analyzing Go source code, using a Go program, that stores data in the GraphDB built in Go!\n#golang #GraphDB #Databases #Dgraph ",
          "tagged_with": [
            {
              "hashtag": "golang"
            },
            {
              "uid": "_:graphdb"
            },
            {
              "hashtag": "Databases"
            },
            {
              "hashtag": "Dgraph"
            }
          ],
          "mentioned": [
            {
              "uid": "_:francesc"
            },
            {
              "uid": "_:dgraphlabs"
            }
          ]
        },
        {
          "uid": "_:gopherpalooza",
          "user_handle": "gopherpalooza",
          "user_name": "Gopherpalooza"
        }
      ]
    }
  ]
}
```
 
`注意：如果您是Dgraph的新手，并且这是您第一次进行突变操作，我们强烈建议您在继续之前阅读之前的教程。`

现在，您有了一个带有推文，用户和主题标签的图。我们已经准备。

![推特图](https://dgraph.io/docs//images/tutorials/5/x-all-tweets.png)

`注意：如果您想知道如何在Dgraph中对推文进行建模，请参考前面的教程。`

让我们首先使用全文搜索功能查找您最喜欢的推文试下吧。

## 全文搜索
在我们学习如何使用全文搜索功能之前，了解什么时机使用它非常重要。

字符串谓词中单词的长度和数量会根据谓词表示的内容而变化。

一些字符串谓词值中仅包含几个术语（term）。代表names, hashtags, twitter handle, city names的谓词是很好的例子。这些谓词很容易使用它们的精确值进行查询。

例如，这个示例查询。

`给我所有用户名等于John Campbell的推文。`

在将哈希或精确索引添加到字符串谓词上之后，您可以轻松地完成这样的查询。

但是，某些字符串谓词存储的是句子。有时甚至是一个或多个文本数据段。代表推文，个人简介，博客文章，产品说明或电影评论的谓词就是这样的。查询这些谓词相对困难。

使用散列或exact字符串索引查询这样的谓词是不切实际的。使用术语索引的基于关键字的搜索是查询此类谓词的良好起点之一。我们在前面的教程中使用了它来查找与GraphQL，Graphs和Go等关键字完全匹配的推文。

但是，对于某些用例，仅基于关键字的搜索可能还不够。您可能需要更强大的搜索功能，因此您应该考虑使用全文搜索。

试写一些查询并详细了解Dgraph的全文搜索功能吧。

为了能够进行全文搜索，您需要首先在tweet谓词上设置全文索引。

在任何字符串谓词上创建全文索引与创建其他的字符串索引类型都是相似。

![全文](https://dgraph.io/docs//images/tutorials/6/a-set-index.png)

`注意：如果不确定如何在字符串谓词上创建索引，请参考前面的教程。`

现在，让我们进行全文搜索查询，以查找与以下主题相关的推文：graph data and analyzing it in graphdb。

您可以使用alloftext或anyoftext内置函数来实现。这两个函数都有两个参数。第一个参数是要搜索的谓词。第二个参数是要搜索的以空格分隔的字符串，我们将其称为搜索字符串。

-alloftext（谓词，“用空格分隔的搜索字符串”）
-anyoftext（谓词，“用空格分隔的搜索字符串”）
 
稍后，我们将介绍这两个功能之间的区别。现在，我们先使用alloftext函数。

转到查询选项卡，在下面粘贴查询，然后单击运行。这是我们的搜索字符串：[graph data and analyze it in graphdb]。

```
{
  search_tweet(func: alloftext(tweet, "graph data and analyze it in graphdb")) {
    tweet
  }
}
```

![推特图](https://dgraph.io/docs//images/tutorials/6/b-full-text-query-1.png)

这是相符的推文，它的结果出来了。
![如图](./twitter3.png)

如果您仔细观察，您会发现搜索字符串中的某些单词在匹配的推文中不存在，但该推文仍衩添加到结果中了。

为了能够有效地使用全文搜索功能，我们必须了解其工作方式。

让我们详细了解它一下吧。

在推文上设置全文索引后，Dgraph会在内部将处理推文，并生成全文的token会被生成。这些token会被 索引。

搜索字符串也会经过相同的处理，全文token也要被生成的。

以下是生成全文token的步骤：

- 将推文拆分为称为token（token化）的单词块。
- 将这些token转换为小写。
- Unicode标准化token。
- 将token减少到其根形式，这称为词干提取（running到run，faster到fast,等等）。
- 删除停止词。

您已经在第四篇教程中看到，Dgraph允许您构建多语言应用程序的。

但是并非所有语言都支持词干提取和停止词的。[这是指向文档的链接](https://docs.dgraph.io/query-language/#full-text-search)，该文档包含语言列表及其对词干和停止词移除的支持内容。

下面是一份示例的表，第一列是搜索字符串。第二列包含着由Dgraph生成的相应的全文token。

|实际的文本|Token化的文本|
|--|--|
|Let’s Go and catch @francesc at @Gopherpalooza today, as he scans into Go source code by building its Graph in Dgraph!\nBe there, as he Goes through analyzing Go source code, using a Go program, that stores data in the GraphDB built in Go!\n#golang #GraphDB #Databases #Dgraph|[analyz build built catch code data databas dgraph francesc go goe golang gopherpalooza graph graphdb program scan sourc store todai us]|
|graph data and analyze it in graphdb|[analyz data graph graphdb]|

从上表可以看出，这些推文被简化为字符串或token的数组。

Dgraph在内部使用Bleve包进行词干处理。

这是为我们的搜索字符串生成的全文token：[analyz，data，graph，graphdb]。

从上表可以看出，为搜索字符串生成的所有全文token都存在于匹配的tweet中。因此，alloftext函数为该tweet返回一个匹配。但是如果该推文中缺少搜索字符串中的token之一，就不会返回匹配了。但是，只要推文和搜索字符串具有至少一个相同的token，anyoftext函数就会返回匹配的结果。

如果您想了解Dgraph的全文token的实际应用，[请点链接](https://gist.github.com/hackintoshrao/0e8d715d8739b12c67a804c7249146a3)。

即使搜索字符串中的单词顺序不同，Dgraph也会生成相同的全文token。因此，使用具有不同顺序的相同搜索字符串不会影响查询结果。

如您所想，以下所有三个查询对于Dgraph都是相同的。

```
{
  search_tweet(func: alloftext(tweet, "graph analyze and it in graphdb data")) {
    tweet
  }
}
```
```
{
  search_tweet(func: alloftext(tweet, "data and data analyze it graphdb in")) {
    tweet
  }
}
```
```
{
  search_tweet(func: alloftext(tweet, "analyze data and it in graph graphdb")) {
    tweet
  }
}
```

现在，让我们进入Dgraph的下一个高级文本搜索功能吧：基于正则表达式的查询。

让我们用它们来查找包含子字符串graph的所有主题标。

## 正则表达式搜索
正则表达式是表达搜索模式的一种强大方法。 Dgraph允许您基于正则表达式搜索字符串谓词。您需要在字符串谓词上设置三元组(trigram)索引，以便能够执行基于正则表达式的查询。

使用基于正则表达式的搜索，匹配具有此特定模式的所有主题标签：`Starts and ends with any characters of indefinite length, but with the substring graph in it 既任意中间包含graph的字符串`。

这是我们可以使用的正则表达式：^.* graph.*$

如果您对编写正则表达式不熟悉，[请查看本教程](https://www.geeksforgeeks.org/write-regular-expressions/)。

首先，使用has()函数查找数据库中的所有主题标签。

```
{
  hash_tags(func: has(hashtag)) {
    hashtag
  }
}
```

![主题标签](https://dgraph.io/docs//images/tutorials/6/has-hashtag.png)

`如果您不熟悉has()函数的使用，请参阅本系列的前面的教程。`

您可以看到我们总共有六个主题标签，其中四个具有子字符串graph：Dgraph，GraphQL，graphqlconf，graphDB。

我们应该使用内置函数regexp来使用正则表达式搜索谓词。此函数有两个参数，第一个是谓词的名称，第二个是正则表达式。

这是regexp函数的语法：regexp（predicate，/ regular-expression/）

让我们执行以下查询，以查找具有子字符串graph的主题标签。

转到查询选项卡，键入查询，然后单击运行。
```
{
  reg_search(func: regexp(hashtag, /^.*graph.*$/)) {
    hashtag
  }
}
``` 

糟糕！我们又有错误！看来我们忘记了在主题标签谓词上设置三元组索引(trigram)。

![主题标签](https://dgraph.io/docs//images/tutorials/6/trigram-error.png)

同样，设置三字母组(trigram)索引类似于设置任何其他字符串索引，让我们对主题标签谓词进行设置。

![主题标签](https://dgraph.io/docs//images/tutorials/6/set-trigram.png)

`注意：如果不确定怎么在字符串谓词上创建索引，请参考前面的教程。`

现在，让我们重新运行regexp查询。

![正则表达式1](https://dgraph.io/docs//images/tutorials/6/regex-query-1.png)


但是结果中只有以下标签：Dgraph和graphqlconf。

这是因为regexp函数默认情况下区分大小写。

在regexp函数的第二个参数的末尾添加字符i，以使其不区分大小写：regexp（predicate，/ regular-expression/i）

![正则表达式2](https://dgraph.io/docs//images/tutorials/6/regex-query-2.png)

现在我们有了四个带有子字符串graph的标签。

让我们修改正则表达式，使其仅匹配具有前缀graph的主题标签。
```
{
  reg_search(func: regexp(hashtag, /^graph.*$/i)) {
    hashtag
  }
}
```

![正则表达式3](https://dgraph.io/docs//images/tutorials/6/regex-query-3.png)

## 总结
在本教程中，我们学习了Dgraph中的全文本搜索和基于正则表达式的搜索功能。

您是否知道Dgraph还提供模糊搜索功能，比如可用于增强电子商务商店中的产品搜索等功能？

让我们在下一个教程中了解模糊搜索吧。

听起来不错吧？