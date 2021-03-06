欢迎使用Dgraph入门最后第八篇教程。

在上一教程中，我们了解了如何使用Dgraph的模糊搜索来构建类似Twitter的用户搜索功能。

在本教程中，我们将绘制旧金山周围旅游地点的图表，并帮助动物学家朋友玛丽和她的团队执行任务，利用Dgraph的地理定位功能保护鸟类。

您可能已经使用Google在您当前位置的一英里范围内找到您附近的餐馆或购物中心。这类程序就是典型的利用您的地理位置数据的应用。

地理定位已成为移动应用程序不可或缺的一部分，尤其是在最近十年中，随着智能手机的出现，围绕用户位置来增强应用功能的程序，已经超出了人们的想象。

以Uber为例，驾驶员和乘客的位置数据对于应用程序的功能至关重要。我们正在收集比以往更多的GPS数据，能够有效地存储和查询位置数据可以使您在竞争者中脱颖而出。

实际数据是相互关联的；他们并不孤立的；在位置数据方面，这一点更为重要。铁路网络，地图，路线的自然表示是图。

好消息是，Dgraph是世界上最先进的图形数据库，它具有包含位置数据的图进行有效存储和执行有用的查询功能。如果您想运行诸如“帮我我查找在金门大桥附近的酒店”，或“在金门公园附近查找所有旅游地点”，Dgraph必定可以完成你的要求。

首先，让我们学习如何在Dgraph中表示地理位置数据。

## 表示地理位置数据
您可以使用两种方法在Dgraph中表示位置数据：

- 点位置
点位置包含您所关注位置的地理坐标元组（纬度，经度）。

下图显示了巴黎埃菲尔铁塔的点位置以及经度和纬度。点位置对于表示精确位置很有用。例如，您在预订出租车时的位置或您的送货地址。

