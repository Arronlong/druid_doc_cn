# 基于Hadoop的批量摄取

通过Hadoop摄取任务支持Druid中基于Hadoop的批量摄取。这些任务可以发布到德鲁伊[霸主](http://druid.io/docs/0.12.3/design/overlord.html)的正在运行的实例中。

## 命令行Hadoop Indexer

如果您不想使用完整索引服务来使用Hadoop将数据导入Druid，您还可以使用独立命令行Hadoop索引器。有关详细信息，请参见[此处](http://druid.io/docs/0.12.3/ingestion/command-line-hadoop-indexer.html)

## 任务语法

示例任务如下所示：

```json
{
  "type" : "index_hadoop",
  "spec" : {
    "dataSchema" : {
      "dataSource" : "wikipedia",
      "parser" : {
        "type" : "hadoopyString",
        "parseSpec" : {
          "format" : "json",
          "timestampSpec" : {
            "column" : "timestamp",
            "format" : "auto"
          },
          "dimensionsSpec" : {
            "dimensions": ["page","language","user","unpatrolled","newPage","robot","anonymous","namespace","continent","country","region","city"],
            "dimensionExclusions" : [],
            "spatialDimensions" : []
          }
        }
      },
      "metricsSpec" : [
        {
          "type" : "count",
          "name" : "count"
        },
        {
          "type" : "doubleSum",
          "name" : "added",
          "fieldName" : "added"
        },
        {
          "type" : "doubleSum",
          "name" : "deleted",
          "fieldName" : "deleted"
        },
        {
          "type" : "doubleSum",
          "name" : "delta",
          "fieldName" : "delta"
        }
      ],
      "granularitySpec" : {
        "type" : "uniform",
        "segmentGranularity" : "DAY",
        "queryGranularity" : "NONE",
        "intervals" : [ "2013-08-31/2013-09-01" ]
      }
    },
    "ioConfig" : {
      "type" : "hadoop",
      "inputSpec" : {
        "type" : "static",
        "paths" : "/MyDirectory/example/wikipedia_data.json"
      }
    },
    "tuningConfig" : {
      "type": "hadoop"
    }
  },
  "hadoopDependencyCoordinates": <my_hadoop_version>
}
```

| 属性                        | 描述                                                         | 需要？ |
| --------------------------- | ------------------------------------------------------------ | ------ |
| 类型                        | 任务类型，这应该始终是“index_hadoop”。                       | 是     |
| 规范                        | Hadoop指数规范 见[吞食](http://druid.io/docs/0.12.3/ingestion/ingestion-spec.html) | 是     |
| hadoopDependencyCoordinates | A JSON array of Hadoop dependency coordinates that Druid will use, this property will override the default Hadoop coordinates. Once specified, Druid will look for those Hadoop dependencies from the location specified by `druid.extensions.hadoopDependenciesDir` | no     |
| classpathPrefix             | Classpath that will be pre-appended for the peon process.    | no     |

also note that, druid automatically computes the classpath for hadoop job containers that run in hadoop cluster. But, in case of conflicts between hadoop and druid's dependencies, you can manually specify the classpath by setting `druid.extensions.hadoopContainerDruidClasspath` property. See the extensions config in [base druid configuration](http://druid.io/docs/0.12.3/configuration/index.html#extensions).

## DataSchema

This field is required. See [Ingestion Spec DataSchema](http://druid.io/docs/0.12.3/ingestion/ingestion-spec.html#dataschema).

## IOConfig

This field is required.

| Field              | Type   | Description                                                  | Required                                                     |
| ------------------ | ------ | ------------------------------------------------------------ | ------------------------------------------------------------ |
| type               | String | This should always be 'hadoop'.                              | yes                                                          |
| inputSpec          | Object | A specification of where to pull the data in from. See below. | yes                                                          |
| segmentOutputPath  | String | The path to dump segments into.                              | yes                                                          |
| metadataUpdateSpec | Object | A specification of how to update the metadata for the druid cluster these segments belong to. | Only used by the [CLI Hadoop Indexer](http://druid.io/docs/0.12.3/ingestion/command-line-hadoop-indexer.html). This field must be null otherwise. |

### InputSpec specification

There are multiple types of inputSpecs:

#### `static`

A type of inputSpec where a static path to the data files is provided.

| Field | Type            | Description                                                  | Required |
| ----- | --------------- | ------------------------------------------------------------ | -------- |
| paths | Array of String | A String of input paths indicating where the raw data is located. | yes      |

For example, using the static input paths:

```text
"paths" : "s3n://billy-bucket/the/data/is/here/data.gz,s3n://billy-bucket/the/data/is/here/moredata.gz,s3n://billy-bucket/the/data/is/here/evenmoredata.gz"
```

#### `granularity`

A type of inputSpec that expects data to be organized in directories according to datetime using the path format: `y=XXXX/m=XX/d=XX/H=XX/M=XX/S=XX` (where date is represented by lowercase and time is represented by uppercase).

| Field           | Type   | Description                                                  | Required |
| --------------- | ------ | ------------------------------------------------------------ | -------- |
| dataGranularity | String | Specifies the granularity to expect the data at, e.g. hour means to expect directories `y=XXXX/m=XX/d=XX/H=XX`. | yes      |
| inputPath       | String | Base path to append the datetime path to.                    | yes      |
| filePattern     | String | Pattern that files should match to be included.              | yes      |
| pathFormat      | String | Joda datetime format for each directory. Default value is `"'y'=yyyy/'m'=MM/'d'=dd/'H'=HH"`, or see [Joda documentation](http://www.joda.org/joda-time/apidocs/org/joda/time/format/DateTimeFormat.html) | no       |

For example, if the sample config were run with the interval 2012-06-01/2012-06-02, it would expect data at the paths:

```text
s3n://billy-bucket/the/data/is/here/y=2012/m=06/d=01/H=00
s3n://billy-bucket/the/data/is/here/y=2012/m=06/d=01/H=01
...
s3n://billy-bucket/the/data/is/here/y=2012/m=06/d=01/H=23
```

#### `dataSource`

Read Druid segments. See [here](http://druid.io/docs/0.12.3/ingestion/update-existing-data.html) for more information.

#### `multi`

Read multiple sources of data. See [here](http://druid.io/docs/0.12.3/ingestion/update-existing-data.html) for more information.

## TuningConfig

The tuningConfig is optional and default parameters will be used if no tuningConfig is specified.

| Field                       | Type    | Description                                                  | Required                                                     |
| --------------------------- | ------- | ------------------------------------------------------------ | ------------------------------------------------------------ |
| workingPath                 | String  | The working path to use for intermediate results (results between Hadoop jobs). | Only used by the [CLI Hadoop Indexer](http://druid.io/docs/0.12.3/ingestion/command-line-hadoop-indexer.html). The default is '/tmp/druid-indexing'. This field must be null otherwise. |
| version                     | String  | The version of created segments. Ignored for HadoopIndexTask unless useExplicitVersion is set to true | no (default == datetime that indexing starts at)             |
| partitionsSpec              | Object  | A specification of how to partition each time bucket into segments. Absence of this property means no partitioning will occur. See 'Partitioning specification' below. | no (default == 'hashed')                                     |
| maxRowsInMemory             | Integer | The number of rows to aggregate before persisting. Note that this is the number of post-aggregation rows which may not be equal to the number of input events due to roll-up. This is used to manage the required JVM heap size. | no (default == 75000)                                        |
| leaveIntermediate           | Boolean | Leave behind intermediate files (for debugging) in the workingPath when a job completes, whether it passes or fails. | no (default == false)                                        |
| cleanupOnFailure            | Boolean | Clean up intermediate files when a job fails (unless leaveIntermediate is on). | no (default == true)                                         |
| overwriteFiles              | Boolean | Override existing files found during indexing.               | no (default == false)                                        |
| ignoreInvalidRows           | Boolean | Ignore rows found to have problems.                          | no (default == false)                                        |
| combineText                 | Boolean | Use CombineTextInputFormat to combine multiple files into a file split. This can speed up Hadoop jobs when processing a large number of small files. | no (default == false)                                        |
| useCombiner                 | Boolean | Use Hadoop combiner to merge rows at mapper if possible.     | no (default == false)                                        |
| jobProperties               | Object  | A map of properties to add to the Hadoop job configuration, see below for details. | no (default == null)                                         |
| indexSpec                   | Object  | Tune how data is indexed. See below for more information.    | no                                                           |
| numBackgroundPersistThreads | Integer | The number of new background threads to use for incremental persists. Using this feature causes a notable increase in memory pressure and cpu usage but will make the job finish more quickly. If changing from the default of 0 (use current thread for persists), we recommend setting it to 1. | no (default == 0)                                            |
| forceExtendableShardSpecs   | Boolean | Forces use of extendable shardSpecs. Experimental feature intended for use with the [Kafka indexing service extension](http://druid.io/docs/0.12.3/development/extensions-core/kafka-ingestion.html). | no (default = false)                                         |
| useExplicitVersion          | Boolean | Forces HadoopIndexTask to use version.                       | no (default = false)                                         |

### jobProperties field of TuningConfig

```json
   "tuningConfig" : {
     "type": "hadoop",
     "jobProperties": {
       "<hadoop-property-a>": "<value-a>",
       "<hadoop-property-b>": "<value-b>"
     }
   }
```

Hadoop's [MapReduce documentation](https://hadoop.apache.org/docs/stable/hadoop-mapreduce-client/hadoop-mapreduce-client-core/mapred-default.xml) lists the possible configuration parameters.

With some Hadoop distributions, it may be necessary to set `mapreduce.job.classpath` or `mapreduce.job.user.classpath.first` to avoid class loading issues. See the [working with different Hadoop versions documentation](http://druid.io/docs/0.12.3/operations/other-hadoop.html) for more details.

### IndexSpec

| Field                | Type   | Description                                                  | Required                 |
| -------------------- | ------ | ------------------------------------------------------------ | ------------------------ |
| bitmap               | Object | Compression format for bitmap indexes. Should be a JSON object; see below for options. | no (defaults to Concise) |
| dimensionCompression | String | Compression format for dimension columns. Choose from `LZ4`, `LZF`, or `uncompressed`. | no (default == `LZ4`)    |
| metricCompression    | String | Compression format for metric columns. Choose from `LZ4`, `LZF`, `uncompressed`, or `none`. | no (default == `LZ4`)    |
| longEncoding         | String | Encoding format for metric and dimension columns with type long. Choose from `auto` or `longs`. `auto` encodes the values using offset or lookup table depending on column cardinality, and store them with variable size. `longs` stores the value as is with 8 bytes each. | no (default == `longs`)  |

#### Bitmap types

For Concise bitmaps:

| Field | Type   | Description        | Required |
| ----- | ------ | ------------------ | -------- |
| type  | String | Must be `concise`. | yes      |

For Roaring bitmaps:

| Field                      | Type    | Description                                                  | Required               |
| -------------------------- | ------- | ------------------------------------------------------------ | ---------------------- |
| type                       | String  | Must be `roaring`.                                           | yes                    |
| compressRunOnSerialization | Boolean | Use a run-length encoding where it is estimated as more space efficient. | no (default == `true`) |

## Partitioning specification

Segments are always partitioned based on timestamp (according to the granularitySpec) and may be further partitioned in some other way depending on partition type. Druid supports two types of partitioning strategies: "hashed" (based on the hash of all dimensions in each row), and "dimension" (based on ranges of a single dimension).

Hashed partitioning is recommended in most cases, as it will improve indexing performance and create more uniformly sized data segments relative to single-dimension partitioning.

### Hash-based partitioning

```json
  "partitionsSpec": {
     "type": "hashed",
     "targetPartitionSize": 5000000
   }
```

Hashed partitioning works by first selecting a number of segments, and then partitioning rows across those segments according to the hash of all dimensions in each row. The number of segments is determined automatically based on the cardinality of the input set and a target partition size.

The configuration options are:

| Field               | Description                                                  | Required                           |
| ------------------- | ------------------------------------------------------------ | ---------------------------------- |
| type                | Type of partitionSpec to be used.                            | "hashed"                           |
| targetPartitionSize | Target number of rows to include in a partition, should be a number that targets segments of 500MB~1GB. | either this or numShards           |
| numShards           | Specify the number of partitions directly, instead of a target partition size. Ingestion will run faster, since it can skip the step necessary to select a number of partitions automatically. | either this or targetPartitionSize |
| partitionDimensions | The dimensions to partition on. Leave blank to select all dimensions. Only used with numShards, will be ignored when targetPartitionSize is set | no                                 |

### Single-dimension partitioning

```json
  "partitionsSpec": {
     "type": "dimension",
     "targetPartitionSize": 5000000
   }
```

Single-dimension partitioning works by first selecting a dimension to partition on, and then separating that dimension into contiguous ranges. Each segment will contain all rows with values of that dimension in that range. For example, your segments may be partitioned on the dimension "host" using the ranges "a.example.com" to "f.example.com" and "f.example.com" to "z.example.com". By default, the dimension to use is determined automatically, although you can override it with a specific dimension.

The configuration options are:

| Field               | Description                                                  | Required    |
| ------------------- | ------------------------------------------------------------ | ----------- |
| type                | Type of partitionSpec to be used.                            | "dimension" |
| targetPartitionSize | Target number of rows to include in a partition, should be a number that targets segments of 500MB~1GB. | yes         |
| maxPartitionSize    | Maximum number of rows to include in a partition. Defaults to 50% larger than the targetPartitionSize. | no          |
| partitionDimension  | The dimension to partition on. Leave blank to select a dimension automatically. | no          |
| assumeGrouped       | Assume that input data has already been grouped on time and dimensions. Ingestion will run faster, but may choose sub-optimal partitions if this assumption is violated. | no          |

## Remote Hadoop Cluster

If you have a remote Hadoop cluster, make sure to include the folder holding your configuration `*.xml` files in your Druid `_common` configuration folder.

If you are having dependency problems with your version of Hadoop and the version compiled with Druid, please see [these docs](http://druid.io/docs/0.12.3/operations/other-hadoop.html).

## Using Elastic MapReduce

If your cluster is running on Amazon Web Services, you can use Elastic MapReduce (EMR) to index data from S3. To do this:

- Create a persistent, [long-running cluster](http://docs.aws.amazon.com/ElasticMapReduce/latest/ManagementGuide/emr-plan-longrunning-transient.html).
- When creating your cluster, enter the following configuration. If you're using the wizard, this should be in advanced mode under "Edit software settings":

```text
classification=yarn-site,properties=[mapreduce.reduce.memory.mb=6144,mapreduce.reduce.java.opts=-server -Xms2g -Xmx2g -Duser.timezone=UTC -Dfile.encoding=UTF-8 -XX:+PrintGCDetails -XX:+PrintGCTimeStamps,mapreduce.map.java.opts=758,mapreduce.map.java.opts=-server -Xms512m -Xmx512m -Duser.timezone=UTC -Dfile.encoding=UTF-8 -XX:+PrintGCDetails -XX:+PrintGCTimeStamps,mapreduce.task.timeout=1800000]
```

- Follow the instructions under "[Configure Hadoop for data loads](http://druid.io/docs/0.12.3/tutorials/cluster.html#configure-cluster-for-hadoop-data-loads)" using the XML files from `/etc/hadoop/conf` on your EMR master.

## Secured Hadoop Cluster

By default druid can use the exisiting TGT kerberos ticket available in local kerberos key cache. Although TGT ticket has a limited life cycle, therefore you need to call `kinit` command periodically to ensure validity of TGT ticket. To avoid this extra external cron job script calling `kinit` periodically, you can provide the principal name and keytab location and druid will do the authentication transparently at startup and job launching time.

| Property                                   | Possible Values                                   | Description         | Default |
| ------------------------------------------ | ------------------------------------------------- | ------------------- | ------- |
| `druid.hadoop.security.kerberos.principal` | `druid@EXAMPLE.COM`                               | Principal user name | empty   |
| `druid.hadoop.security.kerberos.keytab`    | `/etc/security/keytabs/druid.headlessUser.keytab` | Path to keytab file | empty   |

## Loading from S3 with EMR

- In the `jobProperties` field in the `tuningConfig` section of your Hadoop indexing task, add:

```text
"jobProperties" : {
   "fs.s3.awsAccessKeyId" : "YOUR_ACCESS_KEY",
   "fs.s3.awsSecretAccessKey" : "YOUR_SECRET_KEY",
   "fs.s3.impl" : "org.apache.hadoop.fs.s3native.NativeS3FileSystem",
   "fs.s3n.awsAccessKeyId" : "YOUR_ACCESS_KEY",
   "fs.s3n.awsSecretAccessKey" : "YOUR_SECRET_KEY",
   "fs.s3n.impl" : "org.apache.hadoop.fs.s3native.NativeS3FileSystem",
   "io.compression.codecs" : "org.apache.hadoop.io.compress.GzipCodec,org.apache.hadoop.io.compress.DefaultCodec,org.apache.hadoop.io.compress.BZip2Codec,org.apache.hadoop.io.compress.SnappyCodec"
}
```

请注意，此方法使用Hadoop的内置S3文件系统而不是Amazon的EMRFS，并且与特定于Amazon的功能（如S3加密和一致视图）不兼容。如果您需要使用这些功能，则需要通过[使用其他Hadoop发行版](http://druid.io/docs/0.12.3/ingestion/hadoop.html#using-other-hadoop-distributions)部分中描述的一种机制使Amazon EMR Hadoop JAR可用于Druid 。

## 使用其他Hadoop发行版

德鲁伊开箱即用，有许多Hadoop发行版。

如果你在Druid和你的Hadoop版本之间存在依赖冲突，你可以尝试在[Druid用户组中](https://groups.google.com/forum/#!forum/druid-%0Auser)搜索解决方案，或者阅读Druid [Different Hadoop Versions](http://druid.io/docs/0.12.3/operations/other-hadoop.html)文档。