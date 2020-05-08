# ETF_Trading
<h3> 工作进展</h3>



|  日期  |                   完成                   |
| :----: | :--------------------------------------: |
| 5月6日 |           自动获取ETF所有代号            |
|        |       历史时间序列数据从Uqer上获取       |
|        |      每日数据通过爬虫从上交所上更新      |
|        |         数据包括：申赎单，成分表         |
|        | 自动获取ETF指定代号的制定时间段内的数据  |
|        | 数据包括：昨收、是否涨跌停、振幅、换手率 |
|        |        数据格式为：复合dataframe         |
|        | ~~自动可视化展示异常点的~~  是否有必要？ |
| 5月8日 |               计算跟踪误差               |

### 工作目标

| 研究方向                  | 备注                                                         |
| ------------------------- | ------------------------------------------------------------ |
| ETF和基金之间存在跟踪误差 | 申购赎回通道的存在联通了两个市场，ETF套利得以实现，成为目前证券市场上为数不多的“低风险、高收益”投资模式 |

### 阶段

|  阶段  |                   完成                   |
| :----: | :--------------------------------------: |
| 第一阶段 |           获得基本数据            |
|        |       完成信息整理       |
|        |      自动处理数据流      |
|        |      策略研究      |
|     第二阶段   |         自动交易         |
|        | 算法交易，订单拆分  |
|   第三阶段     | 模拟交易，回测 |

------



### ETF交易

问题回答

1. 赎回时，各种现金替代都是给的什么？

   |     方式     |     上证ETF50      |
   | :----------: | :----------------: |
   | 禁止现金替代 | 股票，全部都是股票 |
   | 允许现金替代 | 股票，全部都是股票 |
   | 必须现金替代 | 现金，全部都是现金 |

   退补现金替代是深交所的规则，上交所没有。

   |     方式     |     沪深ETF300     |
   | :----------: | :----------------: |
   | 退补现金替代 | 现金，全部都是现金 |

   所以赎回的时候不存在选择现金替代的问题？不能自己选？特别是对于允许现金替代，因为我申购的时候是可以自己选的。

2. 申购、交易时的成交价格时如何确定的

   |                   | 集合竞价      | 连续竞价                                       |
   | ----------------- | ------------- | ---------------------------------------------- |
   | 申赎阶段          | 不参与        | 时间优先，实时申报。(基金具体交易操作不能透露) |
   | 买卖 ETF 份额阶段 | <u>不参与</u> | 自己操作，与买股票一致                         |

