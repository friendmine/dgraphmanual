## 索引
当Dgraph基于过滤器搜索字符串，日期或其他值时，它需要一个索引才能有效工作。我们已经看到了有关如何在架构突变中指定索引的示例。

int，float，geo和date具有默认索引，但是string需要选择哪种索引类型的选项。可以为相同的字符串值谓词构建多个索引。

对于字符串，以下索引可用

 - term 术语（默认），用于allofterms和anyofterms
 - exact 精确用于不等式-匹配整个字符串
 - hash 哈希值跟精确索引一样，但散列字符串-适用于长字符串
 - fulltext 全文用于使用alloftext和anyoftext进行全文搜索
 - trigram 正则表达式使用的三字母组合

将多个索引指定为@index的参数。

应用架构突变操作添加影片数据的索引， Dgraph需要几分钟的时间来计算所有电影数据的索引。在突变操作完成后继续进行其它的操作。

您可能会发现Docker在突变运行期间停掉了Dgraph进程。那可能是一个内存问题-增加Docker允许的内存量，然后重新启动Dgraph，加载初始架构和数据，然后应用此突变操作。

### 术语搜寻 term
通过术语索引，函数allofterms和anyofterms可以查找与所有列出的术语或任何列出的术语匹配的字符串。 先前查询使用了这些搜索。

尝试这两个选项并注意差异点。

除了要求找到的节点与术语匹配的名称外，查询没有其他限制，因此将返回演员和导演。

### 正则表达式
正则表达式需要trigram索引。trigram组合词是3个连续字符或符号的子串。

如trigram的trigram为：tri，rig，igr，gra和ram。

有效的正则表达式将转换为针对索引的三元组查询。 Dgraph在trigram索引中搜索可能的匹配项，然后针对可能的条件运行完整的正则表达式。

每个正则表达式必须至少匹配一个trigram组。

仔细思考正则表达式的查询是个好主意。查询匹配的可能字母越多，则针对完整正则表达式检查的结果越多，查询的效率就越低。同样，反之亦然。正则表达式执行的trigram越多，Dgraph就可以更好地利用索引，将可能的结果集变小。

因此，编写有效的正则表达式通常意味着使用长的子字符串，并避免尽可能多的选择。

可选的i可以跟在最后/后面，以表示不区分大小写的搜索。

娜塔莉·波特曼（Natalie Portman）是我们数据库中唯一一位在外星人电影中饰演过的演员，并且在她的名字中嵌入了“外星人”一词。

### 精确索引Exact和不等
有了精确索引(exact), 不等式可以在过滤器中使用针对字符串的不等式。 可以将[精确的索引](https://dgraph.io/tour/search/1/)添加到字符串上后，再尝试查询。

哈希hash索引允许使用eq来过滤字符串。

### 全文搜索
全文搜索是Google对网页所做的工作。 它与术语匹配不同，因为它会尝试尊重语言，语法和时态。 例如，搜索词run将与包含run，running和ran的文档匹配。

它与字词不完全匹配，而是利用了下面的技术

 - 词干分析：找到一个共同的基本词，即使在时态，复数/单数形式或其他变体形式上有差异但仍然匹配，并且
 - 停用词：删除诸如and和or之类的词，而且因为这些词出现得太频繁可能无法搜索。
如果将alloftext更改为allofterms，请注意区别。

### 全文搜索：语言支持
全文搜索仅适用于具有词干处理程序和停用词列表的语言。

目前，Dgraph支持
|LANGUAGE|	COUNTRY CODE|
|--|--|
|Danish	|da|
|Dutch|	nl|
|English|	en|
|Finnish|	fi|
|French	|fr|
|German|	de||
|Hungarian|	hu|
|Italian|	it|
|Norwegian|	no|
|Portuguese|	pt|
|Romanian|	ro|
|Russian|	ru|
|Spanish|	es|
|Swedish|	sv|
|Turkish|	tr|
|Chinese|	zh|
|Japanese|	ja|
|Korean|	ko|

### Geo queries : Near
暂时没有.