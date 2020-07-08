欢迎来到Dgraph入门第七篇教程。

在上一篇教程中，我们通过对Twitter进行建模来了解如何在Dgraph中的社交图上构建高级文本搜索。我们使用全文本和Trigram索引查询了这些推文，并在这些推文上实现了全文搜索和正则表达式搜索。

在本教程中，我们将继续使用第五篇和第六篇教程中的twitter模型探索Dgraph的字符串查询功能。特别是，我们将使用Dgraph的模糊搜索功能实现Twitter用户名的搜索。

`该教程的随附视频暂无。`

在深入探讨之前，让我们回顾一下在前两个教程中如何对推文进行建模：

![推特模型](https://dgraph.io/docs//images/tutorials/5/a-graph-model.jpg)

我们使用了三个真实的示例推文作为样本数据集，并使用上述图作为模型将其存储在Dgraph中。

如果您跳过了先前的教程，那么这里再次给出示例数据集。复制下面的突变操作，转到突变选项卡，然后单击运行。
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
`注意：如果您是Dgraph的新手，并且这是您第一次进行突变，我们强烈建议您在继续之前阅读该系列的前面的教程。`

现在，您应该有了一个带有推文，用户和主题标签的图，我们可以进行研究了。

![推特图](https://dgraph.io/docs//images/tutorials/5/x-all-tweets.png)

`注意：如果您想知道我们如何在Dgraph中对推文进行建模，请参阅第五篇教程。`

在向您展示模糊搜索之前，让我们首先了解它是什么以及它如何工作。

## 模糊搜索
要提供对产品或用户名的搜索功能，需要搜索与字符串最接近的匹配项（如果不存在完全匹配项的话）。即使有错字或用户没有根据存储的确切名称进行搜索，这个功能也可以帮助您获得相关的结果。这正是模糊搜索的作用：比较字符串值并返回最接近的匹配项。因此，对于我们在Twitter的用户名上进行搜索的用例来说，这是理想的选择。

所有的模糊搜索是基于用户名与搜索字符串的Levenshtein distance 来计算的。

Levenshtein距离是一个度量，它定义了两个字符串的距离。两个单词之间的Levenshtein distance是将一个单词转换为另一个单词所需的最小单字符编辑（插入，删除或替换）次数。

例如，book与back之间的Levenshtein distance为2。2是因为通过更改两个字符，我们将单词本改为书本。

现在，您已经了解了模糊搜索是什么以及它可以做什么。接下来，让我们学习如何在Dgraph中的字符串谓词上使用它。

## 在Dgraph中使用模糊搜索
若要对Dgraph中的字符串谓词使用模糊搜索，请首先设置三字母组(trigram)索引。

转到“架构”选项卡，并在user_name谓词上设置三元组(trigram)索引。

在user_name谓词上设置了Trigram索引后，您可以使用Dgraph的内置函数match运行模糊搜索查询。

这是match函数的语法：match（谓词，搜索字符串，距离）

match函数采用三个参数：

- 用于查询的字符串谓词的名称。
- 用户提供的搜索字符串
- 一个整数，代表前两个参数之间的最大Levenshtein distance。该值应大于0。例如，当整数为8时，返回谓词的距离值小于或等于8。
为distance参数使用更大的值可能会匹配更多的字符串谓词，但是也会产生不那么准确的结果。

在使用匹配功能之前，首先获取存储在数据库中的用户名列表。
```
{
    names(func: has(user_name)) {
        user_name
    }
}
```
![如图](https://dgraph.io/docs//images/tutorials/7/e-names.png)

从结果中可以看到，我们有四个用户名：Gopherpalooza，Karthic Rao，Francesc Campoy和Dgraph Labs。

首先，我们将Levenshtein Distance参数设置为3。我们希望看到Dgraph返回与提供的搜索字符串相距三个或更短距离的所有用户名谓词。

然后，我们将第二个参数（用户提供的搜索字符串）设置为graphLabs。

转到查询标签，在下面粘贴查询，然后单击运行。
```
{
    user_names_Search(func: match(user_name, "graphLabs", 3)) {
        user_name
    }
}
```
![如图](https://dgraph.io/docs//images/tutorials/7/h-one.png)

我们有一个预期的结果！ 由于搜索字符串graphLabs与Dgraph Labs的谓词值之间的距离为2，因此我们在搜索结果中看到它。

如果您想了解更多有关如何查找两个单词之间的Levenshtein Distance的信息，[那么这里是一个有用的网站](https://planetcalc.com/1721/)。

让我们再次运行上述查询，但是这次我们将使用搜索字符串graphLab。 转到查询标签，在下面粘贴查询，然后单击运行。
```
{
    user_names_Search(func: match(user_name, "graphLab", 3)) {
        user_name
    }
}
```
![如图](https://dgraph.io/docs//images/tutorials/7/i-two.png)
我们得到了user_name谓词相匹配的Dgraph Labs的结果！ 这是因为搜索字符串graphLab与Dgraph Labs的谓词值之间的距离为三，所以我们在搜索结果中看到了它。

在这种情况下，搜索字符串graphLab和谓词Dgraph Labs之间的Levenshtein距离为3，因此匹配。

对于最后一次查询，我们将搜索字符串更改为Dgraph，但将Levenshtein Distance保持为3。
```
{
    user_names_Search(func: match(user_name, "Dgraph", 3)) {
        user_name
    }
}
```
![如图](https://dgraph.io/docs//images/tutorials/7/j-three.png)

现在，您不会看到Dgraph Labs出现在搜索结果中，因为单词Dgraph与Dgraph Labs之间的距离大于3。但是根据正常的人类基本原理，您自然希望Dgraph Labs出现在搜索结果中，同时将Dgraph用作搜索字符串。

这是基于Levenshtein Distance算法的模糊搜索的缺点之一。模糊搜索的有效性随着距离参数值的减小而降低，并且随着字符串谓词中包含的单词数量的增加而降低。

因此，不建议对可能包含许多单词的字符串谓词使用模糊搜索，例如，用于存储博客文章，简历，产品描述等值的谓词。因此，使用模糊搜索的理想候选者是谓词，例如名称，邮政编码，地点，其中字符串谓词中的单词数通常在1-3之间。

同样，基于用例，调整距离参数对于模糊搜索的有效性至关重要。

## 模糊搜索用于评分
在Dgraph，我们致力于改善分布式Graph数据库的全面功能。作为我们最近为改善数据库功能所做的努力之一，我们注意到一位社区成员对Github提出的要求，该要求整合基于tf-idf的文本搜索。这种集成将进一步增强Dgraph的搜索功能。

我们将在产品路线图中优先解决该问题。我们想借此机会对我们的用户社区表示感谢，感谢他们帮助我们改进了产品。

## 总结
模糊搜索是一种适用范围广泛的，简单而有效的搜索技术。除了查询和搜索字符串谓词的现有功能外，基于tf-idf的搜索的添加还将进一步提高Dgraph的功能。

到这里是我们三个教程的结尾，即使用推文的图模型探索字符串索引及其查询。
