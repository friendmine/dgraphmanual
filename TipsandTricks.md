### GraphQL+-: Tips and Tricks

### 获得样本数据
使用has函数获取一些示例节点。

```
{
  result(func: has(director.film), first: 10) {
    uid
    expand(_all_)
  }
}
```
响应
```
{
  "data": {
    "result": [
      {
        "uid": "0x2b",
        "name@en": "Zehra Yiğit"
      },
      {
        "uid": "0x48",
        "name@en": "Charlton Heston",
        "name@fi": "Charlton Heston",
        "name@zh": "查尔顿·赫斯顿",
        "name@hu": "Charlton Heston",
        "name@sl": "Charlton Heston",
        "name@da": "Charlton Heston",
        "name@ca": "Charlton Heston",
        "name@ko": "찰턴 헤스턴",
        "name@pt": "Charlton Heston",
        "name@no": "Charlton Heston",
        "name@th": "ชาร์ลตัน เฮสตัน",
        "name@cs": "Charlton Heston",
        "name@tr": "Charlton Heston",
        "name@hr": "Charlton Heston",
        "name@es": "Charlton Heston",
        "name@sk": "Charlton Heston",
        "name@nl": "Charlton Heston",
        "name@it": "Charlton Heston",
        "name@fa": "چارلتون هستون",
        "name@vi": "Charlton Heston",
        "name@ru": "Чарлтон Хестон",
        "name@fr": "Charlton Heston",
        "name@ja": "チャールトン・ヘストン",
        "name@ro": "Charlton Heston",
        "name@uk": "Чарлтон Гестон",
        "name@id": "Charlton Heston",
        "name@iw": "צ'רלטון הסטון",
        "name@zh-Hant": "卻爾登·希斯頓",
        "name@sr": "Чарлтон Хестон",
        "name@fil": "Charlton Heston",
        "name@el": "Τσάρλτον Ίστον",
        "name@sv": "Charlton Heston",
        "name@pl": "Charlton Heston",
        "name@ar": "شارلتون هيستون",
        "name@es-419": "Charlton Heston",
        "name@bg": "Чарлтън Хестън",
        "name@lv": "Čārltons Hestons",
        "name@de": "Charlton Heston"
      },
      {
        "uid": "0x4e",
        "name@en": "Rajeev Sharma",
        "name@hi": "राजीव शर्मा"
      },
      {
        "uid": "0x9d",
        "name@en": "Gabriel Nussbaum"
      },
      {
        "uid": "0xb4",
        "name@en": "James Wharton O'Keefe",
        "name@sl": "James Wharton O'Keefe"
      },
      {
        "uid": "0xb5",
        "name@en": "Roger Daltrey",
        "name@fi": "Roger Daltrey",
        "name@hu": "Roger Daltrey",
        "name@sl": "Roger Daltrey",
        "name@da": "Roger Daltrey",
        "name@ca": "Roger Daltrey",
        "name@ko": "로저 돌트리",
        "name@pt": "Roger Daltrey",
        "name@no": "Roger Daltrey",
        "name@cs": "Roger Daltrey",
        "name@es": "Roger Daltrey",
        "name@nl": "Roger Daltrey",
        "name@it": "Roger Daltrey",
        "name@fa": "راجر دالتری",
        "name@ru": "Роджер Долтри",
        "name@fr": "Roger Daltrey",
        "name@ja": "ロジャー・ダルトリー",
        "name@ro": "Roger Daltrey",
        "name@id": "Roger Daltrey",
        "name@iw": "רוג'ר דלטרי",
        "name@zh-Hant": "羅傑·達爾屈",
        "name@sr": "Роџер Далтри",
        "name@el": "Ρότζερ Ντάλτρεϊ",
        "name@sv": "Roger Daltrey",
        "name@pl": "Roger Daltrey",
        "name@ar": "روجر دالتري",
        "name@de": "Roger Daltrey"
      },
      {
        "uid": "0xd5",
        "name@en": "Ducho Mundrov"
      },
      {
        "uid": "0x148",
        "name@en": "Alberto Santana"
      },
      {
        "uid": "0x176",
        "name@en": "Norman Lloyd",
        "name@fi": "Norman Lloyd",
        "name@da": "Norman Lloyd",
        "name@pt": "Norman Lloyd",
        "name@nl": "Norman Lloyd",
        "name@it": "Norman Lloyd",
        "name@fa": "نورمن لوید",
        "name@fr": "Norman Lloyd",
        "name@ja": "ノーマン・ロイド",
        "name@ro": "Norman Lloyd",
        "name@fil": "Norman Lloyd",
        "name@sv": "Norman Lloyd",
        "name@pl": "Norman Lloyd",
        "name@de": "Norman Lloyd"
      },
      {
        "uid": "0x177",
        "name@en": "Buster Keaton",
        "name@fi": "Buster Keaton",
        "name@zh": "巴斯特·基顿",
        "name@hu": "Buster Keaton",
        "name@sl": "Buster Keaton",
        "name@da": "Buster Keaton",
        "name@ca": "Buster Keaton",
        "name@ko": "버스터 키턴",
        "name@pt": "Buster Keaton",
        "name@no": "Buster Keaton",
        "name@th": "บัสเตอร์ คีตัน",
        "name@cs": "Buster Keaton",
        "name@tr": "Buster Keaton",
        "name@hr": "Buster Keaton",
        "name@es": "Buster Keaton",
        "name@sk": "Buster Keaton",
        "name@hi": "बस्टर कीटन",
        "name@nl": "Buster Keaton",
        "name@it": "Buster Keaton",
        "name@fa": "باستر کیتون",
        "name@vi": "Buster Keaton",
        "name@ru": "Бастер Китон",
        "name@fr": "Buster Keaton",
        "name@ja": "バスター・キートン",
        "name@ro": "Buster Keaton",
        "name@uk": "Бастер Кітон",
        "name@id": "Buster Keaton",
        "name@iw": "באסטר קיטון",
        "name@zh-Hant": "巴斯特·基頓",
        "name@sr": "Бастер Китон",
        "name@el": "Μπάστερ Κίτον",
        "name@sv": "Buster Keaton",
        "name@pl": "Buster Keaton",
        "name@ar": "باستر كيتون",
        "name@bg": "Бъстър Кийтън",
        "name@et": "Buster Keaton",
        "name@lv": "Basters Kītons",
        "name@de": "Buster Keaton"
      }
    ]
  }
}
```
### 计算连接数目

