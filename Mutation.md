### 三元组
使用set关键字可以完成添加三元组的突变。

```
{
  set {
    # triples in here
  }
}
```

在Mutation突变操作中使用的输入语言为W3C标准[RDF N-Quad格式](https://www.w3.org/TR/n-quads/)的三元组。

每个三元组具有以下形式
```
<subject> <predicate> <object> .
```
意思就是由subject的图节点通过有向边对应的谓词链接到object结点。每个三元组以一个句号结尾[.], 三元组的主题subject始终是图中的节点，而对象object可以是节点或值（文字）。

例如，
```
<0x01> <name> "Alice" .
<0x01> <dgraph.type> "Person" .
```

表示ID为0x01的图节点的名称是字符串值“Alice”。
```
<0x01> <friend> <0x02> .
```

表示ID为0x01的图节点有个好友边链接到节点0x02。

Dgraph为突变操作中的每个空白节点创建唯一的64位标识符-节点的UID。突变可以包括空白节点作为对象或主题的标识符，或者用来自先前突变操作的已知UID。

### 空白节点和UID
突变操作中的空白节点，写为_：identifier，标识突变操作中的节点。 Dgraph会创建一个每个空白节点的UID，并返回创建的UID作为突变操作的结果。 例如，突变：
```
{
 set {
    _:class <student> _:x .
    _:class <student> _:y .
    _:class <name> "awesome class" .
    _:class <dgraph.type> "Class" .
    _:x <name> "Alice" .
    _:x <dgraph.type> "Person" .
    _:x <dgraph.type> "Student" .
    _:x <planet> "Mars" .
    _:x <friend> _:y .
    _:y <name> "Bob" .
    _:y <dgraph.type> "Person" .
    _:y <dgraph.type> "Student" .
 }
}
```
结果输出（此突变操作的运行结果，实际的UID都会有所不同）
```
{
  "data": {
    "code": "Success",
    "message": "Done",
    "uids": {
      "class": "0x2712",
      "x": "0x2713",
      "y": "0x2714"
    }
  }
}
```
这个图数据也就被更新了，跟他们用三元组存储一样。

```
<0x6bc818dc89e78754> <student> <0xc3bcc578868b719d> .
<0x6bc818dc89e78754> <student> <0xb294fb8464357b0a> .
<0x6bc818dc89e78754> <name> "awesome class" .
<0x6bc818dc89e78754> <dgraph.type> "Class" .
<0xc3bcc578868b719d> <name> "Alice" .
<0xc3bcc578868b719d> <dgraph.type> "Person" .
<0xc3bcc578868b719d> <dgraph.type> "Student" .
<0xc3bcc578868b719d> <planet> "Mars" .
<0xc3bcc578868b719d> <friend> <0xb294fb8464357b0a> .
<0xb294fb8464357b0a> <name> "Bob" .
<0xb294fb8464357b0a> <dgraph.type> "Person" .
<0xb294fb8464357b0a> <dgraph.type> "Student" .
```

空白节点的标签_：class，_：x和_：y不能标识突变操作后的节点，所以可以安全地重用以标识以后的突变操作中的新节点。

现有有了新的UID，所以可以用这些UID来做以后的突变操作的基础。例如：添加新的学生到班级中.
```
{
 set {
    <0x6bc818dc89e78754> <student> _:x .
    _:x <name> "Chris" .
    _:x <dgraph.type> "Person" .
    _:x <dgraph.type> "Student" .
 }
}
```
可以直接用UID来查询
```
{
 class(func: uid(0x6bc818dc89e78754)) {
  name
  student {
   name
   planet
   friend {
    name
   }
  }
 }
}
```

### 外部ID
Dgraph的输入语言RDF也支持<a_fixed_identifier> <predicate> literal/node 的三元组以及其上的变体，其中标签a_fixed_identifier旨在作为节点的唯一标识符。 例如，混合使用schema.org标识符，电影数据库标识符和空白节点：

```
_:userA <http://schema.org/type> <http://schema.org/Person> .
_:userA <dgraph.type> "Person" .
_:userA <http://schema.org/name> "FirstName LastName" .
<https://www.themoviedb.org/person/32-robin-wright> <http://schema.org/type> <http://schema.org/Person> .
<https://www.themoviedb.org/person/32-robin-wright> <http://schema.org/name> "Robin Wright" .
```

由于Dgraph本身不支持诸如节点标识符之类的外部ID。 相反，可以将外部ID存储为xid边的节点的属性。 例如，从上面看，谓词名称在Dgraph中是有效的，但是用<http://schema.org/Person>标识的节点可以在Dgraph中识别为UID 0x123，且有个边
```
<0x123> <xid> "http://schema.org/Person" .
<0x123> <dgraph.type> "ExternalType" .
```
而Robin Wright可能是 UID 0x321 且三元组如下：
```
<0x321> <xid> "https://www.themoviedb.org/person/32-robin-wright" .
<0x321> <http://schema.org/type> <0x123> .
<0x321> <http://schema.org/name> "Robin Wright" .
<0x321> <dgraph.type> "Person" .
```
一个相应原Schema架构可能是这样的
```
xid: string @index(exact) .
<http://schema.org/type>: [uid] @reverse .
```

查询示例：所有人
```
{
  var(func: eq(xid, "http://schema.org/Person")) {
    allPeople as <~http://schema.org/type>
  }

  q(func: uid(allPeople)) {
    <http://schema.org/name>
  }
}

```
查询示例：外部Id来查询Robin Wright
```
{
  robin(func: eq(xid, "https://www.themoviedb.org/person/32-robin-wright")) {
    expand(_all_) { expand(_all_) }
  }
}
```
```
注意 xid边不会自动添加到突变操作中。 通常，用户有责任检查现有的xid，并在必要时添加节点和xid边。 Dgraph将所有此类唯一性检查都留给了外部流程。
```

### 外部ID和Upsert块

upsert块会使管理外部ID变得容易。

比如设置Schema架构。
```
xid: string @index(exact) .
<http://schema.org/name>: string @index(exact) .
<http://schema.org/type>: [uid] @reverse .
```
然后再设置个类型
```
{
  set {
    _:blank <xid> "http://schema.org/Person" .
    _:blank <dgraph.type> "ExternalType" .
  }
}
```
现在我们就能创建个新的 Person类型，然后把它的类型跟这个upsert块关联上了。
```
   upsert {
      query {
        var(func: eq(xid, "http://schema.org/Person")) {
          Type as uid
        }
        var(func: eq(<http://schema.org/name>, "Robin Wright")) {
          Person as uid
        }
      }
      mutation {
          set {
           uid(Person) <xid> "https://www.themoviedb.org/person/32-robin-wright" .
           uid(Person) <http://schema.org/type> uid(Type) .
           uid(Person) <http://schema.org/name> "Robin Wright" .
           uid(Person) <dgraph.type> "Person" .
          }
      }
    }
```

你也可以同样删除一个Person， 及其与Type还有Person结点的关系。跟上面一样，不过是换成关键字"set"到关键字"delete"

```
   upsert {
      query {
        var(func: eq(xid, "http://schema.org/Person")) {
          Type as uid
        }
        var(func: eq(<http://schema.org/name>, "Robin Wright")) {
          Person as uid
        }
      }
      mutation {
          delete {
           uid(Person) <xid> "https://www.themoviedb.org/person/32-robin-wright" .
           uid(Person) <http://schema.org/type> uid(Type) .
           uid(Person) <http://schema.org/name> "Robin Wright" .
           uid(Person) <dgraph.type> "Person" .
          }
      }
    }
```

查询一下
```
{
  q(func: eq(<http://schema.org/name>, "Robin Wright")) {
    uid
    xid
    <http://schema.org/name>
    <http://schema.org/type> {
      uid
      xid
    }
  }
}
```

### 语言及RDF类型

RDF N-Quad允许为字符串值和RDF类型指定一种语言。 语言是使用@lang指定的。 例如
```
<0x01> <name> "Adelaide"@en .
<0x01> <name> "Аделаида"@ru .
<0x01> <name> "Adélaïde"@fr .
<0x01> <dgraph.type> "Person" .
```
另外可以看看 [查询中如何处理语言字符串](https://dgraph.io/docs/query-language/graphql-fundamentals/#language-support)。

RDF类型可以使用标准的^^分隔符将文字附加在其上。 例如
```
<0x01> <age> "32"^^<xs:int> .
<0x01> <birthdate> "1985-06-08"^^<xs:dateTime> .
```
支持的RDF数据类型和存储数据的相应内部类型如下。

|存储类型|	Dgraph 类型|
|--|--|
|<xs:string>|	string|
|<xs:dateTime>|	dateTime|
|<xs:date>|	datetime|
|<xs:int>|	int|
|<xs:integer>|	int|
|<xs:boolean>|	bool|
|<xs:double>|	float|
|<xs:float>	|float|
|<geo:geojson>|	geo|
|<xs:password>|	password|
|<http://www.w3.org/2001/XMLSchema#string>|	string|
|<http://www.w3.org/2001/XMLSchema#dateTime>	|dateTime|
|<http://www.w3.org/2001/XMLSchema#date>|	dateTime|
|<http://www.w3.org/2001/XMLSchema#int>	|int|
|<http://www.w3.org/2001/XMLSchema#positiveInteger>	|int|
|<http://www.w3.org/2001/XMLSchema#integer>	|int|
|<http://www.w3.org/2001/XMLSchema#boolean>	|bool|
|<http://www.w3.org/2001/XMLSchema#double>	|float|
|<http://www.w3.org/2001/XMLSchema#float>	|float|

请参阅有关RDF架构类型的部分，以了解RDF类型如何影响突变操作和存储。

### 批量突变操作
每个突变可能包含多个RDF三元组。 对于大数据上传，可以并行处理许多这样的突变操作。 dgraph live命令就是做这个的。 默认情况下，将1000个RDF行批处理到一个查询中，会同时并行运行100个这样的查询。

dgraph live将压缩的N-Quad文件（即没有{set {的三元组列表）作为输入，并对输入中所有三元组进行批处理突变操作。 

该工具的文档。
```
dgraph live --help
``` 
另请参阅[快速数据加载](https://dgraph.io/docs/deploy#fast-data-loading)。

### 删除
用delete关键字表示的delete突变操作，将从数据库中删除三元组。

例如，如果数据库中包含
```
<0xf11168064b01135b> <name> "Lewis Carrol"
<0xf11168064b01135b> <died> "1998"
<0xf11168064b01135b> <dgraph.type> "Person" .
```
然后执行删除突变突变
```
{
  delete {
     <0xf11168064b01135b> <died> "1998" .
  }
}
```
如果存在索引，删除错误的数据，会将其从索引中删除。

对于特定的节点N，谓词P（和相应的索引）的所有数据都可以以S P * 这种模式删除。
```
{
  delete {
     <0xf11168064b01135b> <author.of> * .
  }
}
```
模式S * *从节点中删除所有已知边（节点本身可以​​保留为边的目标），及与已删除边相对应的任何反向边以及已删除数据的任何索引。要删除的谓词是从该节点的类型信息（该节点上dgraph.type边的值及其在模式中的相应定义）派生的。如果缺少该信息，则此操作为空操作。
```
{
  delete {
     <0xf11168064b01135b> * * .
  }
}
```
```
注意不支持模式* P O和* * O，因为它存储/查找所有传入边的成本很高。
```

### 删除非列表谓词
删除非列表谓词（即一对一关系）的值可以通过两种方式完成。

 - 使用上一节中提到的星号。
 - 将对象设置为特定值。如果传递的值不是当前值，则该突变操作虽然会成功但不会生效。如果传递的值是当前值，则该突变将成功并删除三元组。
对于带有语言标签的值，支持以下特殊语法：
```
{
  delete {
    <0x12345> <name@es> * .
  }
}
```

在此示例中，使用语言标记es标记的名称的值将被删除。其他标记的值保持不变。

### RDF列表类型中的构面
构架Schema
```
<name>: string @index(exact).
<nickname>: [string] .
```
在RDF中通过列表创建构面是很方便的
```
{
  set {
    _:Julian <name> "Julian" .
    _:Julian <nickname> "Jay-Jay" (kind="first") .
    _:Julian <nickname> "Jules" (kind="official") .
    _:Julian <nickname> "JB" (kind="CS-GO") .
  }
}
```
查询一下
```
{
  q(func: eq(name,"Julian")){
    name
    nickname @facets
  }
}
```
结果
```
{
  "data": {
    "q": [
      {
        "name": "Julian",
        "nickname|kind": {
          "0": "first",
          "1": "official",
          "2": "CS-GO"
        },
        "nickname": [
          "Jay-Jay",
          "Jules",
          "JB"
        ]
      }
    ]
  }
}
```

### 使用cURL进行突变操作

可以通过向Alpha的/mutate端点发出HTTP的POST请求进行更改。 在命令行上，可以使用curl来完成。 要提交突变操作，请在URL中传递参数commitNow = true。

要运行设定的突变操作：
```
curl -H "Content-Type: application/rdf" -X POST localhost:8080/mutate?commitNow=true -d $'
{
  set {
    _:alice <name> "Alice" .
    _:alice <dgraph.type> "Person" .
  }
}'
```
要运行删除突变操作：
```
curl -H "Content-Type: application/rdf" -X POST localhost:8080/mutate?commitNow=true -d $'
{
  delete {
    # Example: The UID of Alice is 0x56f33
    <0x56f33> <name> * .
  }
}'
```
要运行存储在文件上的RDF，需要使用curl的 --data-binary选项，不同于-d 选项， 数据不是URL编码的
```
curl -H "Content-Type: application/rdf" -X POST localhost:8080/mutate?commitNow=true --data-binary @mutation.txt
```

### JSON的突变操作格式
也可以使用JSON对象指定突变操作。 这可以使突变以更自然的方式表达。 由于大多数语言已经具有JSON操作库，因此它也去除了应用程序自定义序列化代码的需要。

当Dgraph接收到作为JSON对象的突变操作时，它首先转换为多个RDF，然后按常规进行处理。

每个JSON对象代表图中的单个节点。
```
注意JSON突变可通过gRPC客户端执行（例如Go客户端，JS客户端和Java客户端）获得，或者具有dgraph-js-http和cURL的HTTP客户端。 在此处查看[有关cURL的更多信息](https://dgraph.io/docs/mutations/json-mutation-format/#using-json-operations-via-curl)
```

### 设置文字值
设置新值时，突变操作消息中的set_json字段应包含一个JSON对象。

可以通过将键/值添加到JSON对象来设置文字值。 键代表谓词，值代表对象。

例如：
```
{
  "name": "diggy",
  "food": "pizza",
  "dgraph.type": "Mascot"
}
```
会转化成下面的RDF操作
```
_:blank-0 <name> "diggy" .
_:blank-0 <food> "pizza" .
_:blank-0 <dgraph.type> "Mascot" .
```
突变操作的结果也会包含一个map映射， 在这里把uid赋值给了原来的键blank-9. 你也可以定义自己的键
```
{
  "uid": "_:diggy",
  "name": "diggy",
  "food": "pizza",
  "dgraph.type": "Mascot"
}
```
在这里，是把键名设置成了 diggy, 随后uid的值直接赋值给它了。

### 语言支持
RDF和JSON突变操作之间的重要区别在于指定字符串值的语言。 在JSON中，语言标签会附加到边名称，而不是像RDF中那样附加到值。

例如，JSON突变操作
```
{
  "food": "taco",
  "rating@en": "tastes good",
  "rating@es": "sabe bien",
  "rating@fr": "c'est bon",
  "rating@it": "è buono",
  "dgraph.type": "Food"
}
```
它对等的RDF如下
```
_:blank-0 <food> "taco" .
_:blank-0 <dgraph.type> "Food" .
_:blank-0 <rating> "tastes good"@en .
_:blank-0 <rating> "sabe bien"@es .
_:blank-0 <rating> "c'est bon"@fr .
_:blank-0 <rating> "è buono"@it .
```
### 地理位置支持
JSON中也提供了对地理位置数据的支持。 地理位置数据作为JSON对象输入，其键为“类型”和“坐标”。 请记住，我们仅支持对Point，Polygon和MultiPolygon类型进行索引，但是我们可以存储其他类型的地理位置数据。 下面是一个示例：
```
{
  "food": "taco",
  "location": {
    "type": "Point",
    "coordinates": [1.0, 2.0]
  }
}
```

### 引用现有节点
如果JSON对象包含一个名为“uid”的字段，则该字段将被解释为图中现有节点的UID。 此机制使您可以引用现有节点。

例如：
```
{
  "uid": "0x467ba0",
  "food": "taco",
  "rating": "tastes good",
  "dgraph.type": "Food"
}
```
等价的RDF
```
<0x467ba0> <food> "taco" .
<0x467ba0> <rating> "tastes good" .
<0x467ba0> <dgraph.type> "Food" .
```

### 节点之间的边
节点之间的边与文字值表示的方法很相似，不同之处在于对象是JSON对象。

例如：
```
{
  "name": "Alice",
  "friend": {
    "name": "Betty"
  }
}
```
等价的RDF
```
_:blank-0 <name> "Alice" .
_:blank-0 <friend> _:blank-1 .
_:blank-1 <name> "Betty" .
```
突变的结果将包含分配给blank-0和blank-1节点的uid。 如果要在其他键下返回这些uid，则可以将uid字段指定为空白节点。
```
{
  "uid": "_:alice",
  "name": "Alice",
  "friend": {
    "uid": "_:bob",
    "name": "Betty"
  }
}
```
上面的JSON会被转成如下的RDF
```
_:alice <name> "Alice" .
_:alice <friend> _:bob .
_:bob <name> "Betty" .
```

现有节点的引用方式与添加文字值时相同。 例如。 链接两个现有节点：
```
{
  "uid": "0x123",
  "link": {
    "uid": "0x456"
  }
}
```
会被转成如下的操作
```
<0x123> <link> <0x456> .
```
```
注意常见的错误是尝试使用{"uid":"0x123","link":"0x456"}。 这将导致错误。 Dgraph将此JSON对象解释为将链接谓词设置为字符串“ 0x456”，这通常不是你期望的。
```
### 删除
删除突变也可以JSON格式发送。 要发送删除突变，请使用delete_json字段而不是Mutation消息中的set_json字段。
```
注意：如果您使用的是dgraph-js-http客户端或Ratel UI，请检查 [使用Raw HTTP或Ratel UI的JSON语法](https://dgraph.io/docs/mutations/json-mutation-format/#json-syntax-using-raw-http-or-ratel-ui) 部分。
```
使用删除突变时，始终必须引用现有节点。 因此，每个JSON对象中必须存在“ uid”字段。 要删除的谓词应设置为JSON值null。

例如，要删除食品等级：
```
{
  "uid": "0x467ba0",
  "rating": null
}
```

### 删除边
删除单个边与创建该边的JSON对象是一样的。例如。删除谓词链接从“ 0x123”到“ 0x456”：
```
{
  "uid": "0x123",
  "link": {
    "uid": "0x456"
  }
}
```
可以一次删除谓词从单个节点发出的所有边（相当于删除S P *）：
```
{
  "uid": "0x123",
  "link": null
}
```
如果未指定谓词，则将删除节点的所有已知出站边（到其他节点和文字值）（相当于删除S * *）。使用类型系统派生要删除的谓词。有关更多信息，请参考[RDF格式文档](https://dgraph.io/docs/mutations/json-mutation-format/#delete)和[类型系统](https://dgraph.io/docs/query-language/type-system/)部分。
```
{
  "uid": "0x123"
}
```
### 构面
可以使用|创建构面字符，用于分隔JSON对象字段名称中的谓词和构面键。这与用于在查询结果中显示构面的编码模式相同。例如。
```
{
  "name": "Carol",
  "name|initial": "C",
  "dgraph.type": "Person",
  "friend": {
    "name": "Daryl",
    "friend|close": "yes",
    "dgraph.type": "Person"
  }
}
```
产生以下RDF：
```
_:blank-0 <name> "Carol" (initial=C) .
_:blank-0 <dgraph.type> "Person" .
_:blank-0 <friend> _:blank-1 (close=yes) .
_:blank-1 <name> "Daryl" .
_:blank-1 <dgraph.type> "Person" .
```
构面不包含类型信息，但Dgraph将尝试从输入中猜测类型。如果构面的值可以解析为数字，则它将转换为浮点数或整数。如果可以将其解析为布尔值，则将其存储为布尔值。如果该值为字符串，则如果该字符串与Dgraph可以识别的时间格式之一（YYYY，MM-YYYY，DD-MM-YYYY，RFC339等）匹配，则将其存储为日期时间，否则会存储为双引号引用等同的字符串。如果您不想冒将构面数据误解为时间值的风险，最好将数字数据存储为int或float。

### 删除构面
删除构面的最简单方法是覆盖它。当构面的同一实体创建新突变时，现有构面将被自动删除。

例如：
```
<0x1> <name> "Carol" .
<0x1> <friend> <0x2> .
```
另一种方法是使用Upsert模块。

在下面的此查询中，我们将删除“Name”和“Friend”谓词中的“构面”。要覆盖构面，我们需要知道执行此操作的边的值，并使用函数“ val（var）”完成覆盖。
```
curl -H "Content-Type: application/rdf" -X POST localhost:8080/mutate?commitNow=true -d $'
upsert {
  query {
    user as var(func: eq(name, "Carol")){
      Name as name
      Friends as friend
    }
 }

  mutation {
    set {
      uid(user) <name> val(Name) .
      uid(user) <friend> uid(Friends) .
    }
  }
}' | jq
```

### 使用JSON创建列表并与之交互
架构：
```
testList: [string] .
```
```
{
  "testList": [
    "Grape",
    "Apple",
    "Strawberry",
    "Banana",
    "watermelon"
  ]
}
```

试着删除一个"Apple",
```
{
  q(func: has(testList)) {
    uid
    testList
  }
}
```
```
{
  "delete": {
    "uid": "0x6", #UID of the list.
    "testList": "Apple"
  }
}
```
你也可以同时删除多个值
```
{
  "delete": {
    "uid": "0x6",
    "testList": [
          "Strawberry",
          "Banana",
          "watermelon"
        ]
  }
}
```
```
注意 如果你想用 dgraph-js-http客户端或Ratel UI， 请查阅 通过原生 HTTP或Ratel UI使用JSON Syntax 这一部分
```
### 具有JSON的列表类型的构面
架构：
```
<name>: string @index(exact).
<nickname>: [string] .
```

要创建列表类型谓词，您需要在一个列表中指定所有值。所有谓词值的构面应一起指定。它以映射格式完成，列表内的谓词值索引是映射的键，其各自的构面值作为映射的值。没有构面值的谓词值将从构面映射中丢失。例如。
```
{
  "set": [
    {
      "uid": "_:Julian",
      "name": "Julian",
      "nickname": ["Jay-Jay", "Jules", "JB"],
      "nickname|kind": {
        "0": "first",
        "1": "official",
        "2": "CS-GO"
      }
    }
  ]
}
```
在上方您会看到我们有三个值可用于输入，且包含其各自构面的列表。您可以运行此查询来检查包含构面的列表：
```
{
   q(func: eq(name,"Julian")) {
    uid
    nickname @facets
   }
}
```
以后，如果要使用构面添加更多值，只需执行相同的过程，但是这次必须使用实际节点的UID来代替空白节点。
```
{
  "set": [
    {
      "uid": "0x3",
      "nickname|kind": "Internet",
      "nickname": "@JJ"
    }
  ]
}
```
最终结果是：
```
{
  "data": {
    "q": [
      {
        "uid": "0x3",
        "nickname|kind": {
          "0": "first",
          "1": "Internet",
          "2": "official",
          "3": "CS-GO"
        },
        "nickname": [
          "Jay-Jay",
          "@JJ",
          "Jules",
          "JB"
        ]
      }
    ]
  }
}
```
### 指定多个操作
当指定添加或删除突变时，可以使用JSON数组同时指定多个节点。

例如，以下JSON对象可用于添加两个新节点，每个节点都有一个name：
```
[
  {
    "name": "Edward"
  },
  {
    "name": "Fredric"
  }
]
```
### 使用Raw HTTP或Ratel UI的JSON语法
此语法可以在Ratel的最新版本中，dgraph-js-http客户端中或通过cURL使用。

您可以下载[适用于Linux，macOS或Windows的Ratel UI](https://discuss.dgraph.io/t/ratel-installer-for-linux-macos-and-windows-preview-version-ratel-update-from-v1-0-6/2884/)。

突变操作：
```
{
  "set": [
    {
      # One JSON obj in here
    },
    {
      # Another JSON obj in here for multiple operations
    }
  ]
}
```
删除：

删除操作与删除文字值和删除边相同。
```
{
  "delete": [
    {
      # One JSON obj in here
    },
    {
      # Another JSON obj in here for multiple operations
    }
  ]
}
```
### 通过cURL使用JSON操作
首先，您必须配置HTTP标头以指定内容类型。
```
-H 'Content-Type: application/json'
```
注意为了将jq用于JSON格式，您需要jq包。有关安装详细信息，请参见jq下载页面。您还可以将Python内置的json.tool模块与python -m json.tool一起使用以进行JSON格式设置。
```
curl -H "Content-Type: application/json" -X POST localhost:8080/mutate?commitNow=true -d $'
    {
      "set": [
        {
          "name": "Alice"
        },
        {
          "name": "Bob"
        }
      ]
    }' | jq

```

删除：
```
curl -H "Content-Type: application/json" -X POST localhost:8080/mutate?commitNow=true -d $'
    {
      "delete": [
        {
          "uid": "0xa"
        }
      ]
    }' | jq
```
使用JSON文件进行突变：
```
curl -H "Content-Type: application/json" -X POST localhost:8080/mutate?commitNow=true -d @data.json
```
data.json的内容如下所示：
```
{
  "set": [
    {
      "name": "Alice"
    },
    {
      "name": "Bob"
    }
  ]
}
```
对于通过HTTP进行的突变，JSON文件必须遵循相同的格式：具有“set”或“ delete”键的单个JSON对象，以及用于突变操作的JSON对象数组。如果您已经具有包含数据数组的文件，则可以使用jq将数据转换为正确的格式。例如，如果您的data.json文件如下所示：
```
[
  {
    "name": "Alice"
  },
  {
    "name": "Bob"
  }
]
```
那么您可以使用以下jq命令将数据转换为正确的格式，其中。 jq字符串中的代表data.json的内容：
```
cat data.json | jq '{set: .}'
```
```
{
  "set": [
    {
      "name": "Alice"
    },
    {
      "name": "Bob"
    }
  ]
}
```

### Upsert块
upsert块允许在单个请求中执行查询和突变。 upsert块包含一个查询块和一个或多个突变块。查询块中定义的变量，可以使用uid和val函数在突变操作中使用。

通常，upsert块的结构如下：
```
upsert {
  query <query block>
  [fragment <fragment block>]
  mutation <mutation block 1>
  [mutation <mutation block 2>]
  ...
}
```
执行upsert块还会返回执行突变之前对数据库状态执行的查询的响应。为了获得最新结果，我们应该提交突变后再执行另一个查询。

### uid函数
uid函数允许从查询块中定义的变量中提取UID。根据执行查询块的结果，可能有两种结果：

 - 如果变量为空，即没有节点与查询匹配，则在有进行设置操作的情况下，uid函数将返回新的UID，因此将其视为空白节点。另一方面，对于删除/删除操作，它不返回UID，因此该操作变为无操作且被忽略。一个空白节点在所有突变块中都会获得相同的UID。
 - 如果变量存储一个或多个UID，则uid函数返回存储在变量中的所有UID。在这种情况下，将对返回的所有UID一次执行所有操作。

### 值函数val
val函数允许从值变量中提取值。值变量存储从UID到其对应值的映射。因此，将val(v)替换为N-Quad中UID（主题）的映射中存储的值。如果变量v对于给定的UID没有值，则忽略该突变。 val函数也可以与聚合变量的结果一起使用，在这种情况下，突变中的所有UID都将使用聚合值进行更新。

uid函数示例
考虑具有以下架构的示例：
```
curl localhost:8080/alter -X POST -d $'
  name: string @index(term) .
  email: string @index(exact, trigram) @upsert .
  age: int @index(int) .' | jq
```
现在，假设我们要使用电子邮件和名称信息创建一个新用户。我们还希望确保一封电子邮件在数据库中恰好有一个相应的用户。为此，我们需要首先使用给定的电子邮件查询用户是否存在于数据库中。如果存在用户，则使用其UID更新名称信息。如果该用户不存在，我们将创建一个新用户并更新电子邮件和姓名信息。

我们可以使用upsert块来做到这一点，如下所示：
```
curl -H "Content-Type: application/rdf" -X POST localhost:8080/mutate?commitNow=true -d $'
upsert {
  query {
    q(func: eq(email, "user@company1.io")) {
      v as uid
      name
    }
  }

  mutation {
    set {
      uid(v) <name> "first last" .
      uid(v) <email> "user@company1.io" .
    }
  }
}' | jq
```
结果：
```
{
  "data": {
    "q": [],
    "code": "Success",
    "message": "Done",
    "uids": {
      "uid(v)": "0x1"
    }
  },
  "extensions": {...}
}
```
upsert块的查询部分, 将用户的UID和提供的电子邮件存储在变量v中。然后，突变操作部分从变量v提取UID，并将名称和电子邮件信息存储在数据库中。如果用户存在，则信息将更新。如果该用户不存在，则uid（v）被视为空白节点，并如上所述创建新用户。

如果我们再次运行相同的突变，则数据将被覆盖，并且不会创建新的uid。请注意，当再次执行突变并且数据映射（键q）包含在上一个upsert中创建的uid时，uid映射在结果中为空。
```
{
  "data": {
    "q": [
      {
        "uid": "0x1",
        "name": "first last"
      }
    ],
    "code": "Success",
    "message": "Done",
    "uids": {}
  },
  "extensions": {...}
}
```
我们可以使用json数据集实现相同的结果，如下所示：
```
curl -H "Content-Type: application/json" -X POST localhost:8080/mutate?commitNow=true -d $'
{
  "query": "{ q(func: eq(email, \\"user@company1.io\\")) {v as uid} }",
  "set":{
    "uid": "uid(v)",
    "age": "28"
  }
}' | jq
```
现在，我们要为具有相同电子邮件user@company1.io的同一用户增加年龄信息。我们可以使用upsert块执行以下操作：
```
curl -H "Content-Type: application/rdf" -X POST localhost:8080/mutate?commitNow=true -d $'
upsert {
  query {
    q(func: eq(email, "user@company1.io")) {
      v as uid
    }
  }

  mutation {
    set {
      uid(v) <age> "28" .
    }
  }
}' | jq
```
结果：
```
{
  "data": {
    "q": [
      {
        "uid": "0x1"
      }
    ],
    "code": "Success",
    "message": "Done",
    "uids": {}
  },
  "extensions": {...}
}
```

在这里，查询块查询电子邮件为user@company1.io的用户。它将用户的uid存储在变量v中。然后，突变块通过使用uid函数从变量v中提取uid来更新用户的年龄。

我们可以使用json数据集实现相同的结果，如下所示：
```
curl -H "Content-Type: application/json" -X POST localhost:8080/mutate?commitNow=true -d $'
{
  "query": "{ q(func: eq(email, \\"user@company1.io\\")) {v as uid} }",
  "set":{
    "uid": "uid(v)",
    "age": "28"
  }
}' | jq
```
如果我们只想在用户存在时执行变异，则可以使用条件更新。

### val函数示例
假设我们要将谓词年龄迁移到其他年龄。我们可以使用以下突变来做到这一点：
```
curl -H "Content-Type: application/rdf" -X POST localhost:8080/mutate?commitNow=true -d $'
upsert {
  query {
    v as var(func: has(age)) {
      a as age
    }
  }

  mutation {
    # we copy the values from the old predicate
    set {
      uid(v) <other> val(a) .
    }

    # and we delete the old predicate
    delete {
      uid(v) <age> * .
    }
  }
}' | jq
```

结果：
```
{
  "data": {
    "code": "Success",
    "message": "Done",
    "uids": {}
  },
  "extensions": {...}
}
```
在这里，变量a将存储从所有UID到其年龄的映射。然后，变异块将每个UID的相应年龄值存储在另一个谓词other中，并删除该年龄谓词。

我们可以使用json数据集实现相同的结果，如下所示：
```
curl -H "Content-Type: application/json" -X POST localhost:8080/mutate?commitNow=true -d $'{
  "query": "{ v as var(func: regexp(email, /.*@company1.io$/)) }",
  "delete": {
    "uid": "uid(v)",
    "age": null
  },
  "set": {
    "uid": "uid(v)",
    "other": "val(a)"
  }
}' | jq
```

### 批量删除示例
假设我们要从数据库中删除company1的所有用户。这可以使用upsert块在一个查询中完成，如下所示：
```
curl -H "Content-Type: application/rdf" -X POST localhost:8080/mutate?commitNow=true -d $'
upsert {
  query {
    v as var(func: regexp(email, /.*@company1.io$/))
  }

  mutation {
    delete {
      uid(v) <name> * .
      uid(v) <email> * .
      uid(v) <age> * .
    }
  }
}' | jq
```
我们可以使用json数据集实现相同的结果，如下所示：
```
curl -H "Content-Type: application/json" -X POST localhost:8080/mutate?commitNow=true -d '{
  "query": "{ v as var(func: regexp(email, /.*@company1.io$/)) }",
  "delete": {
    "uid": "uid(v)",
    "name": null,
    "email": null,
    "age": null
  }
}' | jq
```

### 有条件的更新Upsert
upsert块还允许使用@if指令指定条件突变块。仅当指定条件为真时，才执行突变。如果条件为假，则忽略该突变。条件Upsert的一般结构如下所示：
```
upsert {
  query <query block>
  [fragment <fragment block>]
  mutation [@if(<condition>)] <mutation block 1>
  [mutation [@if(<condition>)] <mutation block 2>]
  ...
}
```

@if指令接受查询块中定义的变量的条件，并且可以使用AND，OR和NOT进行连接。

### 有条件Upsert的示例
假设在前面的示例中，我们知道company1的员工人数少于100。为了安全起见，我们希望仅在变量v中存储的UID小于100但大于50时才执行该突变。这可以通过以下方式实现：
```
curl -H "Content-Type: application/rdf" -X POST localhost:8080/mutate?commitNow=true -d  $'
upsert {
  query {
    v as var(func: regexp(email, /.*@company1.io$/))
  }

  mutation @if(lt(len(v), 100) AND gt(len(v), 50)) {
    delete {
      uid(v) <name> * .
      uid(v) <email> * .
      uid(v) <age> * .
    }
  }
}' | jq
```
我们可以使用json数据集实现相同的结果，如下所示：
```
curl -H "Content-Type: application/json" -X POST localhost:8080/mutate?commitNow=true -d '{
  "query": "{ v as var(func: regexp(email, /.*@company1.io$/)) }",
  "cond": "@if(lt(len(v), 100) AND gt(len(v), 50))",
  "delete": {
    "uid": "uid(v)",
    "name": null,
    "email": null,
    "age": null
  }
}' | jq
```

### 多个突变块的例子
考虑具有以下架构的示例：
```
curl localhost:8080/alter -X POST -d $'
  name: string @index(term) .
  email: [string] @index(exact) @upsert .' | jq
```
假设我们有许多用户存储在我们的数据库中，每个用户都有一个或多个电子邮件地址。现在，我们得到两个属于同一用户的电子邮件地址。如果电子邮件地址属于数据库中的不同节点，我们要删除现有节点并创建一个新节点，并将这两个电子邮件都附加到该新节点上。否则，我们将使用这两封电子邮件创建/更新新的/现有的节点。
```
curl -H "Content-Type: application/rdf" -X POST localhost:8080/mutate?commitNow=true -d $'
upsert {
  query {
    # filter is needed to ensure that we do not get same UIDs in u1 and u2
    q1(func: eq(email, "user_email1@company1.io")) @filter(not(eq(email, "user_email2@company1.io"))) {
      u1 as uid
    }

    q2(func: eq(email, "user_email2@company1.io")) @filter(not(eq(email, "user_email1@company1.io"))) {
      u2 as uid
    }

    q3(func: eq(email, "user_email1@company1.io")) @filter(eq(email, "user_email2@company1.io")) {
      u3 as uid
    }
  }

  # case when both emails do not exist
  mutation @if(eq(len(u1), 0) AND eq(len(u2), 0) AND eq(len(u3), 0)) {
    set {
      _:user <name> "user" .
      _:user <dgraph.type> "Person" .
      _:user <email> "user_email1@company1.io" .
      _:user <email> "user_email2@company1.io" .
    }
  }

  # case when email1 exists but email2 does not
  mutation @if(eq(len(u1), 1) AND eq(len(u2), 0) AND eq(len(u3), 0)) {
    set {
      uid(u1) <email> "user_email2@company1.io" .
    }
  }

  # case when email1 does not exist but email2 exists
  mutation @if(eq(len(u1), 0) AND eq(len(u2), 1) AND eq(len(u3), 0)) {
    set {
      uid(u2) <email> "user_email1@company1.io" .
    }
  }

  # case when both emails exist and needs merging
  mutation @if(eq(len(u1), 1) AND eq(len(u2), 1) AND eq(len(u3), 0)) {
    set {
      _:user <name> "user" .
      _:user <dgraph.type> "Person" .
      _:user <email> "user_email1@company1.io" .
      _:user <email> "user_email2@company1.io" .
    }

    delete {
      uid(u1) <name> * .
      uid(u1) <email> * .
      uid(u2) <name> * .
      uid(u2) <email> * .
    }
  }
}' | jq

```
空数据库结果 结果
```
{
  "data": {
    "q1": [],
    "q2": [],
    "q3": [],
    "code": "Success",
    "message": "Done",
    "uids": {
      "user": "0x1"
    }
  },
  "extensions": {...}
}
```
如果 有邮件存储，并且分别归属不同结点，结果
```
{
  "data": {
    "q1": [
      {
        "uid": "0x2"
      }
    ],
    "q2": [
      {
        "uid": "0x3"
      }
    ],
    "q3": [],
    "code": "Success",
    "message": "Done",
    "uids": {
      "user": "0x4"
    }
  },
  "extensions": {...}
}
```
两个邮件都存在，且归属不同结点
```
{
  "data": {
    "q1": [
      {
        "uid": "0x2"
      }
    ],
    "q2": [
      {
        "uid": "0x3"
      }
    ],
    "q3": [],
    "code": "Success",
    "message": "Done",
    "uids": {
      "user": "0x4"
    }
  },
  "extensions": {...}
}
```
两个邮件存储，且归属于同一结点
```
{
  "data": {
    "q1": [],
    "q2": [],
    "q3": [
      {
        "uid": "0x4"
      }
    ],
    "code": "Success",
    "message": "Done",
    "uids": {}
  },
  "extensions": {...}
}
```
同样的JSON操作
```
curl -H "Content-Type: application/json" -X POST localhost:8080/mutate?commitNow=true -d '{
  "query": "{q1(func: eq(email, \"user_email1@company1.io\")) @filter(not(eq(email, \"user_email2@company1.io\"))) {u1 as uid} \n q2(func: eq(email, \"user_email2@company1.io\")) @filter(not(eq(email, \"user_email1@company1.io\"))) {u2 as uid} \n q3(func: eq(email, \"user_email1@company1.io\")) @filter(eq(email, \"user_email2@company1.io\")) {u3 as uid}}",
  "mutations": [
    {
      "cond": "@if(eq(len(u1), 0) AND eq(len(u2), 0) AND eq(len(u3), 0))",
      "set": [
        {
          "uid": "_:user",
          "name": "user",
          "dgraph.type": "Person"
        },
        {
          "uid": "_:user",
          "email": "user_email1@company1.io",
          "dgraph.type": "Person"
        },
        {
          "uid": "_:user",
          "email": "user_email2@company1.io",
          "dgraph.type": "Person"
        }
      ]
    },
    {
      "cond": "@if(eq(len(u1), 1) AND eq(len(u2), 0) AND eq(len(u3), 0))",
      "set": [
        {
          "uid": "uid(u1)",
          "email": "user_email2@company1.io",
          "dgraph.type": "Person"
        }
      ]
    },
    {
      "cond": "@if(eq(len(u1), 1) AND eq(len(u2), 0) AND eq(len(u3), 0))",
      "set": [
        {
          "uid": "uid(u2)",
          "email": "user_email1@company1.io",
          "dgraph.type": "Person"
        }
      ]
    },
    {
      "cond": "@if(eq(len(u1), 1) AND eq(len(u2), 1) AND eq(len(u3), 0))",
      "set": [
        {
          "uid": "_:user",
          "name": "user",
          "dgraph.type": "Person"
        },
        {
          "uid": "_:user",
          "email": "user_email1@company1.io",
          "dgraph.type": "Person"
        },
        {
          "uid": "_:user",
          "email": "user_email2@company1.io",
          "dgraph.type": "Person"
        }
      ],
      "delete": [
        {
          "uid": "uid(u1)",
          "name": null,
          "email": null
        },
        {
          "uid": "uid(u2)",
          "name": null,
          "email": null
        }
      ]
    }
  ]
}' | jq
```