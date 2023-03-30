# CRM统计报表特性设计方案 - 数据中心项目 - Confluence
1.1  目的
-------

该文档作为CRM客户服务管理功能的一部分，描述了统计报表的的数据交互、存储等功能设计放哪。

1.2  背景
-------

产品补充

1.3  范围
-------

 云平台设计及开发人员

[![](https://jira.autel.com/secure/viewavatar?size=xsmall&avatarId=10315&avatarType=issuetype)
ESAAS-13765](https://jira.autel.com/browse/ESAAS-13765) \- 【客户管理】客户分析 完成 

3.1  Task
---------

### 3.1.1  功能介绍

【人均支付金额】  
最近6个月内每个月的人均支付金额  
默认显示全部客户的最近6个月内每个月的人均支付金额；

【人均消费频次】  
最近6个月内每个月的人均消费次数  
默认显示全部客户的最近6个月内每个月的人均消费次数；

【营业额】  
最近6个月内每个月商家的实收总金额和折扣优惠总金额  
默认显示全部客户的最近6个月内商家的实收总金额和折扣优惠总金额；

【最近30天充电时段分布】  
最近30天内 0.5*24 每个半点的累计充电次数  
默认显示全部客户的最近30天时段分布累计充电次数；

### 3.1.2 接口定义

##### 3.1.2.1.1  接口名称

| 

URL地址

 | /smart-bi/crm/monthlyIndex |
| 

请求方法

 | post |

##### 3.1.2.3.1 Body/Query参数

| 

 字段

 | 

类型

 | 

描述

 | 

备注

 |
| --- | --- | --- | --- |
| operatorId | long | 商家id |   
 |
| groupIds | list集合 | 分组id | 集合里面字段long类型 |

##### 参数示例

```

```

##### 3.1.2.4.1 响应数据

| 

 字段

 | 

类型

 | 

描述

 | 

备注

 |
| --- | --- | --- | --- |
| groupId | bigint（20） | 分组id |   
 |
| groupName | varchar（255） | 分组名称 |   
 |
| elements | list集合 | 对象集合 |   
 |
| dt | varchar（255） | 月份标识 | yyyy-MM |
| perPersonPaid | decimal(10,2) | 人均支付金额 |   
 |
| perPersonTime | decimal(10,2) | 人均支付次数 |   
 |
| monthlyMoneyReceived | decimal(10,2) | 每月总收入额 |   
 |
| monthlyMoneyCoupon | decimal(10,2) | 每月总优惠金额 |   
 |

##### 响应示例

```

```

##### 3.1.2.1.2 接口名称

| 

URL地址

 | /smart-bi/crm/chargingDistribute |
| 

请求方法

 | post |

##### 3.1.2.3.2 Body/Query参数

| 

 字段

 | 

类型

 | 

描述

 | 

备注

 |
| --- | --- | --- | --- |
| operatorId | long | 商家id |   
 |
| groupIds | list集合 | 分组id | 集合里面字段long类型 |

##### 参数示例

```

```

##### 3.1.2.4.2 响应数据

| 

 字段

 | 

类型

 | 

描述

 | 

备注

 |
| --- | --- | --- | --- |
| groupId | bigint（20） | 分组id |   
 |
| groupName | varchar（255） | 分组名称 |   
 |
| elements | list集合 | 对象集合 |   
 |
| chargeTime | bigint（20） | 充电时段 | 1-48(每半小时加1) |
| chargeTotalTime | bigint（20） | 充电总次数 |   
 |
|   
 |   
 |   
 |   
 |

##### 响应示例

```

```

### 3.1.2 业务流程

_业务流程图、时序图_

### 3.1.3 数据存储设计

#### 3.1.3.1   数据库设计

表1字段类型：

tableName ——  energy\_smart\_bi.ads\_operation\_crm_monthly

| 

字段

 | 

类型

 | 

含义

 | 

其他

 |
| --- | --- | --- | --- |
| operation_id | bigint（20） | 商家id | 联合主键 |
| group_id | bigint（20） | 分组id（9999、分组用户） | 联合主键 |
| group_name | varchar（255） | 分组名称：（all，分组name） |   
 |
| per\_person\_paid | decimal(10,2) | 人均支付金额 |   
 |
| per\_person\_time | decimal(10,2) | 人均消费次数 |   
 |
| monthly\_money\_received | decimal(10,2) | 月实收营业总额 |   
 |
| monthly\_money\_coupon | decimal(10,2) | 月优惠总额 |   
 |
| dt | varchar（255） | 时间（yyyy-MM） | 联合主键 |

表1

_ ![](https://confluence.autel.com/download/attachments/215615366/image2022-8-16_17-27-48.png?version=1&modificationDate=1677069621000&api=v2)
_

_表2字段 类型：_

tableName ——  energy\_smart\_bi.ads\_operation\_crm\_charging\_distri

| 

字段

 | 

类型

 | 

含义

 | 

其他

 |
| --- | --- | --- | --- |
| operation_id | bigint（20） | 商家id | 联合主键 |
| group_id | bigint（20） | 分组id（9999、分组用户） | 联合主键 |
| group_name | varchar（255） | 分组名称：（all，分组name） |   
 |
|  charge_time | bigint（20） | 充电时段(1-48) |   
 |
| charge\_total\_time | bigint（20） | 充电总次数 |   
 |
| dt | varchar（255） | 时间（yyyy-MM-dd） | 联合主键 |

_表2_

_![](https://confluence.autel.com/download/attachments/215615366/image2022-8-16_17-25-59.png?version=1&modificationDate=1677069621000&api=v2)
_

#### 3.1.3.2   缓存设计

_数据结构、缓存策略、同步机制…_

#### 3.1.3.3   搜索引擎设计

_数据结构、同步机制、索引策略…_

### 3.1.4 业务流程

![](https://confluence.autel.com/download/attachments/215615366/image2022-8-16_21-3-36.png?version=1&modificationDate=1677069621000&api=v2)

### 3.1.5 安全和可靠性设计

#### 3.1.5.1   异常处理流程

_NA_

#### 3.1.5.2  补偿机制

_NA_

#### 3.1.5.3  权限控制

_接口访问权限、数据访问权限_

#### 3.1.5.4  错误码定义

_接口错误码定义_

### 3.1.6 兼容性设计

_APP、桩、Web、其他微服务的兼容性设计_

#### 3.1.6.1  风险评估

_影响范围评估，如影响支付、充电鉴权、推送等_

_其他参考或者有关联的文档链接_