使用expand(_all_)扩展节点的边，然后将其分配给变量。 现在可以使用该变量在唯一的相邻节点上进行迭代。 然后使用count(uid)计算一个块中的节点数。
```
{
  uids(func: has(director.film), first: 1) {
    uid
    expand(_all_) { u as uid }
  }

  result(func: uid(u)) {
    count(uid)
  }
}
```
响应
```
{
  "data": {
    "uids": [
      {
        "uid": "0x2b",
        "name@en": "Zehra Yiğit",
        "director.film": [
          {
            "uid": "0x195896"
          }
        ]
      }
    ],
    "result": [
      {
        "count": 1
      }
    ]
  }
}
```

### 在没有索引的谓词上搜索
在值变量中使用has函数搜索未索引的谓词。

```
{
  var(func: has(festival.date_founded)) {
    p as festival.date_founded
  }
  query(func: eq(val(p), "1961-01-01T00:00:00Z")) {
      uid
      name@en
      name@ru
      name@pl
      festival.date_founded
      festival.focus { name@en }
      festival.individual_festivals { total : count(uid) }
  }
}
```
响应
```
{
  "data": {
    "query": [
      {
        "uid": "0x2d005",
        "name@en": "Kraków Film Festival",
        "name@ru": "Кинофестиваль в Кракове",
        "name@pl": "Krakowski Festiwal Filmowy",
        "festival.date_founded": "1961-01-01T00:00:00Z",
        "festival.focus": [
          {
            "name@en": "Short Film"
          }
        ],
        "festival.individual_festivals": [
          {
            "total": 55
          }
        ]
      }
    ]
  }
}
```

#### 按嵌套节点值对边排序
Dgraph排序是基于子图的单个级别。 要按更深级别的值对级别进行排序，需要使用查询变量将嵌套的值提升到要排序的边的级别。

