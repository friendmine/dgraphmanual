# 查询语言 Query Language
Dgraph的GraphQL+-基于Facebook的GraphQL的。 GraphQL并不是为Graph数据库开发的，但其类似图的查询语法，模式验证和子图形响应使其成为绝佳的语言选择。我们修改了该语言使其支持图操作，添加和删除了一些功能，以使其更适合图数据库。我们称这种修改后的语言为“GraphQL+-”。

GraphQL+-正在开发中。我们将添加更多功能，并且可能会进一步简化现有功能。

## 教程 -https://tour.dgraph.io
本文档是Dgraph查询参考资料。这不是一份教程。它是为已经知道如何在GraphQL+-中编写查询但需要检查语法，索引，函数等的用户提供的参考。

注意如果您是Dgraph的新手，并且想学习如何使用Dgraph和GraphQL +-，请查询 -[https://tour.dgraph.io](https://tour.dgraph.io)

## 运行示例
本参考资料中的示例使用了有关电影和演员的2,100万个三元组数据库。示例可以查询运行并返回结果。查询由运行在https://play.dgraph.io/的Dgraph实例执行。要在本地运行查询或进行更多实验，请参阅《入门指南》，该指南还显示了如何加载此处示例中使用的数据集。
`尽管本文中附加了许多的执行结果，但是很多结果的中间内容都是被删除过的，因为太多了。如果要看真实的内容，[请参考网站](https://dgraph.io/docs/query-language/)`
## GraphQL+-基础知识
GraphQL+-的每一个查询都是基于搜索条件查找节点、匹配图中的模式并返回图作为结果的。

查询由多个嵌套语法块组成并且从查询根开始的。根节点找到初始的节点集，对其应用图匹配和过滤。

`注意 [在查询设计概念中查看有关查询的更多信息](https://dgraph.io/docs/design-concepts/#queries)`

## 返回值
每个查询都有一个在查询根目录中定义的名称，并且用相同的名称标识来结果。

如果边是值类型，则可以通过指定边的名称来返回值。

查询示例：在示例数据集中，边是将电影链接到导演与演员，电影有名称，发行日期和许多知名电影数据库的标识符。下面的查询的名称是bladerunner，它的根与电影名称匹配，返回的是80年代早期的科幻经典电影《银翼杀手》。
```
{
  bladerunner(func: eq(name@en, "Blade Runner")) {
    uid
    name@en
    initial_release_date
    netflix_id
  }
}
```
查询结果
```
{
  "data": {
    "bladerunner": [
      {
        "uid": "0x394c",
        "name@en": "Blade Runner",
        "initial_release_date": "1982-06-25T00:00:00Z",
        "netflix_id": "70083726"
      }
    ]
  }
}
```

查询首先使用索引对name边等于“Blade Runner”的所有节点进行高效的搜索。 对于找到的节点，查询将返回列出的边。

每个节点都有一个唯一的64位标识符。 上面查询中的uid边返回该标识符。 如果所需的节点是已知，就用函数uid将找到该节点。

查询示例：UID找到的“Blade Runner”电影数据。
```
{
  bladerunner(func: uid(0x394c)) {
    uid
    name@en
    initial_release_date
    netflix_id
  }
}
```
结果
```
{
  "data": {
    "bladerunner": [
      {
        "uid": "0x394c",
        "name@en": "Blade Runner",
        "initial_release_date": "1982-06-25T00:00:00Z",
        "netflix_id": "70083726"
      }
    ]
  }
}
```
一个查询可以返回多个匹配结果。
查询示例：所有的结点都有一个匹配“Blade”或“Runner”的名字。
```
{
  bladerunner(func: anyofterms(name@en, "Blade Runner")) {
    uid
    name@en
    initial_release_date
    netflix_id
  }
}
```
结果
```
{
  "data": {
    "bladerunner": [
      {
        "uid": "0xd5f",
        "name@en": "The Runner"
      },
      {
        "uid": "0x394c",
        "name@en": "Blade Runner",
        "initial_release_date": "1982-06-25T00:00:00Z",
        "netflix_id": "70083726"
      },
      {
        "uid": "0x485d20",
        "name@en": "Blade of Wrath"
      },
      {
        "uid": "0x48f7a3",
        "name@en": "Tokyo Blade: Live in London",
        "initial_release_date": "1985-01-01T00:00:00Z",
        "netflix_id": "70032395"
      }
    ]
  }
}
```
在函数uid中，可以同时使用多个ID
查询示例：
```
{
  movies(func: uid(0xb5849, 0x394c)) {
    uid
    name@en
    initial_release_date
    netflix_id
  }
}
```
结果
```
{
  "data": {
    "movies": [
      {
        "uid": "0x394c",
        "name@en": "Blade Runner",
        "initial_release_date": "1982-06-25T00:00:00Z",
        "netflix_id": "70083726"
      },
      {
        "uid": "0xb5849",
        "name@en": "James Cameron's Explorers: From the Titanic to the Moon",
        "initial_release_date": "2006-01-01T00:00:00Z",
        "netflix_id": "70058064"
      }
    ]
  }
}
```
`注意如果谓词具有特殊字符，则在查询中要求谓词时，应将其用尖括号括起来。 例如。 <first：name>`

## 扩展图边缘
查询是通过使用{}嵌套查询块来在节点上获得更多的边及相关信息的。

查询示例：在《银翼杀手》中扮演的演员和角色。 该查询首先找到名称为“ Blade Runner”的节点，然后扩展到边到达在该片中出演角色的演员节点。 从那里可以将Performance.actor和Performance.character边扩展，以查找电影中每个演员的演员名称和角色。

```
{
  brCharacters(func: eq(name@en, "Blade Runner")) {
    name@en
    initial_release_date
    starring {
      performance.actor {
        name@en  # actor name
      }
      performance.character {
        name@en  # character name
      }
    }
  }
}
```
结果
```
{
  "data": {
    "brCharacters": [
      {
        "name@en": "Blade Runner",
        "initial_release_date": "1982-06-25T00:00:00Z",
        "starring": [
          {
            "performance.actor": [
              {
                "name@en": "John Edward Allen"
              }
            ],
            "performance.character": [
              {
                "name@en": "Kaiser"
              }
            ]
          },
          {
            "performance.actor": [
              {
                "name@en": "Joanna Cassidy"
              }
            ],
            "performance.character": [
              {
                "name@en": "Zhora"
              }
            ]
          },
          {
            "performance.actor": [
              {
                "name@en": "M. Emmet Walsh"
              }
            ],
            "performance.character": [
              {
                "name@en": "Bryant"
              }
            ]
          }
        ]
      }
    ]
  }
}
```
## 注释
＃之后的任何一行都是注释

## 使用过滤器
查询都是从根开始查找到初始节点集，然后通过向其他节点返回值和相应的边，继续查询， 也就是在根中搜索后通过遍历找到查询中可到达的任何节点。 我们可以应用@filter来过滤找到的任何节点，无论是根结点还是随后查找到的。

查询示例：“银翼杀手”的导演雷德利·斯科特（Ridley Scott）指导的在2000年之前发行的电影。
```
{
  scott(func: eq(name@en, "Ridley Scott")) {
    name@en
    initial_release_date
    director.film @filter(le(initial_release_date, "2000")) {
      name@en
      initial_release_date
    }
  }
}
```
结果
```
{
  "data": {
    "scott": [
      {
        "name@en": "Ridley Scott",
        "director.film": [
          {
            "name@en": "Blade Runner",
            "initial_release_date": "1982-06-25T00:00:00Z"
          },
          {
            "name@en": "Alien",
            "initial_release_date": "1979-05-25T00:00:00Z"
          },
          {
            "name@en": "Boy and Bicycle",
            "initial_release_date": "1997-09-07T00:00:00Z"
          }
        ]
      },
      {
        "name@en": "Ridley Scott"
      }
    ]
  }
}
```
再查询一个，名字包含“Blade”或“Runner”的。
```
{
  bladerunner(func: anyofterms(name@en, "Blade Runner")) @filter(le(initial_release_date, "2000")) {
    uid
    name@en
    initial_release_date
    netflix_id
  }
}
```
结果
```
{
  "data": {
    "bladerunner": [
      {
        "uid": "0x394c",
        "name@en": "Blade Runner",
        "initial_release_date": "1982-06-25T00:00:00Z",
        "netflix_id": "70083726"
      },
      {
        "uid": "0x7160",
        "name@en": "Blade",
        "initial_release_date": "1973-12-01T00:00:00Z"
      },
      {
        "uid": "0x48f7a3",
        "name@en": "Tokyo Blade: Live in London",
        "initial_release_date": "1985-01-01T00:00:00Z",
        "netflix_id": "70032395"
      }
    ]
  }
}
```

## 语言支持
`注意必须在架构(Schema)中指定@lang指令，才能在查询或突变谓词中使用语言标签。`
Dgraph是支持UTF-8字符串。

在查询中，对于字符串的edge值的语法是下面这样的

`edge@lang1：...：langN`

使用以下规则可以指定返回语言的顺序。

- 最多将返回一个结果（语言列表设置为*的情况除外）。
- 偏好列表顺序是从左到右考虑的：如果找不到给定语言的值，就会考虑列表中的下一种语言。
- 如果没有找到任何指定语言的值，就不返回任何值。
- 最后[.]表示返回没有指定语言的值的时候，或者如果没有“没有语言的值”时，将返回“某种”语言的值(可能是随机的，也可能不是，需要看代码吧，还是指定语言顺序的好)。
- 将语言列表值设置为*将返回该谓词的所有多语言的值。不带语言标签的值也将返回。   
### 举些例子吧例如：    
- name=>查找未加语言标签的字符串；如果没有未语言标签的值，则不返回任何内容。
- name@. =>优先寻找无标签的字符串，然后寻找任何语言。
- name@en =>寻找带有标签的字符串；如果不存在带有en标签的字符串，则不返回任何内容。
- name@en:. =>优先查找en标签，没有则选择任何语言。
- name@en:pl =>查找en，然后查找pl，否则返回空。
- name@en:pl:. =>查找en，然后查找pl，然后查找未加标签的标签，然后查找任何语言。
- name@* =>查找此谓词的所有值，并将它们连同其语言一起返回。例如，如果有两个带有en和hi语言标签，则此查询将返回两个名为“name@ en”和“name@hi”的值。
```
注意在函数中，是不能使用语言列表（包括@*表示法）的。可以像之前的叙述那样使用无语言标签的谓词，单一语言标签和.标签。
使用全文搜索功能时（alloftext，anyoftext），当未指定任何语言（未标记或@.）时，将使用默认的（英文）分词器。这并不意味着在查询未标记的值时将搜索带有en标记的值，但是未标记的值将被视为英文文本。如果您不希望出现这种情况，请为所需的语言使用适当的标记，以进行突变操作与查询操作。
```
查询示例：宝莱坞导演和演员法尔汗·阿赫塔尔（Farhan Akhtar）的电影中有些用俄语，印地语和英语存储名称，而其他则没有。
看看怎么操作吧。
```
{
  q(func: allofterms(name@en, "Farhan Akhtar")) {
    name@hi
    name@en

    director.film {
      name@ru:hi:en
      name@en
      name@hi
      name@ru
    }
  }
}
```
结果呢
```
{
  "data": {
    "q": [
      {
        "name@hi": "फरहान अख्तर",
        "name@en": "Farhan Akhtar",
        "director.film": [
          {
            "name@ru:hi:en": "दिल चाहता है",
            "name@en": "Dil Chahta Hai",
            "name@hi": "दिल चाहता है"
          },
          {
            "name@ru:hi:en": "Дон. Главарь мафии 2",
            "name@en": "Don 2",
            "name@hi": "डॉन २",
            "name@ru": "Дон. Главарь мафии 2"
          },
          {
            "name@ru:hi:en": "Дон. Главарь мафии",
            "name@en": "Don: The Chase Begins Again",
            "name@hi": "डॉन",
            "name@ru": "Дон. Главарь мафии"
          }
        ]
      }
    ]
  }
}
```

## 函数

函数可以基于节点的属性或变量进行过滤。它可以可以在查询根目录或过滤器中使用。

`注意Dgraph v1.2.0中添加了对非索引谓词上的过滤器的支持。`

查询根结点中使用的（就是 func:）的比较函数（eq，ge，gt，le，lt）只能应用于索引谓词。从v1.2开始，比较函数也可以在@filter指令上使用，甚至可以在尚未索引的谓词上使用。但是对于大型数据集，对非索引谓词进行过滤可能会很慢，因为它们需要在使用过滤器的级别上遍历所有可能的值。

查询根结点或过滤器中的所有其他函数只能应用于索引谓词。

所有处理字符串谓词的函数，如果未给出语言首选项，则该函数将应用于所有没有语言标签的语言和字符串；如果给出了语言选项，则该功能仅应用于给定语言的字符串。

## 术语匹配
### allofterms
语法示例：allofterms(predicate, "space-separated term list")

Schema类型：字符串

所需索引：term

可以匹配任意顺序的具有所有指定术语的字符串；不区分大小写。

### 根结点使用
查询示例：所有name中包含indiana和jones的节点，以英语返回英语名称和流派。
```
{
  me(func: allofterms(name@en, "jones indiana")) {
    name@en
    genre {
      name@en
    }
  }
}
```
结果
```
{
  "data": {
    "me": [
      {
        "name@en": "The Adventures of Young Indiana Jones: Passion for Life",
        "genre": [
          {
            "name@en": "History"
          }
        ]
      },
      {
        "name@en": "The Adventures of Young Indiana Jones: Attack of the Hawkmen",
        "genre": [
          {
            "name@en": "War film"
          },
          {
            "name@en": "Adventure Film"
          }
        ]
      }
    ]
  }
}
```
### 使用过滤器
查询示例：所有包含indiana和jones一词的史蒂芬·斯皮尔伯格的电影。 @filter（has（director.film））删除了不是导演的名字叫史蒂芬·斯皮尔伯格的节点，有的数据还包含电影史蒂芬·斯皮尔伯格的角色。
```
{
  me(func: eq(name@en, "Steven Spielberg")) @filter(has(director.film)) {
    name@en
    director.film @filter(allofterms(name@en, "jones indiana"))  {
      name@en
    }
  }
}
```
结果
```
{
  "data": {
    "me": [
      {
        "name@en": "Steven Spielberg",
        "director.film": [
          {
            "name@en": "Indiana Jones and the Temple of Doom"
          },
          {
            "name@en": "Indiana Jones and the Raiders of the Lost Ark"
          },
          {
            "name@en": "Indiana Jones and the Kingdom of the Crystal Skull"
          },
          {
            "name@en": "Indiana Jones and the Last Crusade"
          }
        ]
      }
    ]
  }
}
```

### anyofterms
语法示例：anyofterms(predicate, "space-separated term list")

模式类型：字符串

所需索引：term

以任何顺序匹配具有任何指定术语的字符串； 不区分大小写。

### 根结点使用
查询示例：名称中包含有poison或peacock的所有节点。 返回的许多节点都是电影，但像Joan Peacock这样的人也符合搜索条件，因为没有级联指令，查询就不需要在genre上处理什么了。
```
{
  me(func:anyofterms(name@en, "poison peacock")) {
    name@en
    genre {
      name@en
    }
  }
}
```
结果
```
{
  "data": {
    "me": [
      {
        "name@en": "The Peacock Spring",
        "genre": [
          {
            "name@en": "Drama"
          }
        ]
      },
      {
        "name@en": "David Peacock"
      },
      {
        "name@en": "Harry Peacock"
      },
      {
        "name@en": "The New York Peacock",
        "genre": [
          {
            "name@en": "Silent film"
          },
          {
            "name@en": "Drama"
          }
        ]
      },
      {
        "name@en": "Peacock Films Ltd."
      },
      {
        "name@en": "Synte Peacock"
      }
    ]
  }
}
```
### 过滤器用法
查询示例：所有包含war或spies的史蒂芬·斯皮尔伯格电影。 @filter（has（director.film））删除了不是导演的名字叫史蒂芬·斯皮尔伯格的节点-有的数据还包含电影史蒂芬·斯皮尔伯格的角色。
```
{
  me(func: eq(name@en, "Steven Spielberg")) @filter(has(director.film)) {
    name@en
    director.film @filter(anyofterms(name@en, "war spies"))  {
      name@en
    }
  }
}
```
结果
```
{
  "data": {
    "me": [
      {
        "name@en": "Steven Spielberg",
        "director.film": [
          {
            "name@en": "War Horse"
          },
          {
            "name@en": "War of the Worlds"
          },
          {
            "name@en": "Bridge of Spies"
          }
        ]
      }
    ]
  }
}
```
## 正则表达式
语法示例：regexp(predicate, /regular-expression/) 或不区分大小写的 regexp(predicate, /regular-expression/i) 

模式类型：字符串

所需索引：trigram

因为整个程序是用go开发的，所以通过正则表达式匹配字符串要有go的正则表达式。

查询示例：在根结点下，将节点的name与开头是'Steven Sp'匹配，后跟任意字符。 对于每个匹配的uid，匹配包含ryan的影片。 注意与allofterms的区别，allofterm仅匹配ryan，但正则表达式搜索也将匹配（如bryan）。
```
{
  directors(func: regexp(name@en, /^Steven Sp.*$/)) {
    name@en
    director.film @filter(regexp(name@en, /ryan/i)) {
      name@en
    }
  }
}
```
结果
```
{
  "data": {
    "directors": [
      {
        "name@en": "Steven Spencer"
      },
      {
        "name@en": "Steven Spielberg",
        "director.film": [
          {
            "name@en": "Saving Private Ryan"
          }
        ]
      },
      {
        "name@en": "Steven Spielberg And The Return To Film School"
      },
      {
        "name@en": "Steven Spohn"
      },
      {
        "name@en": "Steven Spurrier"
      }
    ]
  }
}
```
## 技术细节
Trigram是三个连续符文的子串。例如，Dgraph具有字母Dgr，gra，rap，aph。

为了确保正则表达式匹配的效率，Dgraph使用Trigram索引。也就是说，Dgraph将正则表达式转换为Trigram查询，使用Trigram索引和Trigram查询查找可能的匹配项，并将完整的正则表达式搜索仅应用在可能的匹配项上。

## 编写有效的正则表达式和限制
设计正则表达式查询时，请牢记以下几点。

- 正则表达式必须至少匹配一个三字母组（不支持短于3个符文的模式）。也就是说，Dgraph需要可以转换为三元组查询的正则表达式。
- 正则表达式匹配的三字母组的数量应尽可能小（[a-zA-Z] [a-zA-Z] [0-9]不是一个好主意）。太多可能的正则表达式匹配意味着对字符串做全文匹配；如果表达式能强制执行更多的三元组来匹配，则Dgraph可以更好地利用索引，并针对较小的可能匹配集来完成完整的正则表达式匹配。
- 因此，正则表达式应尽可能精确。匹配更长的字符串意味着需要更多的三字母组合，这有助于有效地使用索引。
- 如果使用重复表达式（*，+，？，{n，m}），那么整个正则表达式不会与空字符串或任意字符串匹配：例如，*可以像[Aa]bcd*那样使用，但不能像（abcd)*或（abcd)|((defg)*）
- 方括号表达式（例如[fgh]{7}，[0-9]+或[a-z]{3,5}）之后的重复规范通常被视为与任何字符串匹配，因为它们会与太多的三字组匹配。
- 如果在索引扫描期间部分结果（用于三字母组的子集）超过1000000个uid，则停止查询以禁止进行如此昂贵的查询。

## 模糊匹配
语法：match(predicate, string, distance)

模式类型：string

所需索引：trigram

通过计算到字符串的Levenshtein距离来匹配谓词值，也称为模糊匹配。距离参数必须大于零（0)。使用较大的距离值可能会产生更多但不太准确的结果。

