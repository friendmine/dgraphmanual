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

```  
