## 添加架构-变更架构
`我更喜欢用架构来对应 Schema的翻译，模式我不太喜欢`
正如我们在前面的课程中所看到的，Dgraph存储了一个描述谓词类型的架构。

当我们想向现有架构添加新数据时，我们可以直接添加它。 但是，如果要在新架构中添加新数据，则有两种选择

 - 添加数据并让Dgraph计算出对应的架构内容，或者
 - 指定架构，然后添加数据

Dgraph可以很好地解决自动计算架构的问题。 查询根结点的函数只能应用于索引后谓词，为此，我们需要指定架构。 查询根结点级别的比较功能（也称为func:）也只能应用于索引谓词，这些谓词也是通过在架构上注释谓词定义而添加的。 从v1.2开始，比较功能现在可以在@filter指令上使用，甚至可以在尚未索引的谓词上使用。

更改架构可以定义数据类型与添加索引。

运行更改架构。 该索引允许应用过滤器功能，例如搜索参与特定行业的所有公司。

添加数据-突变数据
`mutating, mutation经常会被翻译成突变`
现在，该架构已更新完了，我们可以将数据添加为三元组。

Dgraph为节点创建了自己的内部ID，但是我们需要某种方式在输入数据中多次引用同一节点。 _：company1就是这样做的。

从技术上讲，这些是“空白节点”。 他们告诉Dgraph创建一个节点，为其提供内部ID，并确保其使用上的一致性。

上传后，标签 _:company1 在Dgraph中是不存在的，我们无法查询它。 您会在结果中看到Dgraph用内部ID替换了它。 您可以查询此内部ID，请记住func: uid(<uid_number>)。

对存储在Dgraph中的图进行更改, 称为对数据进行突变操作。

您可以使用JSON格式加载相同的示例数据。 您可以通过我们的客户端，cURL或Ratel UI进行相应的操作。

请参阅JSON：

```
    {
        "set": [
            {
                "uid": "_:company1",
                "industry": "Machinery",
                "dgraph.type": "Company",
                "name": "CompanyABC"
            },
            {
                "uid": "_:company2",
                "industry": "High Tech",
                "dgraph.type": "Company",
                "name": "The other company"
            },
            {
                "uid": "_:jack",
                "works_for": { "uid": "_:company1"},
                "dgraph.type": "Person",
                "name": "Jack"
            },
            {
                "uid": "_:ivy",
                "works_for": { "uid": "_:company1"},
                "boss_of": { "uid": "_:jack"},
                "dgraph.type": "Person",
                "name": "Ivy"
            },
            {
                "uid": "_:zoe",
                "works_for": { "uid": "_:company1"},
                "dgraph.type": "Person",
                "name": "Zoe"
            },
            {
                "uid": "_:jose",
                "works_for": { "uid": "_:company2"},
                "dgraph.type": "Person",
                "name": "Jose"
            },
            {
                "uid": "_:alexei",
                "works_for": { "uid": "_:company2"},
                "boss_of": { "uid": "_:jose"},
                "dgraph.type": "Person",
                "name": "Alexei"
            }
        ]
    }
```
```
提示
JSON示例可能最终可以帮助您更好地理解RDF中的格式。
```
### 外部标识符
Dgraph不支持为节点设置外部ID。 如果应用程序要求Dgraph分配节点的UID以外的其他唯一标识符，则必须将这些标识符作为边提供。 用户应用程序要确保此类ID/密钥的唯一性。

可以查看 https://docs.dgraph.io/mutations/#external-ids 来获得更多信息

### 多国语支持
可以用语言标签来应用到字符串或输出上。
```
_:myID <an_edge> "something"@en .
_:myID <an_edge> "某物"@zh-Hans .
```
也可以应用到边的查询上
同样，操作可以用JSON格式。用在客户端，cURL, 或者Ratel UI
比如 
```
 {
        "set": [
            {
                "uid": "_:myID",
                "an_edge@en": "something",
                "an_edge@zh-Hans": "某物"
            }
        ]
    }
```