3. 申购赎回时的现金差额

   1. 现金差额和预估部分 是和现金替代分开的。

      |          |                           现金替代                           |                    现金差额、预估现金部分                    |
      | :------: | :----------------------------------------------------------: | :----------------------------------------------------------: |
      |   对象   |                           个股退补                           |                       对于基金净值而言                       |
      |   范围   |                             局部                             |                             整体                             |
      | 产生原因 | 券商实际买入的股票价格和申赎清单上的不同。<u>这部分的钱后期会退给我，这部分的钱叫做现金差额嘛？</u> -> 不是。这完全是两码事。 | 基金实际价值和一揽子股票的价值不同。基金的投资目标主要时跟踪指数，但是还是会有5%以下的成分是用于实现其他目的（1）有些股票未必可买（2）指数可能会更换成分股。所以基金里会有少量现金、债券、股指期货等。 |

   2. 现金替代的是对于单个股票而言的，是指**个股的退补**。在账户里面会有“**补券信息**”界面，可供查询与操作。

      | 操作 | 日期 |              |                                                    |
      | ---- | ---- | :----------: | :------------------------------------------------: |
      | 申购 | T    | 预估现金部分 | 为负[券商 -> 我，转账]，为正[我 -> 券商，**冻结**] |
      |      | T+1  |   现金差额   |   为负[券商 -> 我，转账]，为正[我 -> 券商，转账]   |
      | 赎回 | T    | 预估现金部分 | 为负[我 -> 券商，**冻结**]，为正[券商 -> 我，转账] |
      |      | T+1  |   现金差额   |  为负[我 -> 券商，转账]， 为正[券商 -> 我，转账]   |

      |          |                     预估现金部分                      |                      现金差额                       |
      | :------: | :---------------------------------------------------: | :-------------------------------------------------: |
      | 计算方法 | T-1日实际基金净值 - T日一揽子申购股票和现金替代的价值 | T日实际基金净值 - T日一揽子申购股票和现金替代的价值 |

      T日申购，T+1日多退少补的金额：[券商 -> 我，转账] =

       T日预估现金部分（在T日公布） - T日的现金差额（在T+1日公布）

      |   T日公布的基本信息   |         项目         |        举例        |
      | :-------------------: | :------------------: | :----------------: |
      | **T - 1日的信息内容** |       现金差额       |    ￥-13,585.95    |
      |                       | 最小申购赎回单位净值 |   ￥2,217,841.05   |
      |                       |     基金额分净值     |      ￥2.464       |
      |                       |                      |                    |
      |    **T日信息内容**    |     预估现金部分     |    ￥-13,585.95    |
      |                       |   现金替代比例上限   |        30%         |
      |                       |  是否需要公布 IOPV   |         是         |
      |                       |   最小申购赎回单位   |      900,000       |
      |                       |   申购赎回允许情况   | 允许申购、允许赎回 |
      |                       |                      |                    |
      |  **成分股信息部分**   |       股票代码       |       000001       |
      |                       |       股票简称       |      深发展Ａ      |
      |                       |       股票数量       |        1200        |
      |                       |     现金替代标志     |        退补        |
      |                       |   现金替代溢价比例   |        10%         |
      |                       |     固定替代金额     |     19,968.00      |

      

5. 申购时，实际参考什么来预估我的开销呢？

   |                               |                    参考建议                    |                  计算方法                   |
   | :---------------------------: | :--------------------------------------------: | :-----------------------------------------: |
   |   昨日最小申购赎回单位净值    |                    仅供参考                    |                T日开盘前公布                |
   |      IPOV * 最小申赎单位      | IOPV用于讨论，很常见。IOPV太小，套利空间不大。 | IOPV 15s更新一次；最小申赎单位T日开盘前公布 |
   | 申赎清单一揽子股票 + 现金替代 |                     最精确                     | 自己计算，根据自己购买股票的价格来计算成本  |

   

   | 补充信息                                                     |
   | ------------------------------------------------------------ |
   | 申赎清单里面对于一揽子股票的价格参考的方式有两种（计算固定替代金额的时候用到） |
   | （1）昨日收盘价                                              |
   | （2）上交所公布的今日预计开盘价                              |
   | 不同基金公司采用不同的参考方式                               |

   

6. 其他补充信息

   | 其他                                                         |
   | ------------------------------------------------------------ |
   | 即时给ETF份额                                                |
   | 对于沪深ETF300，开设的账户再上交所，所以对于沪市的股票用的是票。对于深市的股票用的是钱。我想自己买好了深市的股票去申购是不可以的。 |
   | 对于申购的50%的上限，对是于整个篮子的股票的50%。（对于种类还是数量，是按价值算的，就是允许现金替代里总量的50%，而且这个50%几乎是固定的，最近几年不会变动，但是还是用实时的数据好一点）对于单只股票而言，只有0或者1的选择：只有选择全部都用现金替代，或者选择全部都不用现金替代。 |
   | 申购的时候，ETF份额是实时到账的。但是股票的是否能买到是在T+2日内的。原则是时间优先。（<u>采用的是买一价吗？</u>还在研究 -> 不对外公布。<u>对于开盘之前可以预提交，然后用开盘价吗？</u>-> 不可以,需要在连续竞价阶段提交申购赎回） |
   | 申购赎回的交易开始时间是 9: 30. 结束时间是 5: 00.            |
   | 对于必须现金替代，采用的价格是停止交易时的价格？-> 是的，就是最新价格。 |

   

7. 其他待解决问题

   | 需要解决                                   |
   | ------------------------------------------ |
   | IOPV的计算，会产生误差吗？有套利的空间吗？ |
   | 写一个自动计算最小申购开销的类 (完成)            |

   



