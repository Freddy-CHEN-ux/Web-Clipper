# Motor 开发文档 - 大数据平台 - Confluence
[转至元数据结尾](#page-metadata-end)

[转至元数据起始](#page-metadata-start)

此文档是针对如下两个需求的开发文档，开发过程中会不断调整文档内容，相关人员需以最终版为准。

*   [Motor收入分拆-数据集市](https://confluence.autel.com/pages/viewpage.action?pageId=58723881)
*   [续费升级支付-数据集市](https://confluence.autel.com/pages/viewpage.action?pageId=58723775)

需求调研文档： [Motor&PayPal 数据调研](https://confluence.autel.com/pages/viewpage.action?pageId=58725528)

此开发文档主要包括两个方面：

1.  数仓开发
2.  任务调度

总体概览
----

![](https://confluence.autel.com/download/attachments/58727643/image2022-4-28_11-52-20.png?version=2&modificationDate=1659419880000&api=v2)

ODS 层
-----

### ods\_pay.ods\_pay\_tb\_pay\_order\_di

#### DDL

```

```

#### ETL

由于该表存在规范的create\_time和update\_time且存在最终状态，可以增量同步。

```

```

### ods\_motor.ods\_motor\_tb\_motor\_order\_di

#### DDL

```

```

#### ETL

由于该表存在update_time且存在最终状态，可以增量同步。

```

```

### ods\_motor.ods\_motor\_tb\_motor_df

DDL

```

```

ETL

1.  全量同步
2.  autel_id 加密
3.  device_password 脱敏

```

```

### ods\_motor.ods\_motor\_tb\_motor\_bind\_df

DDL

```

```

ETL

全量同步

### ods\_motor.ods\_motor\_tb\_motor\_activity\_df

DDL

```

```

ETL

全量同步

### ods\_pro\_web.ods\_autelproweb\_dt\_paylog\_df

DDL

```

```

ETL

存在物理删除数据的场景，所以全量同步

DWD 层
-----

### dwd\_pay.dwd\_pay\_tb\_pay\_order\_df

DDL

```

```

ETL

1.  增量同步数据更新
2.  存在busi\_order\_id的多条无效数据，去重处理，取时间戳最大的(存在支付后还有支付记录的数据，无法筛选有效数据)  
    不做去重处理； 
3.  过滤 pay_source=test 的脏数据

```

```

### dwd\_motor.dwd\_motor\_tb\_motor\_order\_df

DDL

```

```

ELT

1.  增量同步数据更新
2.  新增 update_date 更新日期字段（后续可作为支付日期使用）

```

```

### dwd\_motor.dwd\_motor\_tb\_motor_df

DDL

```

```

ETL

```

```

### dwd\_motor.dwd\_motor\_tb\_motor\_bind\_df

DDL

```

```

ETL

1.  orderCode 不能为空
2.  status 为已支付

```

```

### dwd\_motor.dwd\_motor\_tb\_motor\_activity\_df

DDL

```

```

ETL

```

```

### dwd\_pro\_web.dwd\_autelproweb\_dt\_paylog\_df

DDL

```

```

ETL

```

```

DWB 层
-----

### dwb\_motor.dwb\_motor\_order\_df

DDL

```

```

ETL

dwd\_motor\_tb\_motor\_order_df  
     1. 筛选已支付的订单 status = 'payed'  
     2\. taxes 税费为空的提供默认值0  
     3\. 筛选 price 不为 0 的订单

  
dwd\_pay\_tb\_pay\_order_df  
    1\. 筛选已支付的订单 status=1  
    2\. 筛选来自 motor 的订单

dwd\_autelproweb\_dt\_orderinfo\_df

    1\. 筛选 app_platform = 'motorCloud'

```

```

### dwb\_oper.dwb\_product\_renew\_online_df

DDL

```

```

ETL

新增 t3 t4 t5 t6

```

```

新增处理 motor 数据的工作流：**motor_order**

**![](https://confluence.autel.com/download/attachments/58727643/image2022-5-5_11-16-45.png?version=1&modificationDate=1651720583000&api=v2)**

修改已有工作流：**product\_renew\_online**

**![](https://confluence.autel.com/download/attachments/58727643/image2022-5-5_11-16-2.png?version=1&modificationDate=1651720540000&api=v2)**