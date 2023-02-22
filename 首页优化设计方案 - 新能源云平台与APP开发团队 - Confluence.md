# 首页优化设计方案 - 新能源云平台与APP开发团队 - Confluence
1.1  目的
-------

smart-bi以及首页flink以及spark程序优化重构规划

1.2  背景
-------

_由于历史遗留的问题、以及1.6版本迭代，需要对smart-bi以及支持数据业务的flink程序以及spark程序进行优化调整。_

1.3  范围
-------

_1.3.1 smart-bi服务存在较多跨库查询Sql_

_1.3.2 收入、客户数指标统计逻辑修改_

_1.3.3 枪利用率、滞留时长指标统计涉及的源数据需要增加对桩在线、离线的监控_

1、smart-bi获取普通用户的指标时，通过调用外部API/user/getLocationIds获取场站列表，再通过场站列表获取指标，调用Api：[http://autel-cloud-energy-gateway-enedev.auteltech.cn/swagger-ui/index.html?urls.primaryName=pile-user-app#/](http://autel-cloud-energy-gateway-enedev.auteltech.cn/swagger-ui/index.html?urls.primaryName=pile-user-app#/)

2、收入指标统计逻辑变更=》收入=支付金额-手续费；客户数月粒指标统计逻辑修改：原有活跃用户数、新用户数统计，修改位新用户数、老用户数统计，其中当月老用户=当月用户-当月新用户

3、枪利用率原数据来源：kafka--evse\_status\_topic，新增pile\_status\_online\_topic、pile\_status\_offline\_topic两个topic到工作流中以判断桩的在线、离线；另外枪使用中状态为（Charging/Perparing）,滞留状态为Finishing

### 3.1.1 数据存储设计

#### 3.1.1.1   数据库设计

首页指标：新增手续费、实际收入指标

\-\- 商户维度首页指标统计

CREATE TABLE \`ads\_rt\_homepage\_operator\_stats_dm\` (  
\`period_time\` date NOT NULL COMMENT '周期',  
\`operator_id\` bigint(20) NOT NULL COMMENT '商家Id',  
\`order_count\` bigint(20) DEFAULT NULL COMMENT '订单数',  
\`order\_count\_lpt\` bigint(20) DEFAULT NULL COMMENT '上一周期订单数',  
\`order\_count\_pct\` decimal(20,4) DEFAULT NULL COMMENT '订单数-环比%',  
\`member_count\` bigint(20) DEFAULT NULL COMMENT '充电用户数',  
\`member\_count\_lpt\` bigint(20) DEFAULT NULL COMMENT '上一周期充电用户数',  
\`member\_count\_pct\` decimal(20,4) DEFAULT NULL COMMENT '充电用户数-环比%',  
\`payment_amount\` decimal(20,4) DEFAULT NULL COMMENT '订单收入',  
\`payment\_amount\_lpt\` decimal(20,4) DEFAULT NULL COMMENT '上一周期订单收入',  
\`payment\_amount\_pct\` decimal(20,4) DEFAULT NULL COMMENT '订单收入-环比%',  
\`handling_fee\` decimal(20,4) DEFAULT NULL COMMENT '手续费',  
\`handling\_fee\_lpt\` decimal(20,4) DEFAULT NULL COMMENT '上一周期手续费',  
\`handling\_fee\_pct\` decimal(20,4) DEFAULT NULL COMMENT '手续费-环比%',  
\`auctal_income\` decimal(20,4) DEFAULT NULL COMMENT '实际收入',  
\`auctal\_income\_lpt\` decimal(20,4) DEFAULT NULL COMMENT '上一周期实际收入',  
\`auctal\_income\_pct\` decimal(20,4) DEFAULT NULL COMMENT '实际收入-环比%',  
\`charge_duration\` bigint(20) DEFAULT NULL COMMENT '充电时长:ms',  
\`charge\_duration\_lpt\` bigint(20) DEFAULT NULL COMMENT '上一周期充电时长:ms',  
\`charge\_duration\_pct\` decimal(20,4) DEFAULT NULL COMMENT '充电时长-环比%',  
\`charge_energy\` decimal(20,4) DEFAULT NULL COMMENT '充电能耗:wh',  
\`charge\_energy\_lpt\` decimal(20,4) DEFAULT NULL COMMENT '上一周期充电能耗:wh',  
\`charge\_energy\_pct\` decimal(20,4) DEFAULT NULL COMMENT '充电能耗-环比%',  
PRIMARY KEY (\`period\_time\`,\`operator\_id\`)  
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COMMENT='运营首页-商家维度统计';

\-\- 场站维度首页指标统计

CREATE TABLE \`ads\_rt\_homepage\_location\_stats_dm\` (  
\`period_time\` date NOT NULL COMMENT '周期',  
\`operator_id\` bigint(20) NOT NULL COMMENT '商家Id',  
\`location_id\` bigint(20) NOT NULL COMMENT '场站Id',  
\`location_name\` varchar(1000) DEFAULT NULL COMMENT '场站名称',  
\`order_count\` bigint(20) DEFAULT NULL COMMENT '订单数',  
\`order\_count\_lpt\` bigint(20) DEFAULT NULL COMMENT '上一周期订单数',  
\`order\_count\_pct\` decimal(20,4) DEFAULT NULL COMMENT '订单数-环比%',  
\`member_count\` bigint(20) DEFAULT NULL COMMENT '充电用户数',  
\`member\_count\_lpt\` bigint(20) DEFAULT NULL COMMENT '上一周期充电用户数',  
\`member\_count\_pct\` decimal(20,4) DEFAULT NULL COMMENT '充电用户数-环比%',  
\`payment_amount\` decimal(20,4) DEFAULT NULL COMMENT '订单收入',  
\`payment\_amount\_lpt\` decimal(20,4) DEFAULT NULL COMMENT '上一周期订单收入',  
\`payment\_amount\_pct\` decimal(20,4) DEFAULT NULL COMMENT '订单收入-环比%',  
\`handling_fee\` decimal(20,4) DEFAULT NULL COMMENT '手续费',  
\`handling\_fee\_lpt\` decimal(20,4) DEFAULT NULL COMMENT '上一周期手续费',  
\`handling\_fee\_pct\` decimal(20,4) DEFAULT NULL COMMENT '手续费-环比%',  
\`auctal_income\` decimal(20,4) DEFAULT NULL COMMENT '实际收入',  
\`auctal\_income\_lpt\` decimal(20,4) DEFAULT NULL COMMENT '上一周期实际收入',  
\`auctal\_income\_pct\` decimal(20,4) DEFAULT NULL COMMENT '实际收入-环比%',  
\`charge_duration\` bigint(20) DEFAULT NULL COMMENT '充电时长:ms',  
\`charge\_duration\_lpt\` bigint(20) DEFAULT NULL COMMENT '上一周期充电时长:ms',  
\`charge\_duration\_pct\` decimal(20,4) DEFAULT NULL COMMENT '充电时长-环比%',  
\`charge_energy\` decimal(20,4) DEFAULT NULL COMMENT '充电能耗:wh',  
\`charge\_energy\_lpt\` decimal(20,4) DEFAULT NULL COMMENT '上一周期充电能耗:wh',  
\`charge\_energy\_pct\` decimal(20,4) DEFAULT NULL COMMENT '充电能耗-环比%',  
PRIMARY KEY (\`period\_time\`,\`operator\_id\`,\`location_id\`)  
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COMMENT='运营首页-场站维度统计';

原ads\_rt\_home\_page\_evse\_duration\_stats_sa表丢弃、数据存储到S3（映射到Hive）

新增ads\_rt\_homepage\_operator\_idle_dm、ads\_rt\_homepage\_location\_idle_dm分别统计商户、场站维度的滞留时长、滞留次数

\-\- 统计商户维度的滞留时长

CREATE TABLE \`ads\_rt\_homepage\_operator\_idle_dm\` (  
\`period_time\` date NOT null comment '统计周期',  
\`operator_id\` bigint(20) NOT NULL COMMENT '商家id',  
\`idle_duration\` bigint(20) DEFAULT NULL COMMENT '滞留时长',  
\`idle_count\` int DEFAULT NULL COMMENT '滞留次数',  
PRIMARY KEY (period\_time, \`operator\_id\`)  
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COMMENT='首页-商户滞留时长统计';

\-\- 统计场站维度的滞留时长

CREATE TABLE \`ads\_rt\_homepage\_location\_idle_dm\` (  
\`period_time\` date NOT null comment '统计周期',  
\`location_id\` bigint(20) NOT NULL COMMENT '场站id',  
\`idle_duration\` bigint(20) DEFAULT NULL COMMENT '滞留时长',  
\`idle_count\` int DEFAULT NULL COMMENT '滞留次数',  
PRIMARY KEY (period\_time, \`location\_id\`)  
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COMMENT='首页-场站滞留时长统计';

原ads\_rt\_home\_page\_evse\_status\_stats_xa表丢弃、进行数据分表

新增ads\_rt\_homepage\_operator\_using\_dwin、ads\_rt\_homepage\_location\_using\_dwin分别统计商户、场站维度的枪利用率

数据ads\_rt\_homepage\_operator\_using\_dwin、ads\_rt\_homepage\_location\_using\_dwin每日定期上传Hive并清理数据、数据库只保留最近10天记录

\-\- 统计商户维度的枪利用率

CREATE TABLE ads\_rt\_homepage\_operator\_using_dwin (  
window\_start\_time bigint not null comment '窗口开始时间',  
window\_end\_time bigint not null comment '窗口结束时间',  
operator_id bigint not null comment '商家id',  
evse_count int default null comment '商家充电枪总数',  
evse\_count\_using int default null comment '使用中的枪数量',  
PRIMARY KEY (window\_start\_time,window\_end\_time, \`operator_id\`)  
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COMMENT='首页-商户枪利用率统计';

\-\- 统计场站维度的枪利用率

CREATE TABLE ads\_rt\_homepage\_location\_using_dwin (  
window\_start\_time bigint not null comment '窗口开始时间',  
window\_end\_time bigint not null comment '窗口结束时间',  
location_id bigint not null comment '场站id',  
evse_count int default null comment '场站充电枪总数',  
evse\_count\_using int default null comment '使用中的枪数量',  
PRIMARY KEY (window\_start\_time,window\_end\_time, \`location_id\`)  
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COMMENT='首页-场站枪利用率统计';

\-\- 新用户标签

CREATE TABLE \`dtm\_operation\_stats\_user\_state_xa\` (  
\`operator_id\` bigint(20) NOT NULL COMMENT '商户id',  
\`member_id\` bigint(20) NOT NULL COMMENT '用户id',  
\`first\_charge\_member\` int(11) DEFAULT '0' COMMENT '首充用户: 0是,1不是',  
\`first\_charge\_time\` datetime DEFAULT '1970-01-01 00:00:00' COMMENT '首充时间',  
\`first\_charge\_location_id\` bigint(20) DEFAULT NULL COMMENT '首充场站',  
\`first\_charge\_pile\_evse\_sn\` varchar(100) DEFAULT NULL COMMENT '首充桩',  
\`first\_charge\_evse_sn\` varchar(100) DEFAULT NULL COMMENT '首充设备',  
\`first\_paid\_member\` int(11) DEFAULT '0' COMMENT '首付用户: 0是,1不是',  
\`first\_paid\_time\` datetime DEFAULT '1970-01-01 00:00:00' COMMENT '首付时间',  
\`first\_paid\_location_id\` bigint(20) DEFAULT NULL COMMENT '首付场站',  
\`first\_paid\_pile\_evse\_sn\` varchar(100) DEFAULT NULL COMMENT '首付桩',  
\`first\_paid\_evse_sn\` varchar(100) DEFAULT NULL COMMENT '首付设备',  
PRIMARY KEY (\`operator\_id\`,\`member\_id\`),  
KEY \`idx1\_operation\_stats\_user\_state\_xa\` (\`first\_charge\_location\_id\`)  
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COMMENT='新用户标签表';