提示
```
JSON格式可以很好的帮助你理解RDF
```

### 反向边引用
边是有方向的。 查询不能反向遍历边。

但是双向查询有两种选择

 - 将反向边添加到架构，并添加所有反向边数据。

 - 告诉Dgraph始终在架构中使用@reverse关键字存储反向边。

运行模式变异，Dgraph将计算所有反向边。 an_edge的反向边是〜an_edge。

在数据建模方面，某些反向总是很有意义的，例如朋友。 其他（例如boss_of）有时是双向的，但并不总是双向的。

反向边查询
anEdge的反向边是〜anEdge。

在此查询中，我们想知道谁为““CompanyABC”工作，而不必添加额外的的边。 因此，对于特定情况，我们使用反向边。 然后，我们使用别名“ work_here”来区分查询结果。

### 练习：整合现有数据
我们添加了新的架构并加了一些公司数据，但是如何将以前的朋友数据集与此公司数据进行集成呢？

尝试使用以前的变体中的空白节点将是无效的。 因为空白节点不会保留在存储中，因此，当引用以前的突变中创建的节点时，需要使用它的UID。
所以替代下面的 
```
_:sarah <works_for> _:company1 .
```
是
```
<uid-for-sarah> <works_for> <uid-for-company1> .
```
由于Dgraph选择的uid是唯一的，因此我们这次无法为您提供帮助。 使用您的Dgraph实例给您选择的uid，编写一个将公司和朋友数据链接在一起的突变操作。 提示：先前的查询将告诉您uid。

您在此处完成的过程通常可以通过编程方式完成-查询数据以获取uid，确定突变形式，然后分批更新。

### 删除数据
删除突变内可以有三个删除选项。

 - <uid> <edge> <uid>/"value" .删除一个三元组
 - <uid> <edge> * . 删除给定边的所有三元组
 - <uid> * * . 删除给定节点的所有三元组
此处给出的示例并不完整。在您的实例上分配的Uid将是唯一的。试试看；您不会伤害任何人，只是删除他们的朋友。

您可以使用JSON格式执行相同的示例。您可以通过我们的客户端，cURL或Ratel UI进行操作。

参考JSON：
删除给定节点的所有三元组

```
 {
        "delete": [
            {
                "uid": "0xa"
            }
        ]
    }
```
删除给定边的所有三元组
```
 {
        "delete": [
            {
                "uid": "0xa",
                "friends": null
            }
        ]
    }
```

这不会删除其子结点，这意味着在这种情况下，不会删除“朋友结点”。仅删除该关系。

删除关系，然后删除他们的孩子
```
 {
        "delete": [
            {
                "uid": "0x2", # Answer UID.
                "comment": {
                   "uid": "0x3" # Delete relation (edge) with the Comment
            }
            },
            {
                "uid": "0x3" # Delete the actual comment
            }
        ]
    }
```
```
提示
JSON示例可能可以帮助您更好地理解RDF中的格式。
```

### 扩展谓词
expand(...predicates...)用于查询所有给定的谓词，而不是在查询中列出它们。 查询方式

```
expand(_all_)
```

查询返回查询中与该级别匹配的每个节点中的所有边。Expand扩展可以嵌套在此，然后在下一级扩展所有谓词。

稍后我们将看到如何使用带有变量的扩展，来查询特定的一组边。
```
注意
从v1.1版本开始，您将需要在架构中添加类型才能使expand(_all_)正常工作。 在[类型系统](https://docs.dgraph.io/master/query-language/#type-system)中查看更多信息。
```

您也可以使用_reverse_和_forward_作为expand()的参数。

恭喜啦
这是架构模块的结尾。

到目前为止，您正在为本教程运行的Dgraph实例和模块中的课程为您提供了一个可供使用的沙箱。可以尝试进行任何查询或更改，直到您对到目前为止学到的内容感到满意为止。

完成后，让我们迁移至更大的数据集，并查看Dgraph的更多查询语言。