查询示例：从根结点上开始，模糊匹配节点类似于Stephen，其距离值小于或等于8。
```
{
  directors(func: match(name@en, Stephen, 8)) {
    name@en
  }
}
```
结果
```
{
  "data": {
    "directors": [
      {
        "name@en": "Stephen Chen"
      },
      {
        "name@en": "Step Up 3D"
      }
    ]
  }
}  
```
再查一个Levenshtein distance是3的
```
{
  directors(func: match(name@en, Stephen, 3)) {
    name@en
  }
}
```
结果
```
{
  "data": {
    "directors": [
      {
        "name@en": "Steve"
      },
      {
        "name@en": "Steel"
      }
    ]
  }
}
```
## 全文搜索
语法示例：alloftext(predicate, "space-separated text") 和anyoftext(predicate, "space-separated text")

模式类型：string

所需索引：全文/fulltext

全文搜索是会应用词干和停用词技术进行的，以查找与所有或任何给定文本匹配的字符串。

在索引生成过程中将应用以下步骤并处理全文搜索参数：

1. Token化根据Unicode字边界）。
2. 转换为小写。
3. Unicode正则化（规范化形式为KC）。
4. 使用特定于语言的词干提取器（如果语言支持）。
5. 删除停止单词（如果语言支持）。
Dgraph将bleve用于全文搜索索引。 另请参阅特定于bleve语言的停止词列表。

