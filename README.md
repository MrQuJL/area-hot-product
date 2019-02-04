
# 各区域热门商品

## （一）模块介绍

电商网站运营中，需要对每个区域用户关心的商品进行统计分析，支持用户决策。

用途：

* 分析各区域对产品的不同需求，进行差异化研究，例如北京用户喜欢手机，上海用户喜欢汽车。

* 指导商品折扣，推广策略。

## （二）需求分析

1. 如何定义热门商品？

	* 简单模型：通过用户对商品的点击量来衡量商品热度。
	
	* 复杂模型：通过用户点击 ＋ 购买以及收藏等综合数据对商品进行评价,商品热门程度得分模型 ＝ 点击次数 * 2 + 购买次数 * 5 + 收藏次数 * 3, 其中2，5，3为得分权重。

2. 如何获取区域？

	* 通过用户点击日志，获取访问IP，进而获取区域信息。
	
	* 通过数据库中的订单关联用户表，获取用户的地域信息

3. 深度思考：如何去除爬虫水军（商家为了提高自己的排名，用爬虫来频繁访问自己的店铺）？

	* 一段时间分析用户IP的访问次数
	
	* 关键字

## （三）技术方案

1. 数据采集逻辑（ETL）

	* 电商日志一般存储在日志服务器，通过 Flume 拉取到 HDFS 上。
	
	* 通过 Sqoop 从关系型数据库中读取数据。
	> **因为要访问数据库，所以会对数据库造成很大的压力，而且在真实的生产环境中，一般没有权限直接访问数据库。可以把数据导出成csv文件，放到日志服务器上，再通过Flume采集到HDFS上。假如有权限访问数据库，数据库也需要设置成读写分离的模式，来缓解压力。**

2. 数据清洗逻辑

	* 使用 MapReduce 进行数据清洗。
	
	* 使用 Spark Core 进行数据清洗。

3. 各区域热门商品的分析计算

	* 使用 Hive 进行数据的分析和处理。
	
	* 使用 Spark SQL 进行数据的分析和处理。

	* 注：一般不使用 MapReduce 进行分析处理，多表关联将导致编写 MapReduce 编程逻辑复杂。
	
## （四）实验数据及说明

[各区域热门商品测试数据](https://github.com/MrQuJL/area-hot-product/tree/master/data)

* product（商品）表：

	列名 | 描述 | 数据类型 | 空/非空约束 | 约束条件
	---|---|---|---|---
	product_id|商品号|varchar(18)|Not null|
	product_name|商品名称|varchar(20)|Not null|
	marque|商品型号|varchar(10)|Not null|
	barcode|仓库条码|varchar|Not null|
	price|商品价格|double|Not null|
	brand_id|商品品牌|varchar(8)|Not null|
	market_price|市场价格|double|Not null|
	stock|库存|int|Not null|
	status|状态|int|Not null|
	
	> 补充说明: status: 下架-1，上架0，预售1

* area_info（地区信息）表

	列名 | 描述 | 数据类型 | 空/非空约束 | 约束条件
	---|---|---|---|---
	area_id|地区编号|varchar(18)|Not null|
	area_name|地区名称|varchar(20)|Not null|

* user_click_log（用户点击信息）表

	列名 | 描述 | 数据类型 | 空/非空约束 | 约束条件
	---|---|---|---|---
	user_id|用户ID|varchar(18)|Not null|
	user_ip|用户IP|varchar(20)|Not null|
	url|用户点击 URL|varchar(200)||
	click_time|用户点击时间|varchar(40)||
	action_type|动作名称|varchar(40)||
	area_id|地区ID|varchar(40)||
	
	> 补充说明: action_type: 1 收藏，2 加购物车，3 购买  area_id:已经通过IP地址，解析出了区域信息

* area_hot_product（区域热门商品）表

	列名 | 描述 | 数据类型 | 空/非空约束 | 约束条件
	---|---|---|---|---
	area_id|地区ID|varchar(18)|Not null|
	area_name|地区名称|varchar(20)|Not null|
	product_id|商品ID|varchar(200)|
	product_name|商品名称|varchar(40)|
	pv|访问量|BIGINT||

## （五）技术实现

1. 使用Flume采集用户点击日志
	
	[Flume 配置文件](https://github.com/MrQuJL/area-hot-product/tree/master/01_etl)

	* 启动 Flume agent，在 Flume 的根目录下执行命令：bin/flume-ng agent -n a4 -f myagent/a4.conf -c conf -Dflume.root.logger=INFO,console
	
	* 向 /root/log0204 目录里放入[用户点击日志文件](https://github.com/MrQuJL/area-hot-product/tree/master/data/clicklog)
	
	* Flume 会将 /root/log0204 目录下的文件采集到 hdfs://qujianlei:9000/flume/ 目录下
	
2. 数据的清洗

	* 需要将用户点击日志里面对于商品的点击识别出来
	
	* 过滤不满足6个字段的数据
	
	* 过滤URL为空的数据，即：过滤出包含http开头的日志记录

	* 实现方式一：使用 MapReduce 程序进行数据的清洗
	
		* [源文件及 pom 文件](http://)
		
		* 打成 jar 包，提交到 yarn 上运行：hadoop jar clean-0.0.1-SNAPSHOT.jar clean/CleanDataMain /flume/20190204/events-.1549261170696 /output/190204
	
	* 实现方式二：使用 Spark 程序进行数据的清洗
	
	
3. 各区域热门商品热度统计：基于 Hive 和 Spark SQL
















