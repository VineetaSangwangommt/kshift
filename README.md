# kshift

Kshift is a ETL tool to persist data from kafka to Any Store. As of now it supports Redshift and S3 (Redshift Spectrum) as Sinks. 

It can be run simply by using the below specified command:
```spark-submit --class com.goibibo.dp.kafkaToRedshift.KafkaToRedshift kShift-assembly.jar --task-files taskfile.json --config-file kshiftconfig.conf```

Below is the explaination of various configs of task configs:

| Config Name  | Description | Type | Mandatory | Default |
| ------------ | ----------- | ---- | --------- | ------- |
| isUpsert | If true, data will be upserted. If false, data will be appended. Key on kafka queue is mandatory in this case as data is deduplicated using key only. | Boolean | No | false  |
| isSerialized | if false, processes the data as JSON. If true, uses the encoding class to encode the data. | Boolean | No | false  |
| deserializerClass | class used for deserializing the data in case of serialized data | String | No |  |
| schemaUrlTemplateAvro | Schema Url for template in case avro serialization | String | No |  |
| schemaUrlTopicAvro | Schema Url for topic in case avro serialization | String | No |  |
| columns | Columns being processed from the data. It wil be array of objects ColumnMapping. Column Mapping described later in this section. | List[ColumnMapping] | No |  |
| mergeKeyColumns | Key used in case of data upsert. Multiple columns can be provided in list form | List[String] | No |  |
| rerunOnSchemaChange | if false, processes the data as JSON. If true, uses the encoding class to encode the data. |  |  |  |
| takeDistinctValuesOnly | If true, only the distinct value are taken from kafka for a run. | Boolean | No | false  |
| takeOldestValue | If true, oldest value is taken from kafka for a run. In case false latest value is taken Only used when isUpsert or takeDistinctValuesOnly is true | Boolean | No | false  |
| dataSink | "class used for dataSink. For now there are just 2 classes :         com.goibibo.dp.kafkaToRedshift.dataSink.RedshiftSink and com.goibibo.dp.kafkaToRedshift.dataSink.S3Sink" | String | Yes |  |
| filterQuery | Sql query that can be used to enrich the data during processing only. | String | No |  |
| keyConverter | Converter used to get value from String. As of now, key can either be String or  |  |  |  |
| redshiftUrl | Redshift URL for dumping the data. | String | Yes |  |
| tempS3Location | Temporary S3 location for loading the data Redshift. |  |  |  |
| redshiftTable | Redshift table to which we need to dump data. | String | Yes |  |
| distStyle | Distribution Style in Redshift. | String | Yes |  |
| distKey | Distribution Key in Redshift in case distStyle is set to KEY. https://docs.aws.amazon.com/redshift/latest/dg/c_choosing_dist_sort.html | String | No |  |
| sortKeySpec | Sort Key in Redshift. To be given with Sort type. | String | No |  |
| extraCopyOptions | Extra options for redshift copy. | String | Yes |  |
| preActions | Query that can be executed on redshift before loading the data. | String | No |  |
| postActions | Query that can be executed on redshift after loading the data. | String | No |  |
| redshiftWriteMode | Append/Overwrite | String | No | Append |
| topicName | Kafka Topic name to read from. | String | Yes |  |
| kafkaBrokers | Brokers for kafka. | String | Yes |  |
| kafkaConsumerGroup | Consumer group for kafka consumption.Auto created with table name and topic name in case not provided | String | No |  |
| messagesPerRun | Number of messages to be processed in single run. if it is less than the number of messages in kafka, then multiple runs will be executed for single task. In case no value is provided all the messages from kafka are consumed in single shot.  | Int | No |  |
| s3SinkConfig | S3 Properties for Sink to S3. It will be object of S3SinkConfig which is described in later sections. | Array[S3SinkConfig] | No |  |
| metricConfig | Metrics Logger Configs to record the metrics of the run and task. Right now there are 2 Metric Recorders: File and DynamoDB | MetricConfig | No | DynamoDB recorder |


S3 Sink Config:

| Config Name | Description | Type | Mandatory | Default |
| ----------- | ----------- | ---- | --------- | ------- |
|s3Location | Location in S3 where data needs to be dumped. | String |  | |
|createPartitions | Create partition in the data. if true, partitions will be created. As of now, partition can only be created using date and hour. | Boolean | No | true |
|addPartitions | Add Partititions to Spectrum. If true, partitions will be added. | Boolean | No | false |
|partitionConfig | Configuration about the partitions. Described in next section. | PartitionConfig | Yes | |


Partition Config:

| Config Name | Description | Type | Mandatory | Default |
| ----------- | ----------- | ---- | --------- | ------- |
| partitionFieldName | Column name to be used for partition. | String | No | Current date |
| partitionFieldFormat | Format of the column name to be used to partition. Give date format or timestamp in case of epoch.  | String | No | timestamp |
| datePartitionFormat | Format of date in partitioned data | String | No | yyyy-MM-dd |
| datePartitionName | Name of date column in partitioned data | String | No | __date |
| hourPartitionFormat | Format of hour in partitioned data | String | No | HH |
| hourPartitionName | Name of hour column in partitioned data | String | No | __hour |


Coloumn Mapping: 

| Config Name | Description | Type | Mandatory | Default |
| ----------- | ----------- | ---- | --------- | ------- |
| columnName | Name of the column in Redshift or Spectrum | String | Yes |  |
| columnType | Type of Column in Redshift. | String | No |  |
| columnSource | Source of the column from data in kafka. | String | Yes |  |
| sourceFormat | Format of the column. Right now used only for date. e.g. 'dd-MM-yyyy' | String | No | Calculated on runtime.  |
| columnEncoding | Encoding in Redshift. | String | No | ZSTD |


Metric Recorder Config:

| Config Name | Description | Type | Mandatory | Default |
| ----------- | ----------- | ---- | --------- | ------- |
| className | "Class name for Metric Recorder. Below are the Metric Recorder classes that are implemented. com.goibibo.dp.kafkaToRedshift.metrics.FileMetricRecorder and com.goibibo.dp.kafkaToRedshift.metrics.DynamodbMetricRecorder" | String | no | com.goibibo.dp.kafkaToRedshift.metrics.DynamodbMetricRecorder |
| extraConfig | In a Map you can define all the extra configs to be used by Metric Recorders. | String | no |  |




