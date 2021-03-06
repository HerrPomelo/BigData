# Spark Official Document

## 1.快速入门

## 2.Programming Guide

- 必须创建SparkContext对象,是spark程序的入口
```
import org.apache.spark.SparkContext
import org.apache.spark.SparkConf

val conf = new SparkConf().setAppName(appName).setMaster(master) 
val sc = new SparkContext(conf)
```
- master类型有四个:local,yarn,standalone,mesos
- 创建RDD方式有两种，Parallelized和外部数据源
	- 标准读取代码
	```
	val distFile = sc.textFile(URLPath)
	```
	- 创建的partitions分区数和HDFS文件数相同，也可以设置更高的数量
- RDD操作
	- 两种类型操作，transformations和actions，transformations是惰性的
	- RDDs.collect()集中在driver上
	- transformations的算子：map,filter,flatmap,sample,union,distinct,groupByKey,reduceByKey,sortByKey,join,coalesce,repatition
	- action的算子:reduce,collect,count,take,saveAsText,countByKey,foreach
	- 有shuffle过程的算子：coalesce,repatition,groupByKey,reduceByKey,join
- RDD持久化
	- persist(),可以设置存储级别
	- cache() 保存到内存
- 共享变量
	- Broadcast variables,保存一个只读的变量在executor中
	```
	val broadcastVar = sc.broadcast(RDD)
	```
	- Accumulators
	```
	val accum = sc.longAccumulator("My Accumulator")

	// 只能在driver读取累加器的值
	accum.value
	```



## 3. Spark Streaming

## 4. Spark SQL, DataFrames and Datasets Guide

### 4.1 概述
- SparkSQL执行SQL的方式有两种:sql 和 DF API,其中无论使用SQL，DF API，DS API，底层执行引擎相同
- SQL,执行SQL语句会返还DF/DS
- Dataset也是一个分布式数据集合，会使用Encoder去序列化Java对象，Encoder可以保障一些操作如filter可以不用反序列化就可以进行
- DataFrame是一个Dataset组织成的指定列，相当与表 ？？

### 4.2 入门指南
- 标准操作代码
```
// 创建SparkSession
import org.apache.spark.sql.SparkSession
val spark = SparkSession
  .builder()
  .appName("Spark SQL basic example")
  .config("spark.some.config.option", "some-value")
  .enableHiveSupport()
  .getOrCreate()

// 创建DF
val df = spark.read.json("examples/src/main/resources/people.json")
df.show()

// 通过sql操作df
df.createOrReplaceTempView("people")

val sqlDF = spark.sql("SELECT * FROM people")
sqlDF.show()

//

```
- 创建temporary views，在这个session结束后会消失，但是global temporary views表会在spark application结束才会消失
- RDD转变为DataFrame的方法 ？？
	- 反射推测Schema，利用反射来推断包含特定类型对象的RDD的schema
	```
	import spark.implicits._

	case class Person(name: String, age: Long)

	val peopleDF = spark.sparkContext
		  .textFile("examples/src/main/resources/people.txt")
		  .map(_.split(","))
		  .map(attributes => Person(attributes(0), attributes(1).trim.toInt))
		  .toDF()

	```
	- 编程接口方式指定Schema,构造一个schema并将其应用在已知的RDD上，可以动态生成,不知道列和列的类型
	```
	步骤
	// 从RDD创建row RDD
    // 创建StructType类型的Schema
    // row RDD和Schema匹配

	import org.apache.spark.sql.types._

	val peopleRDD = spark.sparkContext.textFile("examples/src/main/resources/people.txt")

	val schemaString = "name age"

	val fields = schemaString.split(" ")
	  .map(fieldName => StructField(fieldName, StringType, nullable = true))
	val schema = StructType(fields)

	val rowRDD = peopleRDD
	  .map(_.split(","))
	  .map(attributes => Row(attributes(0), attributes(1).trim))

	val peopleDF = spark.createDataFrame(rowRDD, schema)

	```
### 4.3 数据源
- 标准读写外部数据源的方法
```
val peopleDF = spark.read.format("json").load("examples/src/main/resources/people.json")

peopleDF.select("name", "age").write.format("parquet").save("namesAndAges.parquet")
```
- SaveMode的四种模式,error,append,overwrite,ignore
- partiton中，如果read到partiton目录，就没有partiton这列，如果到上一级，就有partiton这列
- Hive
	- 使用hive首先在conf中配置hive-site.xml,其次在SparkSession中加入enableHiveSupport,还要加入MySQL的驱动
	- 基本操作
	```
	val spark = SparkSession
	  .builder()
	  .appName("Spark Hive Example")
	  .config("spark.sql.warehouse.dir", warehouseLocation)
	  .enableHiveSupport()
	  .getOrCreate()

	spark.sql("CREATE TABLE IF NOT EXISTS src (key INT, value STRING)")

	spark.sql("LOAD DATA LOCAL INPATH 'examples/src/main/resources/kv1.txt' INTO TABLE src")

	spark.sql("select * from src")

	```
- 通过JDBC读取其他databases
```
// 读取MySQL数据
val jdbcDF = spark.read
  .format("jdbc")
  .option("url", "jdbc:postgresql:dbserver")
  .option("dbtable", "schema.tablename")
  .option("user", "username")
  .option("password", "password")
  .load()

// 写入MySQL数据，参考ImoocLogApp的操作方法  

```
### 4.4 性能调优
- dataFrame.cache()缓存DF
- spark.sql.shuffle.partitions 设置partitions数

## 5. MLlib