示例：从史蒂文·斯皮尔伯格（Steven Spielberg）电影中按字母顺序排列所有演员。 无法从主演边一次遍历演员的名字； 可通过performance.actor访问该名称。
```
{
  spielbergMovies as var(func: allofterms(name@en, "steven spielberg")) {
    name@en
    director.film (orderasc: name@en, first: 1) {
      starring {
        performance.actor {
          ActorName as name@en
        }
        # Stars is a uid-to-value map mapping
        # starring edges to performance.actor names
        Stars as min(val(ActorName))
      }
    }
  }

  movies(func: uid(spielbergMovies)) @cascade {
    name@en
    director.film (orderasc: name@en, first: 1) {
      name@en
      starring (orderasc: val(Stars)) {
        performance.actor {
          name@en
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
    "movies": [
      {
        "name@en": "Steven Spielberg",
        "director.film": [
          {
            "name@en": "1941",
            "starring": [
              {
                "performance.actor": [
                  {
                    "name@en": "Akio Mitamura"
                  }
                ]
              },
              {
                "performance.actor": [
                  {
                    "name@en": "Andy Tennant"
                  }
                ]
              },
              {
                "performance.actor": [
                  {
                    "name@en": "Antoinette Molinari"
                  }
                ]
              },
              {
                "performance.actor": [
                  {
                    "name@en": "Audrey Landers"
                  }
                ]
              },
              {
                "performance.actor": [
                  {
                    "name@en": "Barbara Gannen"
                  }
                ]
              },
              {
                "performance.actor": [
                  {
                    "name@en": "Bobby Di Cicco"
                  }
                ]
              },
              {
                "performance.actor": [
                  {
                    "name@en": "Brad Gorman"
                  }
                ]
              },
              {
                "performance.actor": [
                  {
                    "name@en": "Brian Frishman"
                  }
                ]
              },
              {
                "performance.actor": [
                  {
                    "name@en": "Carol Ann Williams"
                  }
                ]
              },
              {
                "performance.actor": [
                  {
                    "name@en": "Carol Culver"
                  }
                ]
              },
              {
                "performance.actor": [
                  {
                    "name@en": "Christian Zika"
                  }
                ]
              },
              {
                "performance.actor": [
                  {
                    "name@en": "Christopher Lee"
                  }
                ]
              },
              {
                "performance.actor": [
                  {
                    "name@en": "Dan Aykroyd"
                  }
                ]
              },
              {
                "performance.actor": [
                  {
                    "name@en": "Dan McNally"
                  }
                ]
              },
              {
                "performance.actor": [
                  {
                    "name@en": "David F. Cameron"
                  }
                ]
              },
              {
                "performance.actor": [
                  {
                    "name@en": "David Lander"
                  }
                ]
              },
              {
                "performance.actor": [
                  {
                    "name@en": "Debbie Rothstein"
                  }
                ]
              },
              {
                "performance.actor": [
                  {
                    "name@en": "Deborah Benson"
                  }
                ]
              },
              {
                "performance.actor": [
                  {
                    "name@en": "Denise Cheshire"
                  }
                ]
              },
              {
                "performance.actor": [
                  {
                    "name@en": "Denise Gallup"
                  }
                ]
              },
              {
                "performance.actor": [
                  {
                    "name@en": "Dian Gallup"
                  }
                ]
              },
              {
                "performance.actor": [
                  {
                    "name@en": "Diane Hardin"
                  }
                ]
              },
              {
                "performance.actor": [
                  {
                    "name@en": "Dianne Kay"
                  }
                ]
              },
              {
                "performance.actor": [
                  {
                    "name@en": "Dick Miller"
                  }
                ]
              },
              {
                "performance.actor": [
                  {
                    "name@en": "Don Calfa"
                  }
                ]
              },
              {
                "performance.actor": [
                  {
                    "name@en": "Donovan Scott"
                  }
                ]
              },
              {
                "performance.actor": [
                  {
                    "name@en": "Dub Taylor"
                  }
                ]
              },
              {
                "performance.actor": [
                  {
                    "name@en": "Eddie Deezen"
                  }
                ]
              },
              {
                "performance.actor": [
                  {
                    "name@en": "Edward Hampton Beagle"
                  }
                ]
              },
              {
                "performance.actor": [
                  {
                    "name@en": "Elisha Cook, Jr."
                  }
                ]
              },
              {
                "performance.actor": [
                  {
                    "name@en": "Frank McRae"
                  }
                ]
              },
              {
                "performance.actor": [
                  {
                    "name@en": "Frank Verroca"
                  }
                ]
              },
              {
                "performance.actor": [
                  {
                    "name@en": "Galen Thompson"
                  }
                ]
              },
              {
                "performance.actor": [
                  {
                    "name@en": "Gary Cervantes"
                  }
                ]
              },
              {
                "performance.actor": [
                  {
                    "name@en": "Geno Silva"
                  }
                ]
              },
              {
                "performance.actor": [
                  {
                    "name@en": "Gray Frederickson"
                  }
                ]
              },
              {
                "performance.actor": [
                  {
                    "name@en": "Hank Robinson"
                  }
                ]
              },
              {
                "performance.actor": [
                  {
                    "name@en": "Hiroshi Shimizu"
                  }
                ]
              },
              {
                "performance.actor": [
                  {
                    "name@en": "Iggie Wolfington"
                  }
                ]
              },
              {
                "performance.actor": [
                  {
                    "name@en": "J. Patrick McNamara"
                  }
                ]
              },
              {
                "performance.actor": [
                  {
                    "name@en": "Jack Thibeau"
                  }
                ]
              },
              {
                "performance.actor": [
                  {
                    "name@en": "James Caan"
                  }
                ]
              },
              {
                "performance.actor": [
                  {
                    "name@en": "Jenny Williams"
                  }
                ]
              },
              {
                "performance.actor": [
                  {
                    "name@en": "Jerry Hardin"
                  }
                ]
              },
              {
                "performance.actor": [
                  {
                    "name@en": "Joe Flaherty"
                  }
                ]
              },
              {
                "performance.actor": [
                  {
                    "name@en": "John Belushi"
                  }
                ]
              },
              {
                "performance.actor": [
                  {
                    "name@en": "John Candy"
                  }
                ]
              },
              {
                "performance.actor": [
                  {
                    "name@en": "John Landis"
                  }
                ]
              },
              {
                "performance.actor": [
                  {
                    "name@en": "John McKee"
                  }
                ]
              },
              {
                "performance.actor": [
                  {
                    "name@en": "John Voldstad"
                  }
                ]
              },
              {
                "performance.actor": [
                  {
                    "name@en": "Jordan Brian"
                  }
                ]
              },
              {
                "performance.actor": [
                  {
                    "name@en": "Kerry Sherman"
                  }
                ]
              },
              {
                "performance.actor": [
                  {
                    "name@en": "Lionel Stander"
                  }
                ]
              },
              {
                "performance.actor": [
                  {
                    "name@en": "Lorraine Gary"
                  }
                ]
              },
              {
                "performance.actor": [
                  {
                    "name@en": "Lucille Benson"
                  }
                ]
              },
              {
                "performance.actor": [
                  {
                    "name@en": "Lucinda Dooling"
                  }
                ]
              },
              {
                "performance.actor": [
                  {
                    "name@en": "Luis Contreras"
                  }
                ]
              },
              {
                "performance.actor": [
                  {
                    "name@en": "Marjorie Gaines"
                  }
                ]
              },
              {
                "performance.actor": [
                  {
                    "name@en": "Mark Carlton"
                  }
                ]
              },
              {
                "performance.actor": [
                  {
                    "name@en": "Mark Marias"
                  }
                ]
              },
              {
                "performance.actor": [
                  {
                    "name@en": "Maureen Teefy"
                  }
                ]
              },
              {
                "performance.actor": [
                  {
                    "name@en": "Michael McKean"
                  }
                ]
              },
              {
                "performance.actor": [
                  {
                    "name@en": "Mickey Rourke"
                  }
                ]
              },
              {
                "performance.actor": [
                  {
                    "name@en": "Mo Lauren"
                  }
                ]
              },
              {
                "performance.actor": [
                  {
                    "name@en": "Murray Hamilton"
                  }
                ]
              },
              {
                "performance.actor": [
                  {
                    "name@en": "Nancy Allen"
                  }
                ]
              },
              {
                "performance.actor": [
                  {
                    "name@en": "Ned Beatty"
                  }
                ]
              },
              {
                "performance.actor": [
                  {
                    "name@en": "Patti LuPone"
                  }
                ]
              },
              {
                "performance.actor": [
                  {
                    "name@en": "Paul Cloud"
                  }
                ]
              },
              {
                "performance.actor": [
                  {
                    "name@en": "Penny Marshall"
                  }
                ]
              },
              {
                "performance.actor": [
                  {
                    "name@en": "Perry Lang"
                  }
                ]
              },
              {
                "performance.actor": [
                  {
                    "name@en": "Rita Taggart"
                  }
                ]
              },
              {
                "performance.actor": [
                  {
                    "name@en": "Robert Houston"
                  }
                ]
              },
              {
                "performance.actor": [
                  {
                    "name@en": "Robert Stack"
                  }
                ]
              },
              {
                "performance.actor": [
                  {
                    "name@en": "Ronnie McMillan"
                  }
                ]
              },
              {
                "performance.actor": [
                  {
                    "name@en": "Samuel Fuller"
                  }
                ]
              },
              {
                "performance.actor": [
                  {
                    "name@en": "Slim Pickens"
                  }
                ]
              },
              {
                "performance.actor": [
                  {
                    "name@en": "Steve Moriarty"
                  }
                ]
              },
              {
                "performance.actor": [
                  {
                    "name@en": "Steven Mond"
                  }
                ]
              },
              {
                "performance.actor": [
                  {
                    "name@en": "Susan Backlinie"
                  }
                ]
              },
              {
                "performance.actor": [
                  {
                    "name@en": "Sydney Lassick"
                  }
                ]
              },
              {
                "performance.actor": [
                  {
                    "name@en": "Tim Matheson"
                  }
                ]
              },
              {
                "performance.actor": [
                  {
                    "name@en": "Toshiro Mifune"
                  }
                ]
              },
              {
                "performance.actor": [
                  {
                    "name@en": "Treat Williams"
                  }
                ]
              },
              {
                "performance.actor": [
                  {
                    "name@en": "Trish Garland"
                  }
                ]
              },
              {
                "performance.actor": [
                  {
                    "name@en": "Vito Carenzo"
                  }
                ]
              },
              {
                "performance.actor": [
                  {
                    "name@en": "Walter Olkewicz"
                  }
                ]
              },
              {
                "performance.actor": [
                  {
                    "name@en": "Warren Oates"
                  }
                ]
              },
              {
                "performance.actor": [
                  {
                    "name@en": "Wendie Jo Sperber"
                  }
                ]
              },
              {
                "performance.actor": [
                  {
                    "name@en": "Whitney Rydbeck"
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

### 通过使用变量获得唯一的结果
为了获得独特的结果，请将节点的边分配给变量。 现在可以使用该变量在唯一节点上进行迭代。

示例：从史蒂芬·斯皮尔伯格（Steven Spielberg）执导的所有电影中获得所有独特的流派。

```
{
  var(func: eq(name@en, "Steven Spielberg")) {
    director.film {
      genres as genre
    }
  }

  q(func: uid(genres)) {
    name@.
  }
}
```
响应
```
{
  "data": {
    "q": [
      {
        "name@.": "War film"
      },
      {
        "name@.": "Horror"
      },
      {
        "name@.": "Crime"
      },
      {
        "name@.": "Childhood Drama"
      },
      {
        "name@.": "Science Fiction"
      },
      {
        "name@.": "Thriller"
      },
      {
        "name@.": "Haunted House Film"
      },
      {
        "name@.": "Alien Film"
      },
      {
        "name@.": "Film noir"
      },
      {
        "name@.": "Political thriller"
      },
      {
        "name@.": "Chase Movie"
      },
      {
        "name@.": "Crime Fiction"
      },
      {
        "name@.": "Adventure Comedy"
      },
      {
        "name@.": "Family"
      },
      {
        "name@.": "Biographical film"
      },
      {
        "name@.": "Film adaptation"
      },
      {
        "name@.": "Animation"
      },
      {
        "name@.": "Romantic comedy"
      },
      {
        "name@.": "Children's Fantasy"
      },
      {
        "name@.": "Road movie"
      },
      {
        "name@.": "Adventure Film"
      },
      {
        "name@.": "Romance Film"
      },
      {
        "name@.": "Action/Adventure"
      },
      {
        "name@.": "Action Comedy"
      },
      {
        "name@.": "Drama"
      },
      {
        "name@.": "Comedy"
      },
      {
        "name@.": "Historical fiction"
      },
      {
        "name@.": "Disaster Film"
      },
      {
        "name@.": "Fantasy"
      },
      {
        "name@.": "Comedy-drama"
      },
      {
        "name@.": "Coming of age"
      },
      {
        "name@.": "Historical period drama"
      },
      {
        "name@.": "Short Film"
      },
      {
        "name@.": "Epic film"
      },
      {
        "name@.": "Action Film"
      },
      {
        "name@.": "Children's/Family"
      },
      {
        "name@.": "Political drama"
      },
      {
        "name@.": "Doomsday film"
      },
      {
        "name@.": "Mystery"
      },
      {
        "name@.": "Screwball comedy"
      },
      {
        "name@.": "Suspense"
      },
      {
        "name@.": "Costume Adventure"
      },
      {
        "name@.": "Fantasy Adventure"
      },
      {
        "name@.": "Adventure"
      },
      {
        "name@.": "Mystery film"
      },
      {
        "name@.": "Airplanes and airports"
      },
      {
        "name@.": "Future noir"
      },
      {
        "name@.": "Dystopia"
      }
    ]
  }
}
```