下表包含所有支持的语言，相应的国家/地区代码，词干和停用词的过滤支持。
|Language|	Country Code|	Stemming|	Stop words|
|--|--|--|--|
|Arabic	|ar	|✓	|✓|
|Armenian|	hy|	|	✓|
|Basque|	eu|	|	✓|
|Bulgarian|	bg|	|	✓|
|Catalan|	ca|	|	✓|
|Chinese|	zh|	✓	|✓|
|Czech|	cs|	|	✓|
|Danish|	da|	✓	|✓|
|Dutch	|nl|	✓|	✓|
|English|	en	|✓|	✓|
|Finnish|	fi|	✓	|✓|
|French|	fr|	✓|	✓|
|Gaelic|	ga|		|✓|
|Galician	|gl	|	|✓|
|German	|de	|✓|	✓|
|Greek|	el	|	|✓|
|Hindi	|hi|	✓	|✓|
|Hungarian|	hu|	✓|	✓|
|Indonesian	|id|	|	✓|
|Italian|	it|	✓	|✓|
|Japanese	|ja|	✓|	✓|
|Korean	|ko|	✓|	✓|
|Norwegian|	no|	✓	|✓|
|Persian|	fa|	|	✓|
|Portuguese	|pt|	✓	|✓|
|Romanian	|ro|	✓	|✓|
|Russian	|ru	|✓|	✓|
|Spanish|	es	|✓|	✓|
|Swedish|	sv|	✓|	✓|
|Turkish|	tr|	✓	|✓|
查询示例：具有dog，dogs，bark，barks，barking等的所有名称。停止词the which 会被删除掉。
```
{
  movie(func:alloftext(name@en, "the dog which barks")) {
    name@en
  }
}
```
结果
```
{
  "data": {
    "movie": [
      {
        "name@en": "Black Dogs Barking"
      },
      {
        "name@en": "Barking Dog"
      },
      {
        "name@en": "Barking Dog"
      }
    ]
  }
}
```
## 等量关系
等于
语法示例：

- eq(predicate, value)
- eq(val(varName), value)
- eq(predicate, val(varName))
- eq(count(predicate), value)
- eq(predicate, [val1, val2, ..., valN])
- eq(predicate, [$var1, "value", ..., $varN])

模式类型：int，float，bool，string，dateTime

需要索引：在查询根结点使用eq（predicate，...）格式（请参见下表）时是需要索引。 对于查询根的count（predicate），需要@count索引。 对于变量，因为值已作为查询的一部分进行计算，因此不需要索引。

|类型|索引选项|
|--|--|
|int|	int|
|float|	float|
|bool|	bool|
|string	|exact, hash|
|dateTime	|dateTime|
测试谓词或变量值的相等性，或在值列表中查找相等的值。

布尔常量是true和false，因此对于eq来说，它变为eq(boolPred, true)。

查询示例：电影类型恰好为十三种。
```
{
  me(func: eq(count(genre), 13)) {
    name@en
    genre {
      name@en
    }
  }
}
```
输出
```
{
  "data": {
    "me": [
      {
        "name@en": "Stay Tuned",
        "genre": [
          {
            "name@en": "Horror"
          },
          {
            "name@en": "Thriller"
          },
          {
            "name@en": "Family"
          }
        ]
      }
  }
}
```
查询示例：导演史蒂文（Steven）导演了1,2或3部电影。
```
{
  steve as var(func: allofterms(name@en, "Steven")) {
    films as count(director.film)
  }

  stevens(func: uid(steve)) @filter(eq(val(films), [1,2,3])) {
    name@en
    numFilms : val(films)
  }
}
```
结果
```
{
  "data": {
    "stevens": [
      {
        "name@en": "Steven Wilsey",
        "numFilms": 1
      },
      {
        "name@en": "Steven Bratter",
        "numFilms": 1
      },
      {
        "name@en": "Steven Saussey",
        "numFilms": 1
      }
    ]
  }
}
```  
小于，小于等于，大于，大于等于
语法示例：不等式IE

- IE(predicate, value)
- IE(val(varName), value)
- IE(predicate, val(varName))
- IE(count(predicate), value)

类似的还有
- le 小于或等于
- lt 小于
- ge 大于或等于
- gt 大于
  
这些可以用于的类型有：int，float，string，dateTime

需要索引：在查询根结点中使用IE(predicate, ...)格式（请参见下表）时需要索引。 对于查询根的count（predicate），需要@count索引。 对于变量，由于值已作为查询的一部分进行计算，因此不需要索引。

|类型|索引选项|
|--|--|
|int |int|
|float|float|
|string|exact|
|dateTime| dateTime|

查询示例：1980年之前发行的Ridley Scott电影。
```
{
  me(func: eq(name@en, "Ridley Scott")) {
    name@en
    director.film @filter(lt(initial_release_date, "1980-01-01"))  {
      initial_release_date
      name@en
    }
  }
}
```
结果
```
{
  "data": {
    "me": [
      {
        "name@en": "Ridley Scott",
        "director.film": [
          {
            "initial_release_date": "1979-05-25T00:00:00Z",
            "name@en": "Alien"
          },
          {
            "initial_release_date": "1977-12-01T00:00:00Z",
            "name@en": "The Duellists"
          }
        ]
      },
      {
        "name@en": "Ridley Scott"
      }
    ]
  }
```
查询示例：导演姓名为Steven，导演超过100位演员的电影。
```
{
  ID as var(func: allofterms(name@en, "Steven")) {
    director.film {
      num_actors as count(starring)
    }
    total as sum(val(num_actors))
  }

  dirs(func: uid(ID)) @filter(gt(val(total), 100)) {
    name@en
    total_actors : val(total)
  }
}
```
结果
```
{
  "data": {
    "dirs": [
      {
        "name@en": "Steven Conrad",
        "total_actors": 122
      },
      {
        "name@en": "Steven Knight",
        "total_actors": 102
      },
      {
        "name@en": "Steven Zaillian",
        "total_actors": 123
      }
    ]
  }
}
```
查询示例：每种超过30000部电影的流派的一部电影。 由于没有在类型上指定顺序，因此该顺序将由UID决定。 计算是通过边的数目来完成的，然后进行其它部分的查询。
```
{
  genre(func: gt(count(~genre), 30000)){
    name@en
    ~genre (first:1) {
      name@en
    }
  }
}
```
结果
```
{
  "data": {
    "genre": [
      {
        "name@en": "Documentary film",
        "~genre": [
          {
            "name@en": "Wild Things"
          }
        ]
      },
      {
        "name@en": "Drama",
        "~genre": [
          {
            "name@en": "José María y María José: Una pareja de hoy"
          }
        ]
      },
      {
        "name@en": "Comedy",
        "~genre": [
          {
            "name@en": "José María y María José: Una pareja de hoy"
          }
        ]
      },
      {
        "name@en": "Short Film",
        "~genre": [
          {
            "name@en": "Side Effects"
          }
        ]
      }
    ]
  }
}
```
查询示例：导演是Steven及其电影的initial_release_date大于电影《少数派报告》的电影。
```
{
  var(func: eq(name@en,"Minority Report")) {
    d as initial_release_date
  }

  me(func: eq(name@en, "Steven Spielberg")) {
    name@en
    director.film @filter(ge(initial_release_date, val(d))) {
      initial_release_date
      name@en
    }
  }
}
```
结果
```
{
  "data": {
    "me": [
      {
        "name@en": "Steven Spielberg",
        "director.film": [
          {
            "initial_release_date": "2012-10-08T00:00:00Z",
            "name@en": "Lincoln"
          },
          {
            "initial_release_date": "2011-12-04T00:00:00Z",
            "name@en": "War Horse"
          }
        ]
      }
    ]
  }
}

## uid
语法示例：

- q(func: uid(<uid>))
- predicate @filter(uid(<uid1>, ..., <uidn>))
- predicate @filter(uid(a)) for variable a
- q(func: uid(a,b)) for variables a and b
在当前过滤的查询中限定出UIDs指定的结点。

对于查询变量a，uid（a）代表存储在a中的UID集。 对于值变量b，uid（b）表示从UID到值映射的UID。 如果有两个或多个变量，uid（a，b，...）表示所有变量的并集。

uid（<uid>）就像一个身份查询函数一样，即使节点没有任何边，也将返回请求的结点的UID。

查询示例：如果已知某节点的UID，则可以直接读取该节点的值。 如知道Priyanka Chopra这部电影的UID
```
{
  films(func: uid(0x2c964)) {
    name@hi
    actor.film {
      performance.film {
        name@hi
      }
    }
  }
}
```
结果
```
{
  "data": {
    "films": [
      {
        "name@hi": "प्रियंका चोपड़ा",
        "actor.film": [
          {
            "performance.film": [
              {
                "name@hi": "यकीन"
              }
            ]
          }
        ]
      }
    ]
  }
}
查询示例：按流派划分的Taraji Henson的电影。
```
{
  var(func: allofterms(name@en, "Taraji Henson")) {
    actor.film {
      F as performance.film {
        G as genre
      }
    }
  }

  Taraji_films_by_genre(func: uid(G)) {
    genre_name : name@en
    films : ~genre @filter(uid(F)) {
      film_name : name@en
    }
  }
}
```
结果
```

  "data": {
    "Taraji_films_by_genre": [
      {
        "genre_name": "War film",
        "films": [
          {
            "film_name": "Talk to Me"
          }
        ]
      },
      {
        "genre_name": "Horror",
        "films": [
          {
            "film_name": "Satan's School for Girls"
          }
        ]
      }
    ]
  }
}
```
查询示例：按流派数量排序Taraji Henson的电影，并按Taraji在每种流派制作了多少部电影的顺序列出了流派。

