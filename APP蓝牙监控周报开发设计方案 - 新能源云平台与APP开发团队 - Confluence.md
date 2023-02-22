# APP蓝牙监控周报开发设计方案 - 新能源云平台与APP开发团队 - Confluence
1.1  目的
-------

开发反串讲，加强对业务的理解

1.2  背景
-------

APP的蓝牙连接是充电方式的一种，要求监控蓝牙连接情况，并统计出在连接成功、连接失败的前提下对应app版本、固件版本、具体故障描述等做出不同情况下的去重用户数、去重SN数、连接次数的周报，以邮件形式通知相关人员。

1.3  范围
-------

开发、测试、产品端使用

3.1  功能
-------

### 3.1.1 

功能介绍

针对app在神策埋点的数据，进行数据处理，形成报表，邮件通知相关人员。

### 3.1.2 业务流程

_![](https://confluence.autel.com/download/attachments/102957952/image2022-8-17_20-31-0.png?version=1&modificationDate=1660739461000&api=v2)
_

### 3.1.3 数据存储设计

#### 3.1.3.1   数据库设计

**ODS****层**

APP蓝牙监控的数据来自于神策，通过Python导出到本地进行数据处理。由于神策数据中failure_reason字段为json格式，其中的部分字段又非常重要，因此使用Spark进行解析，将原本的json字段拆分成多列。处理后的数据通过上传s3并在ods层建表存放，表名为：biods.ods\_app\_bluetooth_statistic

![](https://confluence.autel.com/download/thumbnails/102957952/image2022-8-17_20-32-35.png?version=1&modificationDate=1660739556000&api=v2)

**DIM****层**

开发中需要剔除测试数据，包括测试人员uid以及测试桩。

其中包含测试人员uid的表为：test\_biods.ods\_dtm\_prod\_uid\_blacklist 包含测试桩的表为：test\_biods.ods\_charge\_pile

同时要求所有监控的桩SN都需要保证在energy\_device.charge\_pile的产品网站出货清单内，任何不在出货清单内的SN产生的订单、商桩和家桩绑定记录都应该剔除掉，将此表也放在dim层。

创建异常数据标签表，包含异常编号、异常名称和异常描述

![](https://confluence.autel.com/download/attachments/102957952/image2022-8-17_20-36-26.png?version=1&modificationDate=1660739787000&api=v2)

**DWS****层**

本层为聚合表，将4个主题的数据进行聚合处理，为以后扩展考虑建立了以日为统计周期的4张表：

bicoredata.dws\_bt\_connection_day

![](https://confluence.autel.com/download/attachments/102957952/image2022-8-17_20-36-52.png?version=1&modificationDate=1660739813000&api=v2)

bicoredata.dws\_bt\_connection\_app\_day

![](https://confluence.autel.com/download/attachments/102957952/image2022-8-17_20-37-1.png?version=1&modificationDate=1660739822000&api=v2)

bicoredata.dws\_bt\_connection\_firmware\_day

![](https://confluence.autel.com/download/attachments/102957952/image2022-8-17_20-37-12.png?version=1&modificationDate=1660739833000&api=v2)

bicoredata.dws\_bt\_connection\_exception\_day

![](https://confluence.autel.com/download/attachments/102957952/image2022-8-17_20-37-21.png?version=1&modificationDate=1660739841000&api=v2)

按周为统计周期进行聚合后也得到了4张表

bicoredata.dws\_bt\_connection_week

![](https://confluence.autel.com/download/attachments/102957952/image2022-8-17_20-37-32.png?version=1&modificationDate=1660739853000&api=v2)

bicoredata.dws\_bt\_connection\_app\_week

![](https://confluence.autel.com/download/attachments/102957952/image2022-8-17_20-37-41.png?version=1&modificationDate=1660739861000&api=v2)

bicoredata.dws\_bt\_connection\_firmware\_week

![](https://confluence.autel.com/download/attachments/102957952/image2022-8-17_20-37-59.png?version=1&modificationDate=1660739880000&api=v2)

bicoredata.dws\_bt\_connection\_exception\_week

![](https://confluence.autel.com/download/attachments/102957952/image2022-8-17_20-37-51.png?version=1&modificationDate=1660739871000&api=v2)

**DWT****层**

本层为聚合表，由于蓝牙连接质量每周汇总表需要计算环比，因此在本层计算了去重用户数、去重SN数和连接次数的周环比，建立了表：

bicoredata.dwt\_bt\_connection\_week\_pct

![](https://confluence.autel.com/download/attachments/102957952/image2022-8-17_20-38-11.png?version=1&modificationDate=1660739892000&api=v2)

**ADS****层**

本层为结果表，按照需求文档的要求建立了四张表

蓝牙连接质量每周汇总表：biads.ads\_bt\_connection\_week\_pct

![](https://confluence.autel.com/download/attachments/102957952/image2022-8-17_20-32-47.png?version=1&modificationDate=1660739567000&api=v2)

APP版本蓝牙连接质量每周汇总表：biads.ads\_bt\_connection\_app\_week

![](https://confluence.autel.com/download/attachments/102957952/image2022-8-17_20-32-54.png?version=1&modificationDate=1660739575000&api=v2)

固件版本蓝牙连接质量每周汇总表：biads.ads\_bt\_connection\_firmware\_week

![](https://confluence.autel.com/download/attachments/102957952/image2022-8-17_20-32-59.png?version=1&modificationDate=1660739579000&api=v2)

蓝牙异常问题类型每周汇总表：biads.ads\_bt\_connection\_exception\_week

![](https://confluence.autel.com/download/attachments/102957952/image2022-8-17_20-33-6.png?version=1&modificationDate=1660739587000&api=v2)

**MySQL**

通过Sqoop将ads层结果导入MySQL

#### 3.1.3.2   缓存设计

_数据结构、缓存策略、同步机制…_

#### 3.1.3.3   搜索引擎设计

_数据结构、同步机制、索引策略…_

### 3.1.4 安全和可靠性设计

#### 3.1.5.1   异常处理流程

_NA_

#### 3.1.5.2  补偿机制

_NA_

#### 3.1.6 兼容性设计

_APP、桩、Web、其他微服务的兼容性设计_

#### 3.1.6.1  风险评估

_数据缺失可能是当前最大的问题，APP1.6.1版本以后数据量逐渐增多数据准确性将提高。_

_其他参考或者有关联的文档链接_