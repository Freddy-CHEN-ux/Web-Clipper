# 续费升级需求调研文档 - 数据中心项目 - Confluence
[产品网站数据调研-梁凯龙.xlsx](https://confluence.autel.com/download/attachments/50932669/%E4%BA%A7%E5%93%81%E7%BD%91%E7%AB%99%E6%95%B0%E6%8D%AE%E8%B0%83%E7%A0%94.xlsx?version=1&modificationDate=1646365089000&api=v2)

[产品网站数据调研.docx](https://confluence.autel.com/download/attachments/50932669/%E4%BA%A7%E5%93%81%E7%BD%91%E7%AB%99%E6%95%B0%E6%8D%AE%E8%B0%83%E7%A0%94.docx?version=1&modificationDate=1646365089000&api=v2)

[续费升级项目调研文档-党林涛.md](https://confluence.autel.com/download/attachments/50932669/%E7%BB%AD%E8%B4%B9%E5%8D%87%E7%BA%A7%E9%A1%B9%E7%9B%AE%E8%B0%83%E7%A0%94.md?version=1&modificationDate=1646365089000&api=v2)

dt_productinfo
--------------

产品信息表

```

```

### 问题

1、regPwd 是否为敏感字段？敏感字段！

2、serialNoPublicKey 是否为敏感字段？敏感字段！

mysql→ods

1) 同步方式：每天全量同步

**有个问题：该表同步时如果将 num-mappers 设置为1才能同步成功！需要定位原因！**

ods→dwd

1）proDate 格式化 yyyy-MM-dd

2）regTime 格式化为 yyyy-MM-dd

dwd→dwb

1) 注意：不使用该表的 proTypeCode 与 dt\_productforsealer.code 进行关联，由于 dt\_salecontract 表的proTypeCode是以外键的形式存在，准确性更高。

```

```

dt_orderinfo
------------

软件续费订单信息表

```

```

### 问题

1、orderDate 等与具体时间有关的字段，其时间是哪个时区的时间？

      UTC0时区

2、orderState 的枚举值 0 和 1 分别代表什么？

         ![](https://confluence.autel.com/download/thumbnails/50932669/image2022-2-14_19-55-34.png?version=1&modificationDate=1646365089000&api=v2)

     0-无效的，（取消订单场景）

     1-有效的

3、orderPayState 的枚举值 0 、1、 2、 3、 4 分别代表什么以及状态如何流转？

          ![](https://confluence.autel.com/download/thumbnails/50932669/image2022-2-14_19-54-31.png?version=1&modificationDate=1646365089000&api=v2)

      0 - 未支付

      1- 已支付

      4- 退款

      2和3 不用了

4、recConfirmState 的枚举值有哪些？分别代表什么以及状态如何流转？

         ![](https://confluence.autel.com/download/thumbnails/50932669/image2022-2-14_19-53-39.png?version=1&modificationDate=1646365089000&api=v2)

      orderPayState 一致，无需作为条件

5、discountMoney 文档中注释为“折扣后金额”，那意思是否为订单金额都为 0 或 1 ？需确认该字段的准确含义？

      ![](https://confluence.autel.com/download/attachments/50932669/image2022-2-14_19-58-31.png?version=1&modificationDate=1646365089000&api=v2)

      无用！

6、processState 的枚举值有哪些？分别代表什么以及状态如何流转？

     ![](https://confluence.autel.com/download/thumbnails/50932669/image2022-2-14_20-16-19.png?version=1&modificationDate=1646365089000&api=v2)

     无用与orderPayState一样

7、该表是否存在物理删除数据的场景？是否存在将 delState 置为 1 （已删除）的场景？目前表中没有 delState=1的数据！

      无用！**存在手动删除数据的情况**！

8、paypalState 的枚举值有哪些？分别代表什么以及状态？

      无用！

9、referralCode 推荐码的场景是什么？是否是敏感字段？

      下单推荐码，用于标识是谁推荐的订单，给与奖励或提成！不敏感！

![](https://confluence.autel.com/download/attachments/50932669/image2022-2-14_20-51-24.png?version=1&modificationDate=1646365089000&api=v2)

该表其余字段均需要解释下！

**orderOriginalMoney 订单原价**

**orderCurrencyMoney 订单当前价**

**orderCurrencyMoney * rate = feeMoney**

**orderCurrencyMoney + feeMoney = orderMoney**

```

```

dt_orderminsaleunitdetail
-------------------------

订单明细表

```

```

### 问题

1、dt_orderinfo 订单表销售的是什么？设备？软件？如何区分销售设备的订单和续费软件的订单？

     order 表示的是产品软件续费订单！

2、dt_orderminsaleunitdetail 中的 productSerialNo 产品序列号表示什么业务含义，也就是说产品序列号表示的是该订单销售的是该设备吗？

      产品软件续费

3、minSaleUnitCode 最小销售单位的业务含义是什么？其与 dt\_minsaleunit、dt\_minsaleunitmemo、dt\_minsaleunitprice、dt\_minsaleunitsalecfgdetail、dt\_minsaleunitsoftwaredetail、dt\_saleconfig 有什么关系？

     将软件打包作为销售单位

4、consumeType 的枚举值 0 和 1 分别表示什么？

     0-开通    1- 续费 

5、同一个订单的 minSaleUnitPrice 相加的和为 dt_orderinfo 的 orderMoney ? 有些数据不满足这样的条件！   

      ![](https://confluence.autel.com/download/attachments/50932669/image2022-2-15_14-12-41.png?version=1&modificationDate=1646365089000&api=v2)

      不相符的且数值小的是测试数据！

6、year 字段“购买年限”表示的业务含义是什么？

      续费多久！

7、isSoft 字段表示的业务含义是什么？0 和 1 分别表示什么？   

     minSaleunitcode 是以 cfg 开头的是销售配置！msu开头的最小销售单位！

     1- 最小销售单位  0-销售配置

```

```

dt_salecontract
---------------

销售契约表

```

```

### 问题

1、dt_saleconfig 表示什么？

     产品出厂时的属性表

2、有关契约的

     islimit

      欧洲的产品不能卖到美国去！1-限制 

     reChargePrice   定价 、contractDate 无用、expirationTime 无用、upMonth 免费试用月份、warrantyMonth 保质期、isshow  无用、

discountFlag、discountPrice、discountStartDate、discountEndDate （flag=1 表示打折）  的业务含义

业务中没有使用

dt_productforsealer
-------------------

产品机型配置表

```

```

### 问题

1、renewModel 枚举值 1 和 2 分别表示什么？

     1-整机续费     2-单个软件购买

2、type 枚举值 0 和 1 分别表示什么？

      没用！

3、freeVinNum、reward、proSeries 是否有用？如果有用其业务含义是什么？

     暂时没用！

dt_areaconfig
-------------

区域配置表，通过code与dt_salecontract表关联。

```

```

dt_productsoftwarevalidstatus
-----------------------------

产品软件有效期状态表  

```

```

![](https://confluence.autel.com/download/attachments/50932669/image2022-2-15_18-20-52.png?version=1&modificationDate=1646365089000&api=v2)

ods→dwd

1) 同步方式：每天全量同步

2) 过滤有效期为空字符串和 null 的脏数据；

    将 validDate 统一日期格式 yyyy-MM-dd

        a、2021-7-30 转换为 2021-07-30

        b、85353-02-21 转换为 9999-12-31

dwd→dwb

1) 取同一设备的最小过期日期作为这个设备的过期日期；

2) valid_date 与 当前日期比较，如果大于当前日期则未过期，如果小于则过期；

dt_productsoftwarevalidchangelog
--------------------------------

产品软件有效期变更日志表  
select min(id), max(id), count(1) from dt_productsoftwarevalidchangelog;

5       860704     854714

日志表的数据量比事实表少了一个量级！！

该表可以获取续费方式、续费时间、续费多久！

select count(1) from dt_productsoftwarevalidchangelog where productSN is null;  – 4315

select count(1) from dt_productsoftwarevalidchangelog where productSN not regexp '(\[0-9\]|\[A-Z\]){2,}'; – 1449

productSN=1 的数量为 1449

select * from dt_productsoftwarevalidchangelog where productSN='AEM171101622';

![](https://confluence.autel.com/download/attachments/50932669/image2022-2-16_17-35-39.png?version=1&modificationDate=1646365089000&api=v2)

ods→dwd

1) 同步方式：每天全量同步

2）productSN 必须满足 '(\[0-9\]|\[A-Z\]){2,}' 正则表达式

     oldDate 和 newDate 虽然有不规范的数据，但是根据使用场景（计算续费多长时间），不能转换！

dwd→dwb

1) 存在一个设备同一时间的多条续费日志，在计算续费次数时算一次！

tb_ke30

![](https://confluence.autel.com/download/attachments/50932669/image2022-2-18_14-7-35.png?version=1&modificationDate=1646365089000&api=v2)

诊断销售套件、诊断产成品-升级卡、包材-升级卡、虚拟升级卡属于升级卡销售

软件升级收费里面的paypalnet、google和authorize 三个经销商剔除掉属于后台手动升级

虚拟升级卡      9010402

软件升级收费    9010401

诊断产成品-升级卡      1001201

包材-升级卡     5001801

Paypal - Internet

0000200107  

Google LLC  

0000201197  

Authorize.net  

0000201204

![](https://confluence.autel.com/download/thumbnails/50932669/image2022-2-19_14-21-9.png?version=1&modificationDate=1646365089000&api=v2)

  
1、order_no 存在重复，目前来看是有效数据，其对应的物料或物料组是不一样的；  
   order_no 存在为空字符串的数据，确认是否为脏数据？  
2、![](https://confluence.autel.com/download/thumbnails/50932669/image2022-2-19_16-45-33.png?version=1&modificationDate=1646365089000&api=v2)
  销售组织存在为空字符串的数据；  
3、book_date 数据质量高，没有异常数据  
4、

tb_dealer 表数据调研

select count(1) from tb_dealer; \-\- 1492

\-\- 除了省份，其他均不为空  
select count(1) from tb_dealer where district!='';

select sale_area,count(1) from tb_dealer group by sale_area;

![](https://confluence.autel.com/download/attachments/50932669/image2022-2-28_18-4-40.png?version=1&modificationDate=1646365089000&api=v2)

select company_code, company_name, count(1) from tb_dealer group by company_code, company_name;

![](https://confluence.autel.com/download/attachments/50932669/image2022-2-28_18-6-5.png?version=1&modificationDate=1646365089000&api=v2)

select dealer_id, dealer_name, count(1) from tb_dealer group by dealer_id, dealer_name having count(1)>1;

select \* from tb_dealer where dealer_id='0000100143';

![](https://confluence.autel.com/download/attachments/50932669/image2022-2-28_18-9-26.png?version=1&modificationDate=1646365089000&api=v2)

存在一个经销商对应多个销售区域的场景，需要与 company_code 一起关联！

应续设备数方案调研
---------

一个设备从注册日（reg\_date）开始变产生有效期，一般都会有一段免费试用期（up\_month），该有效期会记录在dt\_productsoftwarevalidstatus 中，而dt\_productsoftwarevalidchangelog中没有日志记录。当通过续费操作变更有效期时，会新增dt\_productsoftwarevalidchangelog 日志记录，同时更新dt\_productsoftwarevalidstatus 中的有效期（valid_date）。

问题1：是否计算**目前**已解绑（reg_status=2）的设备的有效期？

解绑日期在理论有效期之前，那么在解绑日期之前的截止有效期为理论有效期，在解绑日期之后的截止有效期是理论有效期还是解绑日期？

解绑日期在理论有效期之后，那么通过 reg\_date + up\_month 来计算免费试用期的截止有效期；

答复：过滤已解绑的设备

问题2：未注册（reg_status=0）设备没有设备有效期，需要过滤。

问题3：一个设备对应多个有效期时，是否以最小日期最为设备有效期？

答复：以最小日期为准

问题4：当 valid_date 为当天时，那么对于当天该设备是否过期，或者说是否为应续费？

答复：未过期，超过当天才算过期

问题5：当判断历史有效期时，operatorTime处于当天时，old\_date是有效期还是new\_date是有效期？

答复：new_date 作为有效期

问题6：历史数据以最早日期同步的数据生成，仅生成一次。

问题7：历史数据的日期边界

答复：2020-01-01

问题8：为何要筛选69种可续费设备？产品网站的订单都是续费订单，也就是说产品网站的dt\_orderinfo 和dt\_orderminsaleunitdetail 中的设备都是可续费设备？因为都有续费订单产生，显然是可续费设备！问题是关联出来目前存在为NULL 的产品线和产品系列？

答复：由于产品线和产品系列是人为定义的维度，某些设备类型目前还没有归属，所以暂时只定义了69种，如果按照之前产品提供的条件，会将产品线和产品系列为NULL的过滤掉，这样会影响统计结果的准确性，导致丢失部分续费收入，所以目前与产品达成一致，将未关联到的产品系列和产品线置为“其他”。

问题9：产品确认要用目前的数据通过变更日志表回溯出有效期表的历史快照，目前调研出日志表存在变更记录缺失的数据，如何处理这种数据？

![](https://confluence.autel.com/download/attachments/50932669/image2022-3-4_10-9-38.png?version=1&modificationDate=1646365089000&api=v2)