```
{
  var(func: allofterms(name@en, "Taraji Henson")) {
    actor.film {
      F as performance.film {
        G as count(genre)
        genre {
          C as count(~genre @filter(uid(F)))
        }
      }
    }
  }

  Taraji_films_by_genre_count(func: uid(G), orderdesc: val(G)) {
    film_name : name@en
    genres : genre (orderdesc: val(C)) {
      genre_name : name@en
    }
  }
}
```
结果
```
{
  "data": {
    "Taraji_films_by_genre_count": [
      {
        "film_name": "Date Night",
        "genres": [
          {
            "genre_name": "Comedy"
          },
          {
            "genre_name": "Crime Fiction"
          }
        ]
      }
    ]
  }
}
```
## uid_in
语法示例：

- q(func: ...) @filter(uid_in(predicate, <uid>))
- predicate1 @filter(uid_in(predicate2, <uid>))
架构类型：UID

所需索引：无

uid函数是基于UID过滤当前级别的节点，而uid_in函数允许沿边检查其是否关联到特定的UID。 这通常可以节省额外的查询块，并避免返回边。

uid_in不能在根结点下使用，它接受一个UID常量作为其参数（而不是变量）。

查询示例：Marc Caro和Jean-Pierre Jeunet（UID 0x99706）的合作。 如果Jean-Pierre Jeunet的UID是已知的，则以这种方式进行查询就无需额外的代码将其UID提取到变量中，并且也无需额外的针对〜director.film的遍历和筛选。
```
{
  caro(func: eq(name@en, "Marc Caro")) {
    name@en
    director.film @filter(uid_in(~director.film, 0x99706)) {
      name@en
    }
  }
}
```
结果
```
{
  "data": {
    "caro": [
      {
        "name@en": "Marc Caro"
      }
    ]
  }
}
```
## has
语法示例：has(predicate)

模式类型：全部

确定节点是否具有特定谓词。

查询示例：前五名导演并且他所有录制有发行日期的电影。 导演们执导了至少一部电影-gt（count（director.film），0）的等效语义。
```
{
  me(func: has(director.film), first: 5) {
    name@en
    director.film @filter(has(initial_release_date))  {
      initial_release_date
      name@en
    }
  }
}
```
结果
```
{
  "data": {
    "me": [
      {
        "name@en": "Zehra Yiğit"
      },
      {
        "name@en": "Charlton Heston",
        "director.film": [
          {
            "initial_release_date": "1982-09-23T00:00:00Z",
            "name@en": "Mother Lode"
          },
          {
            "initial_release_date": "1972-03-02T00:00:00Z",
            "name@en": "Antony and Cleopatra"
          },
          {
            "initial_release_date": "1988-12-21T00:00:00Z",
            "name@en": "A Man for All Seasons"
          }
        ]
      },
      {
        "name@en": "Rajeev Sharma",
        "director.film": [
          {
            "initial_release_date": "2012-01-01T00:00:00Z",
            "name@en": "Nabar"
          },
          {
            "initial_release_date": "2014-05-30T00:00:00Z",
            "name@en": "47 to 84"
          }
        ]
      },
      {
        "name@en": "Gabriel Nussbaum",
        "director.film": [
          {
            "initial_release_date": "2011-01-01T00:00:00Z",
            "name@en": "How It Ended"
          },
          {
            "initial_release_date": "2007-01-01T00:00:00Z",
            "name@en": "Wade in the Water, Children"
          }
        ]
      },
      {
        "name@en": "James Wharton O'Keefe",
        "director.film": [
          {
            "initial_release_date": "2013-01-01T00:00:00Z",
            "name@en": "Go Public: A Day in the Life of an American School District"
          }
        ]
      }
    ]
  }
}
```

## Geolocation

`提示 当前我们只支持Point, Polygon与MultiPolygon这几种几何类型的索引。但是，Dgraph可以存储其它类型的地理信息数据`
请注意，对于地理查询，所有带有孔的多边形都将替换为外部环状查询，会忽略孔。 此外，至版本0.7.7，多边形包含检查都是近似的。

## Mutations 突变
要使用geo函数，您需要在谓词上建立索引。
`loc: geo @index(geo) .`
这是添加点的方法。
```
{
  set {
    <_:0xeb1dde9c> <loc> "{'type':'Point','coordinates':[-122.4220186,37.772318]}"^^<geo:geojson> .
    <_:0xeb1dde9c> <name> "Hamon Tower" .
    <_:0xeb1dde9c> <dgraph.type> "Location" .
  }
}
```
下面将多边形与节点关联的方法。 添加MultiPolygon也是类似的。
```
{
  set {
    <_:0xf76c276b> <loc> "{'type':'Polygon','coordinates':[[[-122.409869,37.7785442],[-122.4097444,37.7786443],[-122.4097544,37.7786521],[-122.4096334,37.7787494],[-122.4096233,37.7787416],[-122.4094004,37.7789207],[-122.4095818,37.7790617],[-122.4097883,37.7792189],[-122.4102599,37.7788413],[-122.409869,37.7785442]],[[-122.4097357,37.7787848],[-122.4098499,37.778693],[-122.4099025,37.7787339],[-122.4097882,37.7788257],[-122.4097357,37.7787848]]]}"^^<geo:geojson> .
    <_:0xf76c276b> <name> "Best Western Americana Hotel" .
    <_:0xf76c276b> <dgraph.type> "Location" .
  }
}

```
以上示例摘自我们的SF Tourism数据集。
## Query
### near
语法示例：near(predicate, [long, lat], distance)

模式类型：geo

所需索引：geo

匹配所有由谓词给出的位置在geojson坐标[long，lat]的多少米距内的实体。

