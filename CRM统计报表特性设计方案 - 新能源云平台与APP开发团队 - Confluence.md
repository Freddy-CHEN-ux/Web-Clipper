# CRM统计报表特性设计方案 - 新能源云平台与APP开发团队 - Confluence
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

##### 参数示例

```

```

##### 3.1.2.4.1 响应数据

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

##### 参数示例

```

```

##### 3.1.2.4.2 响应数据

##### 响应示例

```

```

### 3.1.2 业务流程

_业务流程图、时序图_

### 3.1.3 数据存储设计

#### 3.1.3.1   数据库设计

表1字段类型：

tableName ——  energy\_smart\_bi.ads\_operation\_crm_monthly

表1

_ ![](https://confluence.autel.com/download/attachments/102269590/image2022-8-16_17-27-48.png?version=1&modificationDate=1660642069000&api=v2)
_

_表2字段 类型：_

tableName ——  energy\_smart\_bi.ads\_operation\_crm\_charging\_distri

_表2_

_![](https://confluence.autel.com/download/attachments/102269590/image2022-8-16_17-25-59.png?version=1&modificationDate=1660641960000&api=v2)
_

#### 3.1.3.2   缓存设计

_数据结构、缓存策略、同步机制…_

#### 3.1.3.3   搜索引擎设计

_数据结构、同步机制、索引策略…_

### 3.1.4 业务流程

![](https://confluence.autel.com/download/attachments/102269590/image2022-8-16_21-3-36.png?version=1&modificationDate=1660655017000&api=v2)

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