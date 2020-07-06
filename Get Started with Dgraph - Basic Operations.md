## 欢迎使用Dgraph入门第二篇教程。

在上一个入门指南中，我们学习了Dgraph的一些基础知识。 包括如何运行数据库，添加新节点和谓词以及查询。

![如图](https://dgraph.io/docs//images/tutorials/2/graph-1.jpg)

在本教程中，我们将构建上面的图，并详细了解使用节点的UID（通用标识符）进行的操作。 具体来说，我们将了解：

- 查询和更新节点，使用其UID删除谓词。
- 在现有节点之间添加一条边。
- 向现有节点添加新谓词。
- 遍历图。

您可以在下面看到随附的视频。
[依然是Youtube的](https://youtu.be/8TKD-FFBVgE)

首先，让我们创建图。   
转到Ratel的mutate标签页，将下面的突变操作粘贴到文本区域，然后单击Run。
```
{
  "set":[
    {
      "name": "Michael",
      "age": 40,
      "follows": {
        "name": "Pawan",
        "age": 28,
        "follows":{
          "name": "Leyla",
          "age": 31
        }
      }
    }
  ]
}
```
![如图](https://dgraph.io/docs//images/tutorials/2/a-add-data.gif)

## 使用UID查询
节点的UID可用于结点的查询。 内置函数uid将UID列表作为可变参数，因此您可以根据需要传递一个（例如uid（0x1））或任意多个（例如uid（0x1，0x2））。

无论它们是否存在于数据库中，它都会返回作为输入传递的相同UID。 但是，仅当UID及其谓词都存在时，才会返回所要求的谓词。

让我们看看uid函数的操作。

首先，让我们找到Michael对应节点的UID。

在Ratel 界面中转到查询选项，在下面的查询中键入下面的查询语句，然后单击运行。
```
{
  people(func: has(name)) {
    uid
    name
    age
  }
}
```
现在，从结果中复制Michael节点的UID。
![如图](https://dgraph.io/docs//images/tutorials/2/b-get-uid-1.png)
在下面的查询中，将占位符MICHAELS_UID替换为刚复制的UID，然后运行查询。
```
{
    find_using_uid(func: uid(MICHAELS_UID)){
        uid
        name 
        age
    }
}
```
![如图](https://dgraph.io/docs//images/tutorials/2/c-query-uid.png)
注意：MICHAELS_UID在图中显示为0x8。 您在计算机上获得的UID可能具有不同的值。

您可以看到uid函数返回了与Michael节点的UID匹配的节点。

如果您对查询语句的结构有疑问，请参考上一教程。

## 更新谓词
您还可以使用其UID更新节点的一个或多个谓词。

Michael最近庆祝了自己的41岁生日。 让我们将他的Age更新为41岁。

转到mutate标签并执行突变操作。 同样，不要忘记将占位符MICHAELS_UID替换为Michael节点的实际UID。
```
{
  "set":[
    {
      "uid": "MICHAELS_UID",
      "age": 41
    }
  ]
}
```
我们之前使用set来创建新节点。 但是在使用现有节点的UID时，它会更新其谓词，而不是创建新节点。

您可以看到Michael的年龄已更新为41。
```
{
    find_using_uid(func: uid(MICHAELS_UID)){
        name 
        age
    }
}
```
![如图](https://dgraph.io/docs//images/tutorials/2/d-update-check.png)
同样，您也可以将新谓词添加到现有节点。 由于Michael所在的节点不存在谓词国家，因此它将创建一个新的谓词国家。
```
{
  "set":[
    {
      "uid": "MICHAELS_UID",
      "country": "Australia"
    }
  ]
}
```
## 在现有节点之间添加边
您还可以使用现有节点的UID在现有节点之间添加边。

如，让Leyla开始关注Michael。

我们知道，它们之间的这种关系必须通过在它们之间创建Follows边来表示。
![如图](https://dgraph.io/docs//images/tutorials/2/graph-2.jpg)

首先，让我们从Ratel复制Leyla和Michael的节点的UID。

现在，将占位符LEYLAS_UID和MICHAELS_UID替换为您复制的占位符，然后执行突变操作。
```
{
  "set":[
    {
      "uid": "LEYLAS_UID",
      "follows": {
        "uid": "MICHAELS_UID"
      }
    }
  ]
}
```
## 遍历边
图数据库提供许多独特的功能。 遍历就是其中之一。

遍历会回答与节点之间的关系有关的问题或查询。 因此，诸如Michael Follow 谁 的查询？ 通过遍历以下关系可以得到答案。

让我们运行一下遍历查询，然后详细了解它。
```
{
    find_follower(func: uid(MICHAELS_UID)){
        name 
        age
        follows {
          name 
          age 
        }
    }
}
```
下面是结果
![如图](https://dgraph.io/docs//images/tutorials/2/e-traversal.png)

该查询分为三个部分：

- 选择根节点。
首先，您需要选择一个或多个节点作为遍历的起点。 这些称为根节点。 在上面的查询中，我们使用uid（）函数选择为Michael对应的节点作为根节点。

- 选择要遍历的边
您需要从选定的根节点开始指定要遍历的边。 然后，遍历沿着这些边行进，从一端的结果到另一端的节点。

在我们的查询中，我们选择从Michael的节点开始遍历Follows边。 遍历返回通过Follows边连接到Michael的节点的所有节点。

- 指定要返回的谓词
由于Michael仅Follow一个结点，因此遍历仅返回一个节点。 这些是2级节点。 根节点构成1级的节点。 同样，我们需要指定您要从2级节点取回的谓词。
![如图](https://dgraph.io/docs//images/tutorials/2/j-explain.JPG)

您可以使用2级节点来扩展查询，并进一步遍历该图。 让我们在下一部分中进行探讨。

## 多级遍历
遍历的第一级是Michael Follows的人。 而下一个遍历级别，会进一步返回了他们依次Follows的人。

该模式可以重复多次以实现多级遍历。 当我们遍历图的每个级别时，每次查询的深度增加一级。 那就是我们说的查询很深的时候！
```
{
  find_follower(func: uid(MICHAELS_UID)) {
    name 
    age 
    follows {
      name
      age
      follows {
        name 
        age
      }
    }
  }
}
```
![如图](https://dgraph.io/docs//images/tutorials/2/f-level-3-traverse.png)

下面就是我们的另一个扩展的例子
```
{
  find_follower(func: uid(MICHAELS_UID)) {
    name 
    age 
    follows {
      name
      age
      follows {
        name 
        age
        follows {
          name 
          age
        }
      }
    }
  }
}
```
![如图](https://dgraph.io/docs//images/tutorials/2/g-level-4-traversal.png)

这个查询真的很长！ 该查询有四个层次。 换句话说，查询的深度为四。 如果您要问，难道没有内置的功能可以使多层深度查询或遍历变得容易吗？

答案是肯定的！ 这就是recurse（）函数的作用。 让我们在下一部分中进行探讨。

## 递归遍历
递归查询使执行多级深度遍历更加容易。 它们使您可以轻松遍历图的子集。

使用以下递归查询，我们获得与上一个查询相同的结果。 但是，具有更好的查询体验。
```
{
  find_follower(func: uid(MICHAELS_UID)) @recurse(depth: 4) {
    name 
    age
    follows
  }
}
```
在上面的查询中，递归函数从Michael的节点开始遍历图。 您可以选择任何其他节点作为起点。 depth参数指定遍历查询应考虑的最大深度。

在将占位符替换为Michael的节点的UID之后，运行递归遍历查询。

![如图](https://dgraph.io/docs//images/tutorials/2/h-recursive-traversal.png)

## 边有方向
Dgraph中的边具有方向性。

例如，从Michael节点出现的Follows边指向Pawan的节点。 他们有方向的概念。

对于Dgraph而言，沿边的方向移动是很自然的。 在我们的下一个教程中，我们将学习如何反向遍历边。

## 删除谓词
可以使用delete突变操作删除节点的谓词。 这个是delete突变的语法，用于删除节点的任何谓词，
```
{
    delete {
        <UID> <predicate_name> * .
    }
}
```
使用上面的突变语法，让我们编写一个删除突变操作。 让我们删除Michael节点的年龄谓词。
```
{
  delete {
    <MICHAELS_UID> <age> * .
  }
}
```

## 总结一下
在本教程中，我们学习了使用UID进行CRUD的操作。 我们还了解了recurse（）函数。

在总结之前，先来看看我们的下一个教程。

您知道您可以根据谓词的值来搜索谓词吗？

听起来不错？