查询示例：距旧金山金门公园1000米（1公里）内的旅游地。
```
{
  tourist(func: near(loc, [-122.469829, 37.771935], 1000) ) {
    name
  }
}
```
结果
```
{
  "data": {
    "tourist": [
      {
        "name": "Peace Lantern"
      },
      {
        "name": "De Young Museum"
      },
      {
        "name": "Conservatory of Flowers"
      }
    ]
  }
}
```
### within
语法示例：within(predicate, [[[long1, lat1], ..., [longN, latN]]]

模式类型：geo

所需索引：geo

匹配通过由geojson坐标数组指定的多边形来限定位置内的所有实体。

查询示例：旧金山金门公园指定区域内的旅游目的地。
```
{
  tourist(func: within(loc, [[[-122.47266769409178, 37.769018558337926 ], [ -122.47266769409178, 37.773699921075135 ], [ -122.4651575088501, 37.773699921075135 ], [ -122.4651575088501, 37.769018558337926 ], [ -122.47266769409178, 37.769018558337926]]] )) {
    name
  }
}
```
结果
```
{
  "data": {
    "tourist": [
      {
        "name": "Peace Lantern"
      },
      {
        "name": "De Young Museum"
      },
      {
        "name": "Buddha"
      }
    ]
  }
}
```
contains
语法示例：contains(predicate, [long, lat]) 或 contains(predicate, [[long1, lat1], ..., [longN, latN]])

模式类型：geo

所需索引：geo

匹配所有通过geojson坐标[long，lat]或给定geojson多边形限定的范围内包含的实体。

查询示例：在旧金山动物园的火烈鸟围栏中包含的所有实体点。
```
{
  tourist(func: contains(loc, [ -122.50326097011566, 37.73353615592843 ] )) {
    name
  }
}
```
结果
```
{
  "data": {
    "tourist": [
      {
        "name": "San Francisco Zoo"
      },
      {
        "name": "Flamingo"
      }
    ]
  }
}
```

### intersects
语法示例：intersects(predicate, [[[long1, lat1], ..., [longN, latN]]])
模式类型：geo

所需索引：geo
检索所有与谓词限定的多边形相交的地点
```
{
  tourist(func: intersects(loc, [[[-122.503325343132, 37.73345766902749 ], [ -122.503325343132, 37.733903134117966 ], [ -122.50271648168564, 37.733903134117966 ], [ -122.50271648168564, 37.73345766902749 ], [ -122.503325343132, 37.73345766902749]]] )) {
    name
  }
}`
```
结果
```
{
  "data": {
    "tourist": [
      {
        "name": "San Francisco Zoo"
      },
      {
        "name": "Flamingo"
      }
    ]
  }
}
```
### 连接过滤器
在@filter中，可以将多个过滤函数与布尔连接符一起使用。

AND，OR和NOT
连接词AND，OR和NOT不仅可以加入过滤器，并且可以内置到任意复杂的过滤器中，例如（NOT A OR B）AND（C AND NOT（D OR E））。 
请注意，NOT比AND优先级更高，而AND比OR优先级更高。

查询示例：所有同时包含“印第安纳”和“琼斯”或“侏罗纪”和“公园”的史蒂芬·斯皮尔伯格的电影。
```
{
  me(func: eq(name@en, "Steven Spielberg")) @filter(has(director.film)) {
    name@en
    director.film @filter(allofterms(name@en, "jones indiana") OR allofterms(name@en, "jurassic park"))  {
      uid
      name@en
    }
  }
}
```
结果
```
{
  "data": {
    "me": [
      {
        "name@en": "Steven Spielberg",
        "director.film": [
          {
            "uid": "0x9a41",
            "name@en": "Indiana Jones and the Temple of Doom"
          },
          {
            "uid": "0x1cde3",
            "name@en": "Jurassic Park"
          }
        ]
      }
    ]
  }
}
```
### alias别名
语法示例：

aliasName : predicate
aliasName : predicate { ... }
aliasName : varName as ...
aliasName : count(predicate)
aliasName : max(val(varName))
别名是在结果中提供备用名称。 谓词，变量和集合可以通过在别名之前加上别名来命名。 别名不必与原始谓词名称不同，但是，在一个块内，别名必须与谓词名称和同一块中返回的其他别名不同。 别名可用于在一个块中多次返回同一谓词。

查询示例：具有名称匹配到“Steven”的导演，他的UID，英文名称，每部电影的演员平均人数，电影总数以及每部电影的英语和法语名称。
```
{
  ID as var(func: allofterms(name@en, "Steven")) @filter(has(director.film)) {
    director.film {
      num_actors as count(starring)
    }
    average as avg(val(num_actors))
  }

  films(func: uid(ID)) {
    director_id : uid
    english_name : name@en
    average_actors : val(average)
    num_films : count(director.film)

    films : director.film {
      name : name@en
      english_name : name@en
      french_name : name@fr
    }
  }
}
```
结果
```
{
  "data": {
    "films": [
      {
        "director_id": "0x33a6",
        "english_name": "Steven Wilsey",
        "average_actors": 0,
        "num_films": 1,
        "films": [
          {
            "name": "At Night I Was Beautiful",
            "english_name": "At Night I Was Beautiful"
          }
        ]
      },
      {
        "director_id": "0x3ce3",
        "english_name": "Steven Bratter",
        "average_actors": 10,
        "num_films": 1,
        "films": [
          {
            "name": "First Strike",
            "english_name": "First Strike"
          }
        ]
      },
      {
        "director_id": "0x57c8",
        "english_name": "Steven Saussey",
        "average_actors": 3,
        "num_films": 1,
        "films": [
          {
            "name": "Whisker",
            "english_name": "Whisker"
          }
        ]
      }
    ]
  }
}
```
### 分页
分页只返回一部分结果集，而不是整个结果集。这对于top-k这样的查询以及减小结果集的大小，以便客户端处理是很有用。

分页通常与排序一起使用。
```
注意如果未指定排序顺序，结果是会按uid进行排序的，而uid是随机分配的。因此，默认的排序可能不是您期望的。
```

### 开始First
语法示例：

 - q(func: ..., first: N)
 - predicate (first: N) { ... }
 - predicate @filter(...) (first: N) { ... }
对于正数N，按排序或UID顺序检索返回前N个结果。

对于负数N，按排序或UID顺序检索返回最后N个结果。当前，当前版本还只支持无任何排序的情况下的负数N。要获得带排序结果的负数的效果，请颠倒排序顺序并使用正数N。

查询示例：按史蒂芬·斯皮尔伯格（Steven Spielberg）执导，按UID顺序排序的前两部电影，以及按英文名称按字母顺序排序的该电影的前三类流派。

查询
```
{
  me(func: allofterms(name@en, "Steven Spielberg")) {
    director.film (first: -2) {
      name@en
      initial_release_date
      genre (orderasc: name@en) (first: 3) {
          name@en
      }
    }
  }
}
```
响应
```
{
  "data": {
    "me": [
      {
        "director.film": [
          {
            "name@en": "Firelight",
            "initial_release_date": "1964-03-24T00:00:00Z",
            "genre": [
              {
                "name@en": "Science Fiction"
              },
              {
                "name@en": "Thriller"
              }
            ]
          },
          {
            "name@en": "Amazing Stories: Book One",
            "genre": [
              {
                "name@en": "Comedy"
              },
              {
                "name@en": "Drama"
              }
            ]
          }
        ]
      }
    ]
  }
}
```

查询示例：在所有名叫史蒂文的导演中导演最多的演员的三位名叫史蒂文的导演。

查询
```
{
  ID as var(func: allofterms(name@en, "Steven")) @filter(has(director.film)) {
    director.film {
      stars as count(starring)
    }
    totalActors as sum(val(stars))
  }

  mostStars(func: uid(ID), orderdesc: val(totalActors), first: 3) {
    name@en
    stars : val(totalActors)

    director.film {
      name@en
    }
  }
}
```

响应
```
{
  "data": {
    "mostStars": [
      {
        "name@en": "Steven Spielberg",
        "stars": 1665,
        "director.film": [
          {
            "name@en": "Hook"
          },
          {
            "name@en": "The Color Purple"
          },
          {
            "name@en": "Amazing Stories: Book One"
          }
        ]
      },
      {
        "name@en": "Steven Scarborough",
        "stars": 1169,
        "director.film": [
          {
            "name@en": "Kris Lord vs. Ken Ryker"
          },
          {
            "name@en": "Dr.'s Orders 2: Dilation"
          },
          {
            "name@en": "Screw: Right to the Point"
          }
        ]
      },
      {
        "name@en": "Steven Soderbergh",
        "stars": 1023,
        "director.film": [
          {
            "name@en": "Out of Sight"
          },
          {
            "name@en": "Schizopolis"
          },
          {
            "name@en": "King of the Hill"
          },
          {
            "name@en": "The Great Antonio"
          },
          {
            "name@en": "Wholphin: Issue 2"
          },
          {
            "name@en": "Life Interrupted"
          }
        ]
      }
    ]
  }
}
```
### 偏移offset
语法示例
 - q(func: ..., offset: N)
 - predicate (offset: N) { ... }
 - predicate (first: M, offset: N) { ... }
 - predicate @filter(...) (offset: N) { ... }

使用offset:N时，不返回前N个结果。与first一起综合使用的时候 , first: M, offset: N 结合使用：跳过N个结果并返回M个结果。

查询示例：按英文名称查询徐克的电影，跳过前4部，然后返回以下6部

查询
```
{
  me(func: allofterms(name@en, "Hark Tsui")) {
    name@zh
    name@en
    director.film (orderasc: name@en) (first:6, offset:4)  {
      genre {
        name@en
      }
      name@zh
      name@en
      initial_release_date
    }
  }
}
```
响应
```
{
  "data": {
    "me": [
      {
        "name@zh": "徐克",
        "name@en": "Tsui Hark",
        "director.film": [
          {
            "genre": [
              {
                "name@en": "Science Fiction"
              },
              {
                "name@en": "Superhero movie"
              },
              {
                "name@en": "Action Film"
              }
            ],
            "name@en": "Black Mask 2: City of Masks",
            "initial_release_date": "2002-01-01T00:00:00Z"
          },
          {
            "genre": [
              {
                "name@en": "Adventure game"
              },
              {
                "name@en": "Drama"
              },
              {
                "name@en": "Action Film"
              }
            ],
            "name@zh": "狄仁杰之通天帝国",
            "name@en": "Detective Dee: Mystery of the Phantom Flame",
            "initial_release_date": "2010-09-05T00:00:00Z"
          },
          {
            "genre": [
              {
                "name@en": "Thriller"
              },
              {
                "name@en": "Crime Fiction"
              },
              {
                "name@en": "Action Film"
              }
            ],
            "name@en": "Don't Play with Fire",
            "initial_release_date": "1980-12-04T00:00:00Z"
          },
          {
            "genre": [
              {
                "name@en": "Thriller"
              },
              {
                "name@en": "Buddy film"
              }
            ],
            "name@en": "Double Team",
            "initial_release_date": "1997-04-04T00:00:00Z"
          },
          {
            "genre": [
              {
                "name@en": "Adventure Film"
              },
              {
                "name@en": "Action Film"
              }
            ],
            "name@zh": "龙门飞甲",
            "name@en": "Flying Swords of Dragon Gate",
            "initial_release_date": "2011-12-15T00:00:00Z"
          },
          {
            "genre": [
              {
                "name@en": "World cinema"
              },
              {
                "name@en": "Fantasy Adventure"
              }
            ],
            "name@zh": "青蛇",
            "name@en": "Green Snake",
            "initial_release_date": "1993-01-01T00:00:00Z"
          }
        ]
      },
      {
        "name@zh": "小倩",
        "name@en": "A Chinese Ghost Story: The Tsui Hark Animation"
      },
      {
        "name@en": "Tsui Hark Movie Studio"
      }
    ]
  }
}
```
### After
语法示例：

 - q(func: ..., after: UID)
 - predicate (first: N, after: UID) { ... }
 - predicate @filter(...) (first: N, after: UID) { ... }
跳过某些结果后获得结果的另一种方法是使用默认的UID排序并直接跳过UID指定的节点。例如，第一个查询的形式可能是predicate (after: 0x0, first: N)，也可能predicate(after: <uid of last entity in last result>, first: N)。

查询示例：巴兹·鲁曼（Baz Luhrmann）的前五部电影，按UID顺序排序。

查询
```
{
  me(func: allofterms(name@en, "Baz Luhrmann")) {
    name@en
    director.film (first:5) {
      uid
      name@en
    }
  }
}
```
响应
```
{
  "data": {
    "me": [
      {
        "name@en": "Baz Luhrmann",
        "director.film": [
          {
            "uid": "0x434",
            "name@en": "The Great Gatsby"
          },
          {
            "uid": "0x1e0d",
            "name@en": "Strictly Ballroom"
          },
          {
            "uid": "0x11bdb",
            "name@en": "Moulin Rouge!"
          },
          {
            "uid": "0x27c60",
            "name@en": "Romeo + Juliet"
          },
          {
            "uid": "0x2e760",
            "name@en": "Australia"
          }
        ]
      }
    ]
  }
}
```
第五部电影是澳大利亚电影经典“Strictly Ballroom”。它具有UID 0x99e44。现在可以使用after获得“Strictly Ballroom”之后的结果。
```
{
  me(func: allofterms(name@en, "Baz Luhrmann")) {
    name@en
    director.film (first:5, after: 0x99e44) {
      uid
      name@en
    }
  }
}
```
结果
```
{
  "data": {
    "me": [
      {
        "name@en": "Baz Luhrmann",
        "director.film": [
          {
            "uid": "0xce247",
            "name@en": "Puccini: La Boheme (Sydney Opera)"
          },
          {
            "uid": "0xe40e5",
            "name@en": "No. 5 the Film"
          }
        ]
      }
    ]
  }
}
```

### 计数 Count
语法示例：

 - count(predicate)
 - count(uid)
形式count(predicate)计算从一个节点引出的谓词边数。

形式count(uid)对封闭块中匹配的UID的数量进行计数。

查询示例：每个叫奥兰多的演员表演的电影数量。

查询
```
{
  me(func: allofterms(name@en, "Orlando")) @filter(has(actor.film)) {
    name@en
    count(actor.film)
  }
}
```
响应
```
{
  "data": {
    "me": [
      {
        "name@en": "Orlando Seale",
        "count(actor.film)": 13
      },
      {
        "name@en": "Antonio Orlando",
        "count(actor.film)": 5
      },
      {
        "name@en": "Orlando Viera",
        "count(actor.film)": 1
      },
      {
        "name@en": "Silvio Orlando",
        "count(actor.film)": 32
      },
      {
        "name@en": "Orazio Orlando",
        "count(actor.film)": 9
      },
      {
        "name@en": "Orlando Fagin",
        "count(actor.film)": 1
      }
    ]
  }
}
```
可以在根使用计数，也可以使用别名。

查询示例：导演超过五部电影的导演人数。在查询的根使用时，计数索引是必需的。

查询
```
{
  directors(func: gt(count(director.film), 5)) {
    totalDirectors : count(uid)
  }
}
```
响应
```
{
  "data": {
    "directors": [
      {
        "totalDirectors": 7712
      }
    ]
  }
}
```
可以将计数分配给值变量。

查询示例：按所上映的电影数量排序的李安（Ang Lee）的“饮食男人女人”（Eat Drink Man Woman）中的演员。
查询
```
{
  var(func: allofterms(name@en, "eat drink man woman")) {
    starring {
      actors as performance.actor {
        totalRoles as count(actor.film)
      }
    }
  }

  edmw(func: uid(actors), orderdesc: val(totalRoles)) {
    name@en
    name@zh
    totalRoles : val(totalRoles)
  }
}
```
结果
```
{
  "data": {
    "edmw": [
      {
        "name@en": "Sylvia Chang",
        "name@zh": "张艾嘉",
        "totalRoles": 35
      },
      {
        "name@en": "Chien-lien Wu",
        "name@zh": "吴倩莲",
        "totalRoles": 20
      },
      {
        "name@en": "Yang Kuei-mei",
        "name@zh": "杨贵媚",
        "totalRoles": 14
      },
      {
        "name@en": "Winston Chao",
        "name@zh": "赵文瑄",
        "totalRoles": 11
      },
      {
        "name@en": "Gua Aleh",
        "name@zh": "归亚蕾",
        "totalRoles": 10
      },
      {
        "name@en": "Chung Ting",
        "totalRoles": 1
      },
      {
        "name@en": "Chin-Cheng Lu",
        "totalRoles": 1
      }
    ]
  }
}
```

### 排序 Sorting
语法示例：

 - q(func: ..., orderasc: predicate)
 - q(func: ..., orderdesc: val(varName))
 - predicate (orderdesc: predicate) { ... }
 - predicate @filter(...) (orderasc: N) { ... }
 - q(func: ..., orderasc: predicate1, orderdesc: predicate2)
可排序的类型：int, float, String, dateTime, default （整数，浮点数，字符串，日期时间，默认值）

结果可以由谓词或变量按升序（orderasc）或降序（orderdesc）排序。

为了对可排序索引的谓词进行排序，Dgraph对值和索引进行并行排序，并返回优先计算的结果。

默认情况下，查询最多检索已排序的1000个结果。不过结果可以用First更改。

查询示例：按发布日期排序法国导演让·皮埃尔·朱内（Jean-Pierre Jeunet）的电影。

查询
```
{
  me(func: allofterms(name@en, "Jean-Pierre Jeunet")) {
    name@fr
    director.film(orderasc: initial_release_date) {
      name@fr
      name@en
      initial_release_date
    }
  }
}
```
响应
```
{
  "data": {
    "me": [
      {
        "name@fr": "Jean-Pierre Jeunet",
        "director.film": [
          {
            "name@fr": "L'Évasion",
            "name@en": "L'évasion",
            "initial_release_date": "1978-01-01T00:00:00Z"
          },
          {
            "name@fr": "Micmacs à tire-larigot",
            "name@en": "Micmacs",
            "initial_release_date": "2009-09-15T00:00:00Z"
          },
          {
            "name@fr": "L'Extravagant Voyage du jeune et prodigieux T. S. Spivet",
            "name@en": "The Young and Prodigious Spivet",
            "initial_release_date": "2013-09-28T00:00:00Z"
          }
        ]
      }
    ]
  }
}
``` 
排序可以在根和值变量上操作。

查询示例：按字母顺序排序所有流派，每种流派中最多的五个电影。

查询
```
{
  genres as var(func: has(~genre)) {
    ~genre {
      numGenres as count(genre)
    }
  }

  genres(func: uid(genres), orderasc: name@en) {
    name@en
    ~genre (orderdesc: val(numGenres), first: 5) {
      name@en
      genres : val(numGenres)
    }
  }
}
```
响应
```
{
  "data": {
    "genres": [
      {
        "name@en": "/m/04rlf",
        "~genre": [
          {
            "name@en": "Bookin'",
            "genres": 3
          },
          {
            "name@en": "La brune et moi",
            "genres": 3
          }
        ]
      },
      {
        "name@en": "3D film",
        "~genre": [
          {
            "name@en": "Mr. X",
            "genres": 4
          },
          {
            "name@en": "Sly Cooper",
            "genres": 4
          },
          {
            "name@en": "Aqaye Alef",
            "genres": 1
          }
        ]
      },
      {
        "name@en": "Abstract animation",
        "~genre": [
          {
            "name@en": "The Heart of the World",
            "genres": 7
          },
          {
            "name@en": "Acousticity",
            "genres": 4
          },
          {
            "name@en": "The Garden of Earthly Delights",
            "genres": 4
          }
        ]
      }
    ]
  }
}
```
排序也可以由多个谓词执行，如下所示。如果第一个谓词的值相等，则按第二个谓词对它们进行排序，依此类推。

查询示例：查找所有类型为Person的节点，按其first_name对其排序，在具有相同first_name的那些节点中，按last_name降序对其进行排序。
```
{
  me(func: type("Person"), orderasc: first_name, orderdesc: last_name) {
    first_name
    last_name
  }
}
```
### 多个查询块
在单个查询中，允许多个查询块。结果是所有具有相应块名称的块。

多个查询块是并行执行。

多个块之间无需以任何方式关联。

查询示例：自2008年以来的安吉丽娜·朱莉（Angelina Jolie）的所有电影和流派，以及彼得·杰克逊（Peter Jackson）电影。

查询
```
{
 AngelinaInfo(func:allofterms(name@en, "angelina jolie")) {
  name@en
   actor.film {
    performance.film {
      genre {
        name@en
      }
    }
   }
  }

 DirectorInfo(func: eq(name@en, "Peter Jackson")) {
    name@en
    director.film @filter(ge(initial_release_date, "2008"))  {
        Release_date: initial_release_date
        Name: name@en
    }
  }
}
```
响应
```
{
  "data": {
    "AngelinaInfo": [
      {
        "name@en": "Angelina Jolie",
        "actor.film": [
          {
            "performance.film": [
              {
                "genre": [
                  {
                    "name@en": "Short Film"
                  }
                ]
              }
            ]
          },
          {
            "performance.film": [
              {
                "genre": [
                  {
                    "name@en": "Slice of life"
                  },
                  {
                    "name@en": "Ensemble Film"
                  }
                ]
              }
            ]
          },
          {
            "performance.film": [
              {
                "genre": [
                  {
                    "name@en": "Animation"
                  },
                  {
                    "name@en": "Comedy"
                  }
                ]
              }
            ]
          }
        ]
      }
    ]
  }
}
```
如果查询的答案中有一些重叠，但是结果集仍然会是独立的。

查询示例：参与了电影Mackenzie Crook的，参与了的电影Jack Davenport。因为它们都在加勒比海盗电影中扮演过，所以结果集是重叠的，但是返回的结果是独立的，并且都包含完整的答案集。

查询
```
{
  Mackenzie(func:allofterms(name@en, "Mackenzie Crook")) {
    name@en
    actor.film {
      performance.film {
        uid
        name@en
      }
      performance.character {
        name@en
      }
    }
  }

  Jack(func:allofterms(name@en, "Jack Davenport")) {
    name@en
    actor.film {
      performance.film {
        uid
        name@en
      }
      performance.character {
        name@en
      }
    }
  }
}
```
响应
```
{
  "data": {
    "Mackenzie": [
      {
        "name@en": "Mackenzie Crook",
        "actor.film": [
          {
            "performance.film": [
              {
                "uid": "0x7f33f",
                "name@en": "Sex Lives of the Potato Men"
              }
            ]
          },
          {
            "performance.film": [
              {
                "uid": "0x842bf",
                "name@en": "Solomon Kane"
              }
            ],
            "performance.character": [
              {
                "name@en": "Father Michael"
              }
            ]
          }
        ]
      }
    ]
  }
}
```
### Var块
Var块是以关键字var开头，并且不会在查询结果中返回。

查询示例：按流派排序的安吉丽娜·朱莉的电影。

查询
```
{
  var(func:allofterms(name@en, "angelina jolie")) {
    name@en
    actor.film {
      A AS performance.film {
        B AS genre
      }
    }
  }

  films(func: uid(B), orderasc: name@en) {
    name@en
    ~genre @filter(uid(A)) {
      name@en
    }
  }
}
```
响应
```
{
  "data": {
    "films": [
      {
        "name@en": "Action Film",
        "~genre": [
          {
            "name@en": "Gone in 60 Seconds"
          },
          {
            "name@en": "Kung Fu Panda"
          },
          {
            "name@en": "Lara Croft Tomb Raider: The Cradle of Life"
          }
        ]
      }
    ]
  }
}
```
### 查询变量
语法示例：

 - varName as q(func: ...) { ... }
 - varName as var(func: ...) { ... }
 - varName as predicate { ... }
 - varName as predicate @filter(...) { ... }
类型：uid

可以将在查询中某个位置匹配的节点（UID）存储在变量中，并在其他位置使用。查询变量可以在其他查​​询块或定义块的子节点中使用。

查询变量在定义时不影响查询的语义。查询变量会在匹配的定义块的所有节点求值。

通常，查询块是并行执行的，但是变量会影响某些块的评估顺序。注意，是不允许由变量依赖性引起的循环的。

如果定义了变量，则必须在查询中的其他地方使用它。

通过使用uid(var-name)这种形式，可以提取UID到查询变量。

语法func: uid(A,B)或@filter(uid(A,B))表示变量A和B的UID的并集。

查询示例：Angelia Jolie和Brad Pitt的电影都曾在同一类型的电影中扮演过角色。请注意，B和D匹配所有电影的所有类型，而不是每个电影的类型。

查询
```
{
 var(func:allofterms(name@en, "angelina jolie")) {
   actor.film {
    A AS performance.film {  # All films acted in by Angelina Jolie
     B As genre  # Genres of all the films acted in by Angelina Jolie
    }
   }
  }

 var(func:allofterms(name@en, "brad pitt")) {
   actor.film {
    C AS performance.film {  # All films acted in by Brad Pitt
     D as genre  # Genres of all the films acted in by Brad Pitt
    }
   }
  }

 films(func: uid(D)) @filter(uid(B)) {   # Genres from both Angelina and Brad
  name@en
   ~genre @filter(uid(A, C)) {  # Movies in either A or C.
     name@en
   }
 }
}
```
响应
```
{
  "data": {
    "films": [
      {
        "name@en": "War film",
        "~genre": [
          {
            "name@en": "Two-Fisted Tales"
          },
          {
            "name@en": "Inglourious Basterds"
          },
          {
            "name@en": "Fury"
          }
        ]
      },
      {
        "name@en": "Indie film",
        "~genre": [
          {
            "name@en": "Babel"
          },
          {
            "name@en": "Hunk"
          },
          {
            "name@en": "Happy Together"
          }
        ]
      }
    ]
  }
}
```
### 值变量
语法示例：

 - varName as scalarPredicate
 - varName as count(predicate)
 - varName as avg(...)
 - varName as math(...)
类型：int，float，String，dateTime，default，geo，bool

值变量是用来存储标量值的。值变量是从封闭块的UID到相应值的映射。

因此，只有在与相同UID匹配的上下文中使用来自值的变量的值才有意义-如果在与不同UID匹配的块中使用，则值变量是未定义的。

定义值变量但在查询中的其他地方不使用它是错误的。

通过使用val(var-name)来提取提取值或使用uid(var-name)提取UID。

Facet 构面值可以存储在值变量中(就是键值对)。

查询示例：80年代经典电影《公主新娘》的演员扮演的电影的角色数量。查询变量pbActors与影片中所有演员的UID匹配。因此，值变量roles是从角色UID到角色数量的映射。可以在totalRoles查询块中使用值变量roles，因为该查询块还与pbActors 的UID匹配，因此可以使用actor到角色数的映射。

查询
```
{
  var(func:allofterms(name@en, "The Princess Bride")) {
    starring {
      pbActors as performance.actor {
        roles as count(actor.film)
      }
    }
  }
  totalRoles(func: uid(pbActors), orderasc: val(roles)) {
    name@en
    numRoles : val(roles)
  }
}
```
响应
```
{
  "data": {
    "totalRoles": [
      {
        "name@en": "Mark Knopfler",
        "numRoles": 1
      },
      {
        "name@en": "Annie Dyson",
        "numRoles": 2
      },
      {
        "name@en": "André the Giant",
        "numRoles": 7
      },
      {
        "name@en": "Malcolm Storry",
        "numRoles": 11
      },
      {
        "name@en": "Margery Mason",
        "numRoles": 12
      }
    ]
  }
}
```
可以使用值变量代替UID变量, 因为能通过从映射中提取到UID列表。

查询示例：与上一示例相同的查询，但是使用值变量roles来匹配totalRoles查询块中的UID。

查询
```
{
  var(func:allofterms(name@en, "The Princess Bride")) {
    starring {
      performance.actor {
        roles as count(actor.film)
      }
    }
  }
  totalRoles(func: uid(roles), orderasc: val(roles)) {
    name@en
    numRoles : val(roles)
  }
}
```
响应
```
{
  "data": {
    "totalRoles": [
      {
        "name@en": "Mark Knopfler",
        "numRoles": 1
      },
      {
        "name@en": "Annie Dyson",
        "numRoles": 2
      },
      {
        "name@en": "André the Giant",
        "numRoles": 7
      },
      {
        "name@en": "Malcolm Storry",
        "numRoles": 11
      },
      {
        "name@en": "Margery Mason",
        "numRoles": 12
      }
    ]
  }
}
```
### 变量的传播
像查询变量一样，值变量可用于其他查询块和嵌套在定义块中的块中。当在定义变量的块内嵌套的块中使用时，该值将作为沿使用点的所有路径的父节点的变量的和来计算。这称为变量传播。

例如：
```
{
  q(func: uid(0x01)) {
    myscore as math(1)          # A
    friends {                   # B
      friends {                 # C
        ...myscore...
      }
    }
  }
}
```
 
在A行，将值变量myscore定义为UID 0x01到值1的映射节点。在B处，每个朋友的值仍为1：每个朋友只有一条路径。传播变量myscore，以便每个朋友的朋友将收到其父代值的总和：如果一个朋友的朋友只能从一个朋友访问，则该值仍为1，如果他们可以从两个朋友访问，则值为2，依此类推。也就是说，标记为C的块中每个朋友的myscore的值将是通往他们的路径的数量。

传播变量接收节点收到的值是其所有父节点的值的和。

例如，这种传播在以下方面很有用：对用户之间的总和进行归一化，找到节点之间的路径数并通过图累积和。

查询示例：对于每部《哈利波特》电影，演员沃里克·戴维斯扮演的角色数量。

查询
```
{
    num_roles(func: eq(name@en, "Warwick Davis")) @cascade @normalize {

    paths as math(1)  # records number of paths to each character

    actor : name@en

    actor.film {
      performance.film @filter(allofterms(name@en, "Harry Potter")) {
        film_name : name@en
        characters : math(paths)  # how many paths (i.e. characters) reach this film
      }
    }
  }
}
```
响应
```
{
  "data": {
    "num_roles": [
      {
        "actor": "Warwick Davis",
        "characters": 1,
        "film_name": "Harry Potter and the Prisoner of Azkaban"
      },
      {
        "actor": "Warwick Davis",
        "characters": 1,
        "film_name": "Harry Potter and the Goblet of Fire"
      },
      {
        "actor": "Warwick Davis",
        "characters": 2,
        "film_name": "Harry Potter and the Deathly Hallows – Part 2"
      },
      {
        "actor": "Warwick Davis",
        "characters": 1,
        "film_name": "Harry Potter and the Deathly Hallows - Part I"
      }
    ]
  }
}
```
查询示例：曾在彼得·杰克逊（Peter Jackson）电影中出演的每个演员，以及他们曾出演过的彼得·杰克逊（Peter Jackson）电影中的时长。

查询
```
{
    movie_fraction(func:eq(name@en, "Peter Jackson")) @normalize {

    paths as math(1)
    total_films : num_films as count(director.film)
    director : name@en

    director.film {
      starring {
        performance.actor {
          fraction : math(paths / (num_films/paths))
          actor : name@en
        }
      }
    }
  }
}
```
响应
```
{
  "data": {
    "movie_fraction": [
      {
        "actor": "Pete O'Herne",
        "director": "Peter Jackson",
        "fraction": 0,
        "total_films": 19
      },
      {
        "actor": "Peter Jackson",
        "director": "Peter Jackson",
        "fraction": 0,
        "total_films": 19
      },
      {
        "actor": "Peter Jackson",
        "director": "Peter Jackson",
        "fraction": 0,
        "total_films": 19
      }
    ]
  }
}
```

### 聚合Aggregation
语法范例：AG(val(varName))

对于AG替换为

 - min：在值变量varName中选择最小值
 - max：选择最大值
 - sum：将值变量varName中的所有值相加
 - avg：计算varName中值的平均值

模式类型：

|聚合|架构类型|
|--|--|
|min / max|int, float, string, dateTime, default|
|sum / avg |int, float|

汇总只能应用于值变量。不需要索引（值已经找到并将其存储在值变量映射中了）。

聚合应该应用于包含变量定义的查询块。与全局的查询变量和值变量相反，聚合是在本地计算的。
例如：

```
A as predicateA {
  ...
  B as predicateB {
    x as ...some value...
  }
  min(val(x))
}
```

此处，A和B是与这些块匹配的所有UID的列表。值变量x是从B中的UID到值的映射。但是，针对A中的每个UID计算聚合min（val（x））。即，其语义为：对于A中的每个UID，取与A的谓词边B相对应的x并计算这些值的汇总最小。

可以将聚合本身分配给值变量，从而将UID映射到聚合映射。

### min
根用法 
查询示例：获取哈利波特电影的最小初始发行日期。

将发布日期分配给变量，然后将其汇总后在一个空块中获取。

查询
```
{
  var(func: allofterms(name@en, "Harry Potter")) {
    d as initial_release_date
  }
  me() {
    min(val(d))
  }
}
```
响应
```
{
  "data": {
    "me": [
      {
        "min(val(d))": "2001-11-04T00:00:00Z"
      }
    ]
  }
}
```
#### 其他级别的用法

查询示例：所有名字是史蒂文（Steven）的导演的首部电影，按升序排序的发行日期。

查询
```
{
  stevens as var(func: allofterms(name@en, "steven")) {
    director.film {
      ird as initial_release_date
      # ird is a value variable mapping a film UID to its release date
    }
    minIRD as min(val(ird))
    # minIRD is a value variable mapping a director UID to their first release date
  }

  byIRD(func: uid(stevens), orderasc: val(minIRD)) {
    name@en
    firstRelease: val(minIRD)
  }
}
```
响应
```
{
  "data": {
    "byIRD": [
      {
        "name@en": "Steven McMillan",
        "firstRelease": "0214-02-28T00:00:00Z"
      },
      {
        "name@en": "J. Steven Edwards",
        "firstRelease": "1929-01-01T00:00:00Z"
      },
      {
        "name@en": "Steven Spielberg",
        "firstRelease": "1964-03-24T00:00:00Z"
      },
      {
        "name@en": "Steven Jacobson",
        "firstRelease": "1966-01-01T00:00:00Z"
      },
      {
        "name@en": "Steven Arnold",
        "firstRelease": "1968-01-01T00:00:00Z"
      }
    ]
  }
}
```
### Max最高
根级使用
查询示例：获取哈利波特电影的最大初始发行日期。

将发布日期分配给变量，然后将其汇总并在一个空块中获取。

查询
```
{
  var(func: allofterms(name@en, "Harry Potter")) {
    d as initial_release_date
  }
  me() {
    max(val(d))
  }
}
```
响应
```
{
  "data": {
    "me": [
      {
        "max(val(d))": "2011-07-27T00:00:00Z"
      }
    ]
  }
}
```
其他级别的用法
查询示例：昆汀·塔伦蒂诺（Quentin Tarantino）的电影以及最新电影的发行日期。

查询
```
{
  director(func: allofterms(name@en, "Quentin Tarantino")) {
    director.film {
      name@en
      x as initial_release_date
    }
    max(val(x))
  }
}
```
响应
```
{
  "data": {
    "director": [
      {
        "director.film": [
          {
            "name@en": "Kill Bill Volume 1",
            "initial_release_date": "2003-10-10T00:00:00Z"
          },
          {
            "name@en": "Django Unchained",
            "initial_release_date": "2012-12-25T00:00:00Z"
          },
          {
            "name@en": "Sin City",
            "initial_release_date": "2005-03-28T00:00:00Z"
          }
        ]
      }
    ]
  }
}
```
### Sum与Avg 求和和平均
根级使用
查询示例：获取名字中有史蒂文与汤姆的人执导的电影总数的总和与平均值。

查询
```
{
  var(func: anyofterms(name@en, "Steven Tom")) {
    a as count(director.film)
  }

  me() {
    avg(val(a))
    sum(val(a))
  }
}
```
响应
```
{
  "data": {
    "me": [
      {
        "avg(val(a))": 0.224982
      },
      {
        "sum(val(a))": 1558
      }
    ]
  }
}
```
其他级别的用法
查询示例：史蒂芬·斯皮尔伯格（Steven Spielberg）的电影，以及每部电影的流派数以及每部电影的流派总数和平均流派数。

查询
```
{
  director(func: eq(name@en, "Steven Spielberg")) {
    name@en
    director.film {
      name@en
      numGenres : g as count(genre)
    }
    totalGenres : sum(val(g))
    genresPerMovie : avg(val(g))
  }
}
```
响应
```
{
  "data": {
    "director": [
      {
        "name@en": "Steven Spielberg",
        "director.film": [
          {
            "name@en": "Hook",
            "numGenres": 8
          },
          {
            "name@en": "The Color Purple",
            "numGenres": 1
          },
          {
            "name@en": "Schindler's List",
            "numGenres": 5
          },
          {
            "name@en": "Amistad",
            "numGenres": 3
          },
          {
            "name@en": "Indiana Jones and the Temple of Doom",
            "numGenres": 4
          },
          {
            "name@en": "Duel",
            "numGenres": 4
          },
          {
            "name@en": "Lincoln",
            "numGenres": 5
          },
          {
            "name@en": "Jurassic Park",
            "numGenres": 3
          },
          {
            "name@en": "Indiana Jones and the Raiders of the Lost Ark",
            "numGenres": 2
          },
          {
            "name@en": "War Horse",
            "numGenres": 2
          },
          {
            "name@en": "Close Encounters of the Third Kind",
            "numGenres": 3
          },
          {
            "name@en": "War of the Worlds",
            "numGenres": 9
          },
          {
            "name@en": "1941",
            "numGenres": 5
          },
          {
            "name@en": "Minority Report",
            "numGenres": 12
          },
          {
            "name@en": "Twilight Zone: The Movie",
            "numGenres": 3
          },
          {
            "name@en": "Saving Private Ryan",
            "numGenres": 3
          },
          {
            "name@en": "Jaws",
            "numGenres": 8
          },
          {
            "name@en": "Empire of the Sun",
            "numGenres": 7
          },
          {
            "name@en": "Indiana Jones and the Kingdom of the Crystal Skull",
            "numGenres": 7
          },
          {
            "name@en": "The Lost World: Jurassic Park",
            "numGenres": 5
          },
          {
            "name@en": "The Adventures of Tintin: The Secret of the Unicorn",
            "numGenres": 6
          },
          {
            "name@en": "Indiana Jones and the Last Crusade",
            "numGenres": 4
          },
          {
            "name@en": "Catch Me If You Can",
            "numGenres": 4
          },
          {
            "name@en": "The Terminal",
            "numGenres": 5
          },
          {
            "name@en": "Munich",
            "numGenres": 5
          },
          {
            "name@en": "Slipstream",
            "numGenres": 1
          },
          {
            "name@en": "E.T. the Extra-Terrestrial",
            "numGenres": 5
          },
          {
            "name@en": "Bridge of Spies",
            "numGenres": 1
          },
          {
            "name@en": "A.I. Artificial Intelligence",
            "numGenres": 4
          },
          {
            "name@en": "The Sugarland Express",
            "numGenres": 3
          },
          {
            "name@en": "The BFG",
            "numGenres": 0
          },
          {
            "name@en": "Amblin",
            "numGenres": 1
          },
          {
            "name@en": "Something Evil",
            "numGenres": 3
          },
          {
            "name@en": "Robopocalypse",
            "numGenres": 3
          },
          {
            "name@en": "Always",
            "numGenres": 4
          },
          {
            "name@en": "Savage",
            "numGenres": 2
          },
          {
            "name@en": "The Attack of the Mummies",
            "numGenres": 0
          },
          {
            "name@en": "Firelight",
            "numGenres": 2
          },
          {
            "name@en": "Amazing Stories: Book One",
            "numGenres": 2
          }
        ],
        "totalGenres": 154,
        "genresPerMovie": 3.948718
      },
      {
        "name@en": "Steven Spielberg"
      },
      {
        "name@en": "Steven Spielberg"
      },
      {
        "name@en": "Steven Spielberg"
      }
    ]
  }
}
```
### 聚合的聚合
可以将聚合分配给值变量，因此可以依次进行这些变量的聚合计算。

查询示例：对于彼得·杰克逊（Peter Jackson）电影中的每个演员，请查在每一部电影中扮演的角色数量。将这些内容相加即可得出电影中所有演员曾经扮演的角色总数。然后总结一下，找出彼得·杰克逊电影中曾出现过的演员所扮演的角色总数。请注意，这演示了如何聚合聚合；不过，这种情况下的答案并不十分准确，因为在多部彼得·杰克逊（Peter Jackson）电影中出演的演员都被计算了一次以上。

查询
```
{
  PJ as var(func:allofterms(name@en, "Peter Jackson")) {
    director.film {
      starring {  # starring an actor
        performance.actor {
          movies as count(actor.film)
          # number of roles for this actor
        }
        perf_total as sum(val(movies))
      }
      movie_total as sum(val(perf_total))
      # total roles for all actors in this movie
    }
    gt as sum(val(movie_total))
  }

  PJmovies(func: uid(PJ)) {
    name@en
    director.film (orderdesc: val(movie_total), first: 5) {
      name@en
      totalRoles : val(movie_total)
    }
    grandTotal : val(gt)
  }
}
```
响应
```
{
  "data": {
    "PJmovies": [
      {
        "name@en": "Peter Jackson",
        "director.film": [
          {
            "name@en": "The Lord of the Rings: The Two Towers",
            "totalRoles": 1565
          },
          {
            "name@en": "The Lord of the Rings: The Return of the King",
            "totalRoles": 1518
          },
          {
            "name@en": "The Lord of the Rings: The Fellowship of the Ring",
            "totalRoles": 1335
          },
          {
            "name@en": "The Hobbit: An Unexpected Journey",
            "totalRoles": 1274
          },
          {
            "name@en": "The Hobbit: The Desolation of Smaug",
            "totalRoles": 1261
          }
        ],
        "grandTotal": 10592
      },
      {
        "name@en": "Peter Jackson"
      },
      {
        "name@en": "Peter Jackson"
      },
      {
        "name@en": "Peter Jackson"
      },
      {
        "name@en": "Sam Peter Jackson"
      }
    ]
  }
}
```
数值变量数学
值变量可以使用数学函数进行组合。例如，这可用于关联得分，然后将其用于排序或执行其他操作，例如可能用于构建新闻提要，简单的推荐系统等。

数学语句必须包含在math( <exp> )这一表达式中，并且必须存储到值变量中。

支持的运算符如下：

|运算符|接受的类型|功能|
|--|--|--|
|+ - * / %|int，float|执行相应的运算|
|min max|除了geo，bool（二进制函数）以外的所有类型| 在这两种类型中选择min / max值|
|< > <= >= == !=|除geo，bool以外的所有类型|根据值返回true或false|
|floor ceil ln exp sqrt|int, float (unary function)|执行相应的计算|
|since|dateTime|返回从指定时间开始的浮动类型秒数|
|pow(a, b)|int, float|返回a的b次方|
|logbase(a,b)|int，float|返回以b为底的a的对数|
|cond(a, b, c)|第一个操作数必须是布尔值|如果a为true，则选择b否则为c|

查询示例：史蒂芬·斯皮尔伯格的每部电影评分，既计算包括演员人数，类型和国家/地区的总和。按降序排列列出前五部此类电影。

查询
```
{
	var(func:allofterms(name@en, "steven spielberg")) {
		films as director.film {
			p as count(starring)
			q as count(genre)
			r as count(country)
			score as math(p + q + r)
		}
	}

	TopMovies(func: uid(films), orderdesc: val(score), first: 5){
		name@en
		val(score)
	}
}
```
响应
```
{
  "data": {
    "TopMovies": [
      {
        "name@en": "Lincoln",
        "val(score)": 179
      },
      {
        "name@en": "Minority Report",
        "val(score)": 156
      },
      {
        "name@en": "Schindler's List",
        "val(score)": 145
      },
      {
        "name@en": "The Terminal",
        "val(score)": 118
      },
      {
        "name@en": "Saving Private Ryan",
        "val(score)": 99
      }
    ]
  }
}
```
值变量及其聚合结果可在过滤器中使用。

查询示例：计算每部史蒂文·斯皮尔伯格电影的得分，条件为发行日期为条件，以对超过10年的电影进行惩罚性过滤，并根据结果得分进行过滤。

查询
```
{
  var(func:allofterms(name@en, "steven spielberg")) {
    films as director.film {
      p as count(starring)
      q as count(genre)
      date as initial_release_date
      years as math(since(date)/(365*24*60*60))
      score as math(cond(years > 10, 0, ln(p)+q-ln(years)))
    }
  }

  TopMovies(func: uid(films), orderdesc: val(score)) @filter(gt(val(score), 2)){
    name@en
    val(score)
    val(date)
  }
}
```
响应
```
{
  "data": {
    "TopMovies": [
      {
        "name@en": "Lincoln",
        "val(score)": 8.089933,
        "val(date)": "2012-10-08T00:00:00Z"
      },
      {
        "name@en": "The Adventures of Tintin: The Secret of the Unicorn",
        "val(score)": 6.529441,
        "val(date)": "2011-10-23T00:00:00Z"
      },
      {
        "name@en": "War Horse",
        "val(score)": 4.024157,
        "val(date)": "2011-12-04T00:00:00Z"
      }
    ]
  }
}
```
通过数学运算计算出的值将存储到值变量中，因此可以对它们进行汇总。

查询示例：计算史蒂文·斯皮尔伯格（Steven Spielberg）的每部电影的得分，然后合计得分。
查询
```
{
	steven as var(func:eq(name@en, "Steven Spielberg")) @filter(has(director.film)) {
		director.film {
			p as count(starring)
			q as count(genre)
			r as count(country)
			score as math(p + q + r)
		}
		directorScore as sum(val(score))
	}

	score(func: uid(steven)){
		name@en
		val(directorScore)
	}
}
```
响应
```
{
  "data": {
    "score": [
      {
        "name@en": "Steven Spielberg",
        "val(directorScore)": 1865
      }
    ]
  }
}
```
