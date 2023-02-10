# EMR升级文档指南 - 新能源云平台与APP开发团队 - Confluence
1、确定升级环境，找运维申请开通账号权限
--------------------

1.1 EMR控制台权限

1.2 S3的读写权限

1.3 账号下集群的jump的root权限

2、创建新集群
-------

2.1 创建集群的过程参照[大数据测试环境搭建指南](https://confluence.autel.com/pages/viewpage.action?pageId=69304366)这个文档建设，具体需要看情况定；

2.2 EMR版本计划升级为6.8.0，集群大小为1+2+2的配置；（机型统一m5机型）

3、开通新集群的junm权限及端口
-----------------

![](https://confluence.autel.com/download/attachments/158990565/image2022-11-18_11-28-28.png?version=1&modificationDate=1668742109000&api=v2)

除以上端口外，还需要开通DS的12345端口，开通新集群的jump权限；

4、修改新集群的配置并安装ds
---------------

4.1 上传需要用到的mysql连接mysql-connector-java-5.1.47.jar的jar和mysql-connector-java-8.0.29.jar并且将mysql-connector-java-5.1.47.jar的复制到sqoop的lib目录下；

4.2 安装的DS版本为新能源数字维修的2.0.3，具体的安装过程参考[https://dolphinscheduler.apache.org/zh-cn/docs/2.0.3/user_doc/guide/installation/pseudo-cluster.html](https://dolphinscheduler.apache.org/zh-cn/docs/2.0.3/user_doc/guide/installation/pseudo-cluster.html)

4.3 配置DS的微信机器人告警，具体参考文档[DolphinScheduler 报警配置](https://confluence.autel.com/pages/viewpage.action?pageId=47886292)

4.4 配置flink数据无法写入S3的问题；

4.4.1 在flink的安装目录下；cd /lib/flink 执行

mkdir -p ./plugins/s3-fs-hadoop

cp ./opt/flink-s3-fs-hadoop-1.xx-SNAPSHOT.jar ./plugins/s3-fs-hadoop/

4.4.2 进入flink-conf.yaml里面

在文件的最后面追加写：（不同环境不一样国内的如下图；国外的s3.endpoint:后面没有.cn）

![](https://confluence.autel.com/download/attachments/158990565/image2022-11-18_11-48-42.png?version=1&modificationDate=1668743322000&api=v2)

4.5 更改hue的界面无法看到S3的问题；（不同环境不一样国内的如下图；国外的host:后面没有.cn）

![](https://confluence.autel.com/download/attachments/158990565/image2022-11-18_11-50-13.png?version=1&modificationDate=1668743414000&api=v2)

5、测试通过后关闭旧集群
------------