![模型](https://dgraph.io/docs//images/tutorials/8/b-paris.png)

- 多边形位置
仅使用点位置是无法表示分布在多个地理坐标中的地理实体。要表示诸如城市，湖泊或国家公园之类的地理实体，应使用多边形位置。

下边是一个例子：

![模型](https://dgraph.io/docs//images/tutorials/8/c-delhi.jpg)

上面的多边形围栏代表印度德里市。该多边形围栏或地理围栏是通过连接多个直线边界而形成的，它们使用格式为[（纬度，经度），（纬度，经度...）]的位置元组数组共同表示。每个元组对（2个元组和4个坐标）代表地理围栏的直线边界，多边形围栏可以包含任意数量的线。

让我们从建立一个简单的旧金山游客图开始，这里是图模型。

![模型](https://dgraph.io/docs//images/tutorials/8/a-graph.jpg)

上图具有由节点表示的三个实体：

- 市
城市节点代表旅游城市。我们的数据集仅包含旧金山市，并且图中的一个节点表示它。

- 位置
位置节点及其位置名称包含感兴趣的地点的点或多边形位置。

- 位置类型
位置类型由位置类型组成。我们的数据集中有四种类型的位置：动物园，博物馆，酒店或旅游景点。

带有地理坐标的位置节点的酒店还包含其定价信息。

有多种方法可以对同一张图进行建模。例如，位置类型可以只是位置节点的属性或谓词，而不是其自身的节点。

您想要执行的查询或您要探索的关系会影响建模决策。本教程的目标不是获得理想的图模型，而是使用简单的数据集来演示Dgraph的地理定位功能。

在本教程的其余部分中，我们将代表城市的节点称为城市节点，将代表位置的节点称为位置节点，将代表位置类型的节点称为位置类型节点。

### 这些节点之间的关系如下：

- 每个城市节点city node都通过has_location边连接到位置节点。
- 每个位置节点location node都通过has_type边连接到代表位置类型的节点。
`注意：Dgraph允许您使用其类型系统功能为节点关联一个或多个类型，目前，我们正在使用没有类型的节点，我们将在以后的教程中了解节点的类型系统。如果要探索节点的类型系统功能，[请参考此页面](https://docs.dgraph.io/query-language/#type-system)。`

这是我们的样本数据集。打开Ratel，转到mutate选项卡，粘贴突变操作，然后单击Run。
```
{
  "set": [
    {
      "city": "San Francisco",
      "uid": "_:SFO",
      "has_location": [
        {
          "name": "USS Pampanito",
          "location": {
            "type": "Polygon",
            "coordinates": [
              [
                [
                  -122.4160088,
                  37.8096674
                ],
                [
                  -122.4161147,
                  37.8097628
                ],
                [
                  -122.4162064,
                  37.8098357
                ],
                [
                  -122.4163467,
                  37.8099312
                ],
                [
                  -122.416527,
                  37.8100471
                ],
                [
                  -122.4167504,
                  37.8101792
                ],
                [
                  -122.4168272,
                  37.8102137
                ],
                [
                  -122.4167719,
                  37.8101612
                ],
                [
                  -122.4165683,
                  37.8100108
                ],
                [
                  -122.4163888,
                  37.8098923
                ],
                [
                  -122.4162492,
                  37.8097986
                ],
                [
                  -122.4161469,
                  37.8097352
                ],
                [
                  -122.4160088,
                  37.8096674
                ]
              ]
            ]
          },
          "has_type": [
            {
              "uid": "_:museum",
              "loc_type": "Museum"
            }
          ]
        },
        {
          "name": "Alameda Naval Air Museum",
          "location": {
            "type": "Polygon",
            "coordinates": [
              [
                [
                  -122.2995054,
                  37.7813924
                ],
                [
                  -122.2988538,
                  37.7813582
                ],
                [
                  -122.2988421,
                  37.7814972
                ],
                [
                  -122.2994937,
                  37.7815314
                ],
                [
                  -122.2995054,
                  37.7813924
                ]
              ]
            ]
          },
          "street": "Ferry Point Road",
          "has_type": [
            {
              "uid": "_:museum"
            }
          ]
        },
        {
          "name": "Burlingame Museum of PEZ Memorabilia",
          "location": {
            "type": "Polygon",
            "coordinates": [
              [
                [
                  -122.3441509,
                  37.5792003
                ],
                [
                  -122.3438207,
                  37.5794257
                ],
                [
                  -122.3438987,
                  37.5794587
                ],
                [
                  -122.3442289,
                  37.5792333
                ],
                [
                  -122.3441509,
                  37.5792003
                ]
              ]
            ]
          },
          "street": "California Drive",
          "has_type": [
            {
              "uid": "_:museum"
            }
          ]
        },
        {
          "name": "Carriage Inn",
          "location": {
            "type": "Polygon",
            "coordinates": [
              [
                [
                  -122.3441509,
                  37.5792003
                ],
                [
                  -122.3438207,
                  37.5794257
                ],
                [
                  -122.3438987,
                  37.5794587
                ],
                [
                  -122.3442289,
                  37.5792333
                ],
                [
                  -122.3441509,
                  37.5792003
                ]
              ]
            ]
          },
          "street": "7th street",
          "price_per_night": 350.00,
          "has_type": [
            {
              "uid": "_:hotel",
              "loc_type": "Hotel"
            }
          ]
        },
        {
          "name": "Lombard Motor In",
          "location": {
            "type": "Polygon",
            "coordinates": [
              [
                [
                  -122.4260484,
                  37.8009811
                ],
                [
                  -122.4260137,
                  37.8007969
                ],
                [
                  -122.4259083,
                  37.80081
                ],
                [
                  -122.4258724,
                  37.8008144
                ],
                [
                  -122.4257962,
                  37.8008239
                ],
                [
                  -122.4256354,
                  37.8008438
                ],
                [
                  -122.4256729,
                  37.8010277
                ],
                [
                  -122.4260484,
                  37.8009811
                ]
              ]
            ]
          },
          "street": "Lombard Street",
          "price_per_night": 400.00,
          "has_type": [
            {
              "uid": "_:hotel"
            }
          ]
        },
        {
          "name": "Holiday Inn San Francisco Golden Gateway",
          "location": {
            "type": "Polygon",
            "coordinates": [
              [
                [
                  -122.4214895,
                  37.7896108
                ],
                [
                  -122.4215628,
                  37.7899798
                ],
                [
                  -122.4215712,
                  37.790022
                ],
                [
                  -122.4215987,
                  37.7901606
                ],
                [
                  -122.4221004,
                  37.7900985
                ],
                [
                  -122.4221044,
                  37.790098
                ],
                [
                  -122.4219952,
                  37.7895481
                ],
                [
                  -122.4218207,
                  37.78957
                ],
                [
                  -122.4216158,
                  37.7895961
                ],
                [
                  -122.4214895,
                  37.7896108
                ]
              ]
            ]
          },
          "street": "Van Ness Avenue",
          "price_per_night": 250.00,
          "has_type": [
            {
              "uid": "_:hotel"
            }
          ]
        },
        {
          "name": "Golden Gate Bridge",
          "location": {
            "type": "Polygon",
            "coordinates": [
              [
                [
                  -122.479784,
                  37.8288329
                ],
                [
                  -122.4775646,
                  37.8096291
                ],
                [
                  -122.4775538,
                  37.8095165
                ],
                [
                  -122.4775465,
                  37.8093304
                ],
                [
                  -122.4775823,
                  37.8093296
                ],
                [
                  -122.4775387,
                  37.8089749
                ],
                [
                  -122.4773545,
                  37.8089887
                ],
                [
                  -122.4773402,
                  37.8089575
                ],
                [
                  -122.4772752,
                  37.8088285
                ],
                [
                  -122.4772084,
                  37.8087099
                ],
                [
                  -122.4771322,
                  37.8085903
                ],
                [
                  -122.4770518,
                  37.8084793
                ],
                [
                  -122.4769647,
                  37.8083687
                ],
                [
                  -122.4766802,
                  37.8080091
                ],
                [
                  -122.4766629,
                  37.8080195
                ],
                [
                  -122.4765701,
                  37.8080751
                ],
                [
                  -122.476475,
                  37.8081322
                ],
                [
                  -122.4764106,
                  37.8081708
                ],
                [
                  -122.476396,
                  37.8081795
                ],
                [
                  -122.4764936,
                  37.8082814
                ],
                [
                  -122.476591,
                  37.8083823
                ],
                [
                  -122.4766888,
                  37.8084949
                ],
                [
                  -122.47677,
                  37.808598
                ],
                [
                  -122.4768444,
                  37.8087008
                ],
                [
                  -122.4769144,
                  37.8088105
                ],
                [
                  -122.4769763,
                  37.8089206
                ],
                [
                  -122.4770373,
                  37.8090416
                ],
                [
                  -122.477086,
                  37.809151
                ],
                [
                  -122.4771219,
                  37.8092501
                ],
                [
                  -122.4771529,
                  37.809347
                ],
                [
                  -122.477179,
                  37.8094517
                ],
                [
                  -122.4772003,
                  37.809556
                ],
                [
                  -122.4772159,
                  37.8096583
                ],
                [
                  -122.4794624,
                  37.8288561
                ],
                [
                  -122.4794098,
                  37.82886
                ],
                [
                  -122.4794817,
                  37.8294742
                ],
                [
                  -122.4794505,
                  37.8294765
                ],
                [
                  -122.4794585,
                  37.8295453
                ],
                [
                  -122.4795423,
                  37.8295391
                ],
                [
                  -122.4796312,
                  37.8302987
                ],
                [
                  -122.4796495,
                  37.8304478
                ],
                [
                  -122.4796698,
                  37.8306078
                ],
                [
                  -122.4796903,
                  37.830746
                ],
                [
                  -122.4797182,
                  37.8308784
                ],
                [
                  -122.4797544,
                  37.83102
                ],
                [
                  -122.479799,
                  37.8311522
                ],
                [
                  -122.4798502,
                  37.8312845
                ],
                [
                  -122.4799025,
                  37.8314139
                ],
                [
                  -122.4799654,
                  37.8315458
                ],
                [
                  -122.4800346,
                  37.8316718
                ],
                [
                  -122.4801231,
                  37.8318137
                ],
                [
                  -122.4802112,
                  37.8319368
                ],
                [
                  -122.4803028,
                  37.8320547
                ],
                [
                  -122.4804046,
                  37.8321657
                ],
                [
                  -122.4805121,
                  37.8322792
                ],
                [
                  -122.4805883,
                  37.8323459
                ],
                [
                  -122.4805934,
                  37.8323502
                ],
                [
                  -122.4807146,
                  37.8323294
                ],
                [
                  -122.4808917,
                  37.832299
                ],
                [
                  -122.4809526,
                  37.8322548
                ],
                [
                  -122.4809672,
                  37.8322442
                ],
                [
                  -122.4808396,
                  37.8321298
                ],
                [
                  -122.4807166,
                  37.8320077
                ],
                [
                  -122.4806215,
                  37.8319052
                ],
                [
                  -122.4805254,
                  37.8317908
                ],
                [
                  -122.4804447,
                  37.8316857
                ],
                [
                  -122.4803548,
                  37.8315539
                ],
                [
                  -122.4802858,
                  37.8314395
                ],
                [
                  -122.4802227,
                  37.8313237
                ],
                [
                  -122.4801667,
                  37.8312051
                ],
                [
                  -122.4801133,
                  37.8310812
                ],
                [
                  -122.4800723,
                  37.8309602
                ],
                [
                  -122.4800376,
                  37.8308265
                ],
                [
                  -122.4800087,
                  37.8307005
                ],
                [
                  -122.4799884,
                  37.8305759
                ],
                [
                  -122.4799682,
                  37.8304181
                ],
                [
                  -122.4799501,
                  37.8302699
                ],
                [
                  -122.4798628,
                  37.8295146
                ],
                [
                  -122.4799157,
                  37.8295107
                ],
                [
                  -122.4798451,
                  37.8289002
                ],
                [
                  -122.4798369,
                  37.828829
                ],
                [
                  -122.479784,
                  37.8288329
                ]
              ]
            ]
          },
          "street": "Golden Gate Bridge",
          "has_type": [
            {
              "uid": "_:attraction",
              "loc_type": "Tourist Attraction"
            }
          ]
        },
        {
          "name": "Carriage Inn",
          "location": {
            "type": "Polygon",
            "coordinates": [
              [
                [
                  -122.3441509,
                  37.5792003
                ],
                [
                  -122.3438207,
                  37.5794257
                ],
                [
                  -122.3438987,
                  37.5794587
                ],
                [
                  -122.3442289,
                  37.5792333
                ],
                [
                  -122.3441509,
                  37.5792003
                ]
              ]
            ]
          },
          "street": "7th street",
          "has_type": [
            {
              "uid": "_:attraction"
            }
          ]
        },
        {
          "name": "San Francisco Zoo",
          "location": {
            "type": "Polygon",
            "coordinates": [
              [
                [
                  -122.5036126,
                  37.7308562
                ],
                [
                  -122.5028991,
                  37.7305879
                ],
                [
                  -122.5028274,
                  37.7305622
                ],
                [
                  -122.5027812,
                  37.7305477
                ],
                [
                  -122.5026992,
                  37.7305269
                ],
                [
                  -122.5026211,
                  37.7305141
                ],
                [
                  -122.5025342,
                  37.7305081
                ],
                [
                  -122.5024478,
                  37.7305103
                ],
                [
                  -122.5023667,
                  37.7305221
                ],
                [
                  -122.5022769,
                  37.7305423
                ],
                [
                  -122.5017546,
                  37.7307008
                ],
                [
                  -122.5006917,
                  37.7311277
                ],
                [
                  -122.4992484,
                  37.7317075
                ],
                [
                  -122.4991414,
                  37.7317614
                ],
                [
                  -122.4990379,
                  37.7318177
                ],
                [
                  -122.4989369,
                  37.7318762
                ],
                [
                  -122.4988408,
                  37.731938
                ],
                [
                  -122.4987386,
                  37.7320142
                ],
                [
                  -122.4986377,
                  37.732092
                ],
                [
                  -122.4978359,
                  37.7328712
                ],
                [
                  -122.4979122,
                  37.7333232
                ],
                [
                  -122.4979485,
                  37.7333909
                ],
                [
                  -122.4980162,
                  37.7334494
                ],
                [
                  -122.4980945,
                  37.7334801
                ],
                [
                  -122.4989553,
                  37.7337384
                ],
                [
                  -122.4990551,
                  37.7337743
                ],
                [
                  -122.4991479,
                  37.7338184
                ],
                [
                  -122.4992482,
                  37.7338769
                ],
                [
                  -122.4993518,
                  37.7339426
                ],
                [
                  -122.4997605,
                  37.7342142
                ],
                [
                  -122.4997578,
                  37.7343433
                ],
                [
                  -122.5001258,
                  37.7345486
                ],
                [
                  -122.5003425,
                  37.7346621
                ],
                [
                  -122.5005576,
                  37.7347566
                ],
                [
                  -122.5007622,
                  37.7348353
                ],
                [
                  -122.500956,
                  37.7349063
                ],
                [
                  -122.5011438,
                  37.7349706
                ],
                [
                  -122.5011677,
                  37.7349215
                ],
                [
                  -122.5013556,
                  37.7349785
                ],
                [
                  -122.5013329,
                  37.7350294
                ],
                [
                  -122.5015181,
                  37.7350801
                ],
                [
                  -122.5017265,
                  37.7351269
                ],
                [
                  -122.5019229,
                  37.735164
                ],
                [
                  -122.5021252,
                  37.7351953
                ],
                [
                  -122.5023116,
                  37.7352187
                ],
                [
                  -122.50246,
                  37.7352327
                ],
                [
                  -122.5026074,
                  37.7352433
                ],
                [
                  -122.5027534,
                  37.7352501
                ],
                [
                  -122.5029253,
                  37.7352536
                ],
                [
                  -122.5029246,
                  37.735286
                ],
                [
                  -122.5033453,
                  37.7352858
                ],
                [
                  -122.5038376,
                  37.7352855
                ],
                [
                  -122.5038374,
                  37.7352516
                ],
                [
                  -122.5054006,
                  37.7352553
                ],
                [
                  -122.5056182,
                  37.7352867
                ],
                [
                  -122.5061792,
                  37.7352946
                ],
                [
                  -122.5061848,
                  37.7352696
                ],
                [
                  -122.5063093,
                  37.7352671
                ],
                [
                  -122.5063297,
                  37.7352886
                ],
                [
                  -122.5064719,
                  37.7352881
                ],
                [
                  -122.5064722,
                  37.735256
                ],
                [
                  -122.506505,
                  37.7352268
                ],
                [
                  -122.5065452,
                  37.7352287
                ],
                [
                  -122.5065508,
                  37.7351214
                ],
                [
                  -122.5065135,
                  37.7350885
                ],
                [
                  -122.5065011,
                  37.7351479
                ],
                [
                  -122.5062471,
                  37.7351127
                ],
                [
                  -122.5059669,
                  37.7349341
                ],
                [
                  -122.5060092,
                  37.7348205
                ],
                [
                  -122.5060405,
                  37.7347219
                ],
                [
                  -122.5060611,
                  37.734624
                ],
                [
                  -122.5060726,
                  37.7345101
                ],
                [
                  -122.5060758,
                  37.73439
                ],
                [
                  -122.5060658,
                  37.73427
                ],
                [
                  -122.5065549,
                  37.7342676
                ],
                [
                  -122.5067262,
                  37.7340364
                ],
                [
                  -122.506795,
                  37.7340317
                ],
                [
                  -122.5068355,
                  37.733827
                ],
                [
                  -122.5068791,
                  37.7335407
                ],
                [
                  -122.5068869,
                  37.7334106
                ],
                [
                  -122.5068877,
                  37.733281
                ],
                [
                  -122.5068713,
                  37.7329795
                ],
                [
                  -122.5068598,
                  37.7328652
                ],
                [
                  -122.506808,
                  37.7325954
                ],
                [
                  -122.5067837,
                  37.732482
                ],
                [
                  -122.5067561,
                  37.7323727
                ],
                [
                  -122.5066387,
                  37.7319688
                ],
                [
                  -122.5066273,
                  37.731939
                ],
                [
                  -122.5066106,
                  37.7319109
                ],
                [
                  -122.506581,
                  37.7318869
                ],
                [
                  -122.5065404,
                  37.731872
                ],
                [
                  -122.5064982,
                  37.7318679
                ],
                [
                  -122.5064615,
                  37.731878
                ],
                [
                  -122.5064297,
                  37.7318936
                ],
                [
                  -122.5063553,
                  37.7317985
                ],
                [
                  -122.5063872,
                  37.7317679
                ],
                [
                  -122.5064106,
                  37.7317374
                ],
                [
                  -122.5064136,
                  37.7317109
                ],
                [
                  -122.5063998,
                  37.7316828
                ],
                [
                  -122.5063753,
                  37.7316581
                ],
                [
                  -122.5061296,
                  37.7314636
                ],
                [
                  -122.5061417,
                  37.731453
                ],
                [
                  -122.5060145,
                  37.7313791
                ],
                [
                  -122.5057839,
                  37.7312678
                ],
                [
                  -122.5054352,
                  37.7311479
                ],
                [
                  -122.5043701,
                  37.7310447
                ],
                [
                  -122.5042805,
                  37.7310343
                ],
                [
                  -122.5041861,
                  37.7310189
                ],
                [
                  -122.5041155,
                  37.7310037
                ],
                [
                  -122.5036126,
                  37.7308562
                ]
              ]
            ]
          },
          "street": "San Francisco Zoo",
          "has_type": [
            {
              "uid": "_:zoo",
              "loc_type": "Zoo"
            }
          ]
        },
        {
          "name": "Flamingo Park",
          "location": {
            "type": "Polygon",
            "coordinates": [
              [
                [
                  -122.5033039,
                  37.7334601
                ],
                [
                  -122.5032811,
                  37.7334601
                ],
                [
                  -122.503261,
                  37.7334601
                ],
                [
                  -122.5032208,
                  37.7334495
                ],
                [
                  -122.5031846,
                  37.7334357
                ],
                [
                  -122.5031806,
                  37.7334718
                ],
                [
                  -122.5031685,
                  37.7334962
                ],
                [
                  -122.5031336,
                  37.7335078
                ],
                [
                  -122.503128,
                  37.7335189
                ],
                [
                  -122.5031222,
                  37.7335205
                ],
                [
                  -122.5030954,
                  37.7335269
                ],
                [
                  -122.5030692,
                  37.7335444
                ],
                [
                  -122.5030699,
                  37.7335677
                ],
                [
                  -122.5030813,
                  37.7335868
                ],
                [
                  -122.5031034,
                  37.7335948
                ],
                [
                  -122.5031511,
                  37.73359
                ],
                [
                  -122.5031933,
                  37.7335916
                ],
                [
                  -122.5032228,
                  37.7336022
                ],
                [
                  -122.5032697,
                  37.7335937
                ],
                [
                  -122.5033194,
                  37.7335874
                ],
                [
                  -122.5033515,
                  37.7335693
                ],
                [
                  -122.5033723,
                  37.7335518
                ],
                [
                  -122.503369,
                  37.7335068
                ],
                [
                  -122.5033603,
                  37.7334702
                ],
                [
                  -122.5033462,
                  37.7334474
                ],
                [
                  -122.5033073,
                  37.733449
                ],
                [
                  -122.5033039,
                  37.7334601
                ]
              ]
            ]
          },
          "street": "San Francisco Zoo",
          "has_type": [
            {
              "uid": "_:zoo"
            }
          ]
        },
        {
          "name": "Peace Lantern",
          "location": {
            "type": "Point",
            "coordinates": [
              -122.4705776,
              37.7701084
            ]
          },
          "street": "Golden Gate Park",
          "has_type": [
            {
              "uid": "_:attraction"
            }
          ]
        },
        {
          "name": "Buddha",
          "location": {
            "type": "Point",
            "coordinates": [
              -122.469942,
              37.7703183
            ]
          },
          "street": "Golden Gate Park",
          "has_type": [
            {
              "uid": "_:attraction"
            }
          ]
        },
        {
          "name": "Japanese Tea Garden",
          "location": {
            "type": "Polygon",
            "coordinates": [
              [
                [
                  -122.4692131,
                  37.7705116
                ],
                [
                  -122.4698998,
                  37.7710069
                ],
                [
                  -122.4702431,
                  37.7710137
                ],
                [
                  -122.4707248,
                  37.7708919
                ],
                [
                  -122.4708911,
                  37.7701541
                ],
                [
                  -122.4708428,
                  37.7700354
                ],
                [
                  -122.4703492,
                  37.7695011
                ],
                [
                  -122.4699255,
                  37.7693989
                ],
                [
                  -122.4692131,
                  37.7705116
                ]
              ]
            ]
          },
          "street": "Golden Gate Park",
          "has_type": [
            {
              "uid": "_:attraction"
            }
          ]
        }
      ]
    }
  ]
}
```
`注意：如果您不熟悉此突变语法，请参考第一个教程以学习Dgraph中的突变操作基础。`

运行以下查询以获取整个图：
```
{
  entire_graph(func: has(city)) {
    city
    has_location {
    name 
    has_type {
      loc_type
      }
    }
  }
}
```
下面是相应的图
![如图](https://dgraph.io/docs//images/tutorials/8/d-full-graph.png)
我们的图有：

- 一个蓝色城市节点。我们只有一个节点代表旧金山市。
- 绿色的是位置节点。我们共有13个地点。
- 粉色节点代表位置类型。我们的数据集中有四种位置：博物馆，动物园，酒店和旅游景点。
您还可以看到Dgraph从“模式”选项卡中自动检测到谓词的数据类型，并且位置谓词已被自动分配为地理类型(geo)。

![类型检测](https://dgraph.io/docs//images/tutorials/8/e-schema.png)

注意：请查看以前的教程，以了解有关Dgraph中数据类型的更多信息。

在开始之前，请先向动物学家玛丽问好，她一直致力于保护各种鸟类的研究。

在本教程的其余部分中，让我们帮助Mary和她的动物学家团队执行保护鸟类的任务。

## 输入旧金山：酒店预订
玛丽所做的几个研究项目表明，火烈鸟在栖息地有大量水体时会壮成长。

她的团队获得批准可以为旧金山动物园中的火烈鸟增加水源，并且她的团队已准备好前往旧金山旅行，玛丽可以远程监控团队的进度。

她的队友希望呆在金门大桥附近，以便他们每天早晨可以在金门大桥周围骑自行车，享受微风和日出。

让我们帮助他们找到距离金门大桥较近的酒店，我们将用Dgraph的地理位置功能来做到这一点。

Dgraph提供了多种功能来查询地理位置数据。要使用它们，您必须先设置地理位置索引。

转到“架构”选项卡，然后在位置谓词上设置索引。

![地理索引](https://dgraph.io/docs//images/tutorials/8/f-index.png)

在位置谓词上设置地理位置索引后，您可以使用Dgraph的内置功能在附近查找金门大桥附近的酒店。

这是near函数的语法：near(geo-predicate，[long，lat]，distance)。

Near函数匹配并返回数据库中存储的所有地理谓词的值符合要求的位置，这些谓词在用户提供的geojson坐标[long，lat]的距离单位xxxx米之内。

让我们搜索距离金门大桥7公里以内的酒店。

转到查询标签，在下面粘贴查询，然后单击运行。
```
{
  find_hotel(func: near(location, [-122.479784,37.82883295],7000) )  {
    name
    has_type {
      loc_type
    }
  }
}
```
![如图](https://dgraph.io/docs//images/tutorials/8/g-near-1.png)
等乖！ 搜索不仅返回酒店，还返回距金门大桥点坐标7公里以内的所有其他位置。

让我们使用@filter函数过滤仅包含酒店的搜索结果。 您可以查看之前的第三篇教程，以熟悉有关使用@filter指令的讨论。
```
{
  find_hotel(func: near(location, [-122.479784,37.82883295],7000)) {
    name
    has_type @filter(eq(loc_type, "hotel")){
      loc_type 
    }
  }
}
```
我们又忘了加eq对应谓词的索引了。
![如图](https://dgraph.io/docs//images/tutorials/8/h-near-2.png)
让我们为loc_tpe加个hash索引后，再运行一下查询吧。
![如图](https://dgraph.io/docs//images/tutorials/8/i-near-3.png)
![如图](https://dgraph.io/docs//images/tutorials/8/j-near-4.png)

`注意：请参阅本系列的第三篇教程，以了解有关Dgraph中的哈希索引和比较函数的更多信息。`

搜索结果仍然包含表示不是酒店的位置的节点。这是因为根查询首先找到距指定点位置7公里之内的所有位置节点，然后再遍历位置类型节点的同时应用过滤器。

只能在根级别上过滤位置节点中的谓词，并且如果不遍历位置类型节点，就无法过滤位置类型。

在遍历位置类型节点时，我们可以使用过滤器仅选择酒店。我们能否将过滤器级联或冒泡到根级别，以便最终结果中只有旅馆呢？

结果肯定是肯定的啊！您可以使用@cascade指令来完成这一功能。

@cascade指令可帮助您将应用于内部查询遍历的过滤器级联或冒泡到根级别节点，这样做，我们只会在结果中获得酒店的位置。
```
{
  find_hotel(func: near(location, [-122.479784,37.82883295],7000)) @cascade {
   name
   price_per_night
   has_type @filter(eq(loc_type,"Hotel")){
     loc_type
    }
  }
}
```
![如图](https://dgraph.io/docs//images/tutorials/8/k-near-5.png)
看！ 您可以在结果中看到，在查询中添加@cascade指令后，结果中仅显示类型为hotel的位置。

结果是我们有两家酒店，其中一家超过了每晚300$的预算。 让我们添加另一个过滤器来搜索每晚价格低于$300的酒店。

每个酒店的价格信息及其坐标都存储在位置节点中，因此，价格过滤器应位于查询的根级别，而不是遍历位置类型节点的级别。

在开始运行查询之前，请不要忘记在price_per_night谓词上添加索引。
```
{
  find_hotel(func: near(location, [-122.479784,37.82883295],7000)) @cascade @filter(le(price_per_night, 300)){
    name
    price_per_night
    has_type @filter(eq(loc_type,"Hotel")){
      loc_type
    }
  }
}
```

![如图](https://dgraph.io/docs//images/tutorials/8/m-final-result.png)
现在，我们拥有一家经济实惠的酒店，并且靠近金门大桥！

## 总结
在本教程中，我们了解了Dgraph中的地理定位功能，并帮助Mary的团队预订了金桥附近的酒店。

在下一个教程中，如果还有话，会有些意思吧。