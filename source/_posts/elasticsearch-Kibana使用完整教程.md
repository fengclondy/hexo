---
layout: post
title: elasticsearch--Kibana使用完整版教程
date: 2018-06-11 06:15:41
tags: elasticsearch
categories: elasticsearch
---

参考：[官网](https://www.elastic.co/guide/cn/kibana/current/deb.html)

### 目录结构

|类型 |	描述 |	默认位置|
| --  |:------:| -----:|
|home | Kibana home 目录或 $KIBANA_HOME 。 | /usr/share/kibana  |
|bin  | 二进制脚本，包括 kibana 启动 Kibana server 和 kibana-plugin 安装插件。  |/usr/share/kibana/bin  |
|config  | 配置文件，包括 kibana.yml 。  | /etc/kibana  |
|data | Kibana 和其插件写入磁盘的数据文件位置。  | /var/lib/kibana  |
|optimize  | 编译过的源码。某些管理操作 (如，插件安装) 导致运行时重新编译源码。 | /usr/share/kibana/optimi   |
|plugins  | 插件文件位置。每一个插件都有一个单独的二级目录。  | /usr/share/kibana/plugins  |

<!-- more -->

### 配置(常用配置)

```properties
## 默认配置
#server.port: 5601
#server.host: localhost
#elasticsearch.url: "http://localhost:9200"
#server.ssl.enabled: "false" 

elasticsearch.username: "elastic"
elasticsearch.password: "elastic"
```

>https://www.elastic.co/guide/cn/kibana/current/settings.html

>关于docker的操作：参考：https://www.elastic.co/guide/cn/kibana/current/docker.html

### 使用

#### 导入数据源

1、在数据源目录下（accounts.json文件），导入/bank序列下面：

```
curl -H 'Content-Type: application/x-ndjson' -XPOST 'localhost:9200/bank/account/_bulk?pretty'
--data-binary @accounts.json; echo
```

2、导入完成，查看结果：
```
GET /_cat/indices?v
```

#### 定义索引模式

在Kibana的左侧模块 **Management** 里，新建`shakes*`索引模式，匹配所有的以 shakes 开头的索引。
![](http://p2jr3pegk.bkt.clouddn.com/kibana_management.png)

对于包含时间序列数据集，
要确保 **Index contains time-based events** 被选中，并从 **Time-field name** 下拉列表中选择 **@timestamp** 字段。

#### 搜索数据

在 **Discover** 模块里，先选择过滤器 **filter**，然后在搜索栏里输入：
`account_number:<100 AND balance:>47500` 即可在所有的以 ba 开头的索引里搜索想要的结果。

默认会筛选出结果的全部字段，可通过点击字段的 add 按钮来进行定向显示。
![](http://p2jr3pegk.bkt.clouddn.com/kibana_discover01.png)

#### 可视化数据图

##### 饼状图（Pie）

在 Visualize 模块里，先创建一个需要显示的图表类型 Pie，然后选择对应的索引模式。

默认匹配所有数据，所以初始状态无分区。选择elasticsearch的桶聚合，完整数据的显示。

定义每个区间桶：
- 1）点击 **Split Slices** 桶类别。
- 2)从 **Aggregation** 列表中选择 **Range** 。
- 3)从 **Field** 列表中选择 XXX 某字段。
- 4)点击四次 **Add Range** 把区间总数增加到6个。
- 5）输入每个区间的起始位置序号。
- 6）点击 **Apply changes**（运行按钮） 更新图表。

![](http://p2jr3pegk.bkt.clouddn.com/kibana_visualize01.png)

还可以进行自匹配，查看其他属性：

- 1)点击桶列表中的 **Add sub-buckets** 。
- 2)点击桶类型列表中的 **Split Slices** 。
- 3)在聚合列表中选择 **Terms** 。
- 4)在字段列表中选择 YYY 某字段 。
- 5)点击 **Apply changes images/apply-changes-button.png**。

![](http://p2jr3pegk.bkt.clouddn.com/kibana_visualize02.png)


##### 条状图（Vertical bar chart）

在 **Visualize** 模块里，先创建一个需要显示的图表类型 **Bar**，然后选择对应的索引模式。

这里需要用到elasticsear的 指标聚合 配置y轴；在x轴上显示不同的剧目，选择X轴桶种类，然后在聚合列表中选择 **Terms**

![](http://p2jr3pegk.bkt.clouddn.com/kibana_visualize03.png)


##### 可视化地图（Coordinate map）

基本操作：
- 1)点击 **New** 。
- 2)选择 **Coordinate map** 。
- 3)选择 **logstash-*** 索引模式。
- 4)设置我们要查看的事件的时间窗口：
- 5)在  Kibana 工具栏中点击时间控件选择。
- 6)点击 **Absolute** 。
- 7)设置开始时间为 May 18, 2015，结束时间为 May 20, 2015

![](http://p2jr3pegk.bkt.clouddn.com/kibana_visualize04.png)

- 8)选择 **Geo Coordinates** 作为桶，并点击 **Apply changes** 来显示日志文件中对应的地理坐标 

![](http://p2jr3pegk.bkt.clouddn.com/kibana_visualize05.png)


##### 仪表盘数据汇总

- 1）在侧边导航栏点击 **Dashboard** 。
- 2）点击 **Add** 显示已保存的可视化控件列表。
- 3）单击 可选择的图表 ，然后通过点击列表底部向上的小箭头关闭可视化控件列表，查看其它的一些参数。

![](http://p2jr3pegk.bkt.clouddn.com/kibana-dashboard01.png)

其它的一些kibana设置参考官方文档：https://www.elastic.co/guide/cn/kibana/current/advanced-options.html
