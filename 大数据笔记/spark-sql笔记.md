# Spark-SQL
## 1 概述
### Spark-SQL是什么
SparkSQL是Spark用于处理结构化数据的一个组件，它的编程抽象是DataFrame，可以理解为一个基于RDD数据模型并带有结构化元信息（schema）的数据模型。

### Spark-SQL的特性
- 易整合：可以在编程中混搭SQL和算子API；
- 统一的数据访问方式：为各类数据源提供统一的访问方式，可以跨数据源进行join。支持的数据源有Hive、CSV、Parquet、ORC、josn、JDBC等。
- 兼容Hive：支持HiveQL语法，并允许访问已经存在的Hive数仓数据；
- 标准的数据连接： 可以当作一个转换层，向下对接各种不同的结构化数据源，向上提供不同的数据访问方式。

## 2 DataFrame编程
> 我们使用sparksql加载结构化数据 返回Dataframe(DataSet) , 
> - 使用DF创建视图后就可以编写SQL分析处理数据
> - 在DF上还有类似于SQL的API select  where  groupBy  sum ....TableAPI 
> 
> SparkSQL编程其实就是对Dataframe进行编程

### 2.1 创建DataFrame-加载数据源
在Spark SQL中SparkSession是创建DataFrames和执行SQL的入口
创建DataFrames有三种方式：
1) 从一个已存在的RDD进行转换
2) 从JSON/Parquet/CSV/ORC等结构化文件源创建
1) 从Hive/JDBC各种外部结构化数据源（服务）创建

核心要义：创建DataFrame，需要创建 “RDD + 元信息schema定义” + 执行计划

#### 加载结构化数据
```scala
import org.apache.spark.sql.SparkSession

object Demo01_MakeDF_Csv {
  def main(args: Array[String]): Unit = {

    val session = SparkSession
      .builder()
      .master("local[*]")
      .appName("获取 spark-sql环境")
      .config("", "")
      // 添加对hive的支持
      // .enableHiveSupport()
      .getOrCreate()

    val df = session.read
    .option("header" , true)    // 加载有文件头的内容
    .option("inferSchema",true)     // 自动推导数据类型
    .csv("data/csv/Teacher2.csv")
     // 打印结构
    df.printSchema()
```

#### 自定义数据的schema
```scala
def main(args: Array[String]): Unit = {
  val session = SparkUtil.getSession
  // 根据结构化数据的结构  自定义  Schema
  val structType = new StructType()
    .add("tid" , DataTypes.LongType)
    .add("tname" , DataTypes.StringType)
    .add("tage" , DataTypes.IntegerType)
    .add("tgender",DataTypes.StringType)
    .add("tcity" , DataTypes.StringType)
  // 加载数据的时候指定schema信息
  val df = session.read.schema(structType).csv("data/csv/Teacher.csv")

  /**
   * root
   * |-- tid: long (nullable = true)
   * |-- tname: string (nullable = true)
   * |-- tage: integer (nullable = true)
   * |-- tgender: string (nullable = true)
   * |-- tcity: string (nullable = true)
   */

  // 打印默认的结构和数据
  df.printSchema()
```

#### DataTypes支持的数据类型
```scala
public static final DataType StringType;
public static final DataType BinaryType;
public static final DataType BooleanType;
public static final DataType DateType;
public static final DataType TimestampType;
public static final DataType CalendarIntervalType;
public static final DataType DoubleType;
public static final DataType FloatType;
public static final DataType ByteType;
public static final DataType IntegerType;
public static final DataType LongType;
public static final DataType ShortType;
public static final DataType NullType;
createArrayType
createDecimalType
createMapType
```

### 2.2 加载外部数据源
#### 加载MySQL 创建DF
1. 添加MySQL的依赖 
2. 有操作的MySQL中的表 
3. 编程

```scala
import java.util.Properties
import cn.doitedu.util.SparkUtil

object Demo02_MakeDF_MySQL {
  def main(args: Array[String]): Unit = {

    val session = SparkUtil.getSession
    /**
     * 参数一  JDBC 协议的url定位到数据库
     * 参数二   表名
     * 参数三   连接数据库的必要参数
     */
    val  uri = "jdbc:mysql://localhost:3306/demo"
    val  tableName = "tb_student"
    val props =  new Properties()
    props.setProperty("user" ,"root")
    props.setProperty("password" , "root")

    val df = session.read.jdbc(uri, tableName, props)
    df.printSchema()
    df.show()
    // 创建视图
    df.createTempView("tb")
    // 编写SQL  统计需求
    session.sql(
      """
        |select
        |*
        |from
        |tb
        |""".stripMargin).show()
  }
}
```

#### 加载hive中的表 创建DF
1. 导入MySQL的依赖
2. 导入spark和hive的整合依赖
   - core-site.xml  核心是 HDFS文件系统的地址
   - hive-site.xml  hive有关的信息  [元数据服务的地址]
   - hdfs-site.xml  从HDFS上加载数据  
3. 元数据服务启动 hohup hive --service metastore 1>/dev/null 2>&1 &

```scala
object Demo03_MakeDF_hive {
  Logger.getLogger("org").setLevel(Level.ERROR)
  def main(args: Array[String]): Unit = {

    val session = SparkSession
      .builder()
      .master("local[*]")
      .appName(this.getClass.getSimpleName)
      // 添加对hive的支持
      /**
       * 添加MySQL和spark-hive的依赖
       * 添加hive-site.xml   访问元数据服务
       *
       */
      .enableHiveSupport()
      .getOrCreate()
    /*val df = session.sql("show databases")
    val df = session.sql("show tables")*/
    val df = session.sql("select * from  default.tb_order")
    df.show()
  }
}
```

#### 由RDD转换成DF
```scala
package cn.doitedu.sql

import cn.doitedu.utils.SparkUtil
import org.apache.spark.rdd.RDD

case  class  Teacher(tid:Int , tName:String, tage:Int, tgender:String , tHeight:Long)
object Demo04_CreateDF_RDD_CaseClass {
  def main(args: Array[String]): Unit = {
    val session = SparkUtil.getSession
    // 获取RDD
    val sc = session.sparkContext

    val rdd: RDD[String] = sc.textFile("data/teacher/")
  // 加载数据  封装成RDD
    val caseClassRDD: RDD[Teacher] = rdd.map(line => {
      val arr = line.split(",")
      //2,星哥,46,M,180
      Teacher(arr(0).toInt, arr(1), arr(2).toInt, arr(3), arr(4).toLong)
    })
    /**
     * case class 有数据结构 属性名  属性数据类型
     */
    val df = session.createDataFrame(caseClassRDD)

    df.printSchema()
    df.show()

    session.close()

  }
}
```