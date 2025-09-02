# info-flink
info-flink

!!!!! A lot of flink information is in your repo https://github.com/bobdemp/info-confluent-kafka !!!!! 
******************************************************************************************************************************
--------------------------------------
Overview
--------------------------------------

-- Flink Architecture

https://nightlies.apache.org/flink/flink-docs-release-2.0/docs/learn-flink/overview/

https://nightlies.apache.org/flink/flink-docs-stable/docs/concepts/flink-architecture/

-- Flink Web UI

http://localhost:8081
	
-- Apache Kafka® and Apache Flink® Tutorials

https://developer.confluent.io/tutorials/#master-advanced-concepts	

--------------------------------------
Web Links
--------------------------------------
 
--------------------------------------
Good Vids
--------------------------------------

Apache Flink Series-The Ultimate Flink DataStream Course

https://www.youtube.com/playlist?list=PLD6DcxwkW8BcbMSbNWeg_xKhS3kS5DQcB 

https://www.udemy.com/course/apache-flink-series-the-ultimate-flink-datastream-course/?referralCode=023912394FCC80BA1341&couponCode=LEARNNOWPLANSGB

https://www.udemy.com/course/apache-flink-a-real-time-hands-on-course-on-flink/?couponCode=LEARNNOWPLANSGB

******************************************************************************************************************************
--------------------------------------
Checkpoints vs. Savepoints
--------------------------------------
	
https://nightlies.apache.org/flink/flink-docs-master/docs/ops/state/checkpoints_vs_savepoints/

In order to make state fault tolerant, Flink needs to checkpoint the state. Checkpoints allow Flink to recover state and positions in the streams to give the application the same semantics as a failure-free execution.

A Savepoint is a consistent image of the execution state of a streaming job, created via Flink’s checkpointing mechanism. You can use Savepoints to stop-and-resume, fork, or update your Flink jobs. Savepoints consist of two parts: a directory with (typically large) binary files on stable storage (e.g. HDFS, S3, …) and a (relatively small) meta data file. The files on stable storage represent the net data of the job’s execution state image. The meta data file of a Savepoint contains (primarily) pointers to all files on stable storage that are part of the Savepoint, in form of relative paths.

Conceptually, Flink’s savepoints are different from checkpoints in a way that’s analogous to how backups are different from recovery logs in traditional database systems.

The primary purpose of checkpoints is to provide a recovery mechanism in case of unexpected job failures. A checkpoint’s lifecycle is managed by Flink, i.e. a checkpoint is created, owned, and released by Flink - without user interaction. Because checkpoints are being triggered often, and are relied upon for failure recovery, the two main design goals for the checkpoint implementation are i) being as lightweight to create and ii) being as fast to restore from as possible. Optimizations towards those goals can exploit certain properties, e.g., that the job code doesn’t change between the execution attempts.

Checkpoints are automatically deleted if the application is terminated by the user (except if checkpoints are explicitly configured to be retained).
Checkpoints are stored in state backend-specific (native) data format (may be incremental depending on the specific backend).
Although savepoints are created internally with the same mechanisms as checkpoints, they are conceptually different and can be a bit more expensive to produce and restore from. Their design focuses more on portability and operational flexibility, especially with respect to changes to the job. The use case for savepoints is for planned, manual operations. For example, this could be an update of your Flink version, changing your job graph, and so on.

Savepoints are created, owned and deleted solely by the user. That means, Flink does not delete savepoints neither after job termination nor after restore.
Savepoints are stored in a state backend independent (canonical) format (Note: Since Flink 1.15, savepoints can be also stored in the backend-specific native format which is faster to create and restore but comes with some limitations.	

******************************************************************************************************************************
--------------------------------------
WaterMarks
--------------------------------------
		
https://nightlies.apache.org/flink/flink-docs-release-2.0/docs/learn-flink/streaming_analytics/#working-with-watermarks
	
This is precisely what watermarks do — they define when to stop waiting for earlier events.
	
###

Let’s work through a simple example that will show why watermarks are needed, and how they work.

In this example you have a stream of timestamped events that arrive somewhat out of order, as shown below. The numbers shown are timestamps that indicate when these events actually occurred. The first event to arrive happened at time 4, and it is followed by an event that happened earlier, at time 2, and so on:

··· 23 19 22 24 21 14 17 13 12 15 9 11 7 2 4 →

Now imagine that you are trying create a stream sorter. This is meant to be an application that processes each event from a stream as it arrives, and emits a new stream containing the same events, but ordered by their timestamps.

Some observations:

(1) The first element your stream sorter sees is the 4, but you can’t just immediately release it as the first element of the sorted stream. It may have arrived out of order, and an earlier event might yet arrive. In fact, you have the benefit of some god-like knowledge of this stream’s future, and you can see that your stream sorter should wait at least until the 2 arrives before producing any results.

Some buffering, and some delay, is necessary.

(2) If you do this wrong, you could end up waiting forever. First the sorter saw an event from time 4, and then an event from time 2. Will an event with a timestamp less than 2 ever arrive? Maybe. Maybe not. You could wait forever and never see a 1.

Eventually you have to be courageous and emit the 2 as the start of the sorted stream.

(3) What you need then is some sort of policy that defines when, for any given timestamped event, to stop waiting for the arrival of earlier events.

This is precisely what watermarks do — they define when to stop waiting for earlier events.

Event time processing in Flink depends on watermark generators that insert special timestamped elements into the stream, called watermarks. A watermark for time t is an assertion that the stream is (probably) now complete up through time t.

When should this stream sorter stop waiting, and push out the 2 to start the sorted stream? When a watermark arrives with a timestamp of 2, or greater.

(4) You might imagine different policies for deciding how to generate watermarks.

Each event arrives after some delay, and these delays vary, so some events are delayed more than others. One simple approach is to assume that these delays are bounded by some maximum delay. Flink refers to this strategy as bounded-out-of-orderness watermarking. It is easy to imagine more complex approaches to watermarking, but for most applications a fixed delay works well enough.	

 
### Latency vs. Completeness 

Another way to think about watermarks is that they give you, the developer of a streaming application, control over the tradeoff between latency and completeness. Unlike in batch processing, where one has the luxury of being able to have complete knowledge of the input before producing any results, with streaming you must eventually stop waiting to see more of the input, and produce some sort of result.

You can either configure your watermarking aggressively, with a short bounded delay, and thereby take the risk of producing results with rather incomplete knowledge of the input – i.e., a possibly wrong result, produced quickly. Or you can wait longer, and produce results that take advantage of having more complete knowledge of the input stream(s).

It is also possible to implement hybrid solutions that produce initial results quickly, and then supply updates to those results as additional (late) data is processed. This is a good approach for some applications.

******************************************************************************************************************************
--------------------------------------
Streaming
--------------------------------------

Time Attributes

https://nightlies.apache.org/flink/flink-docs-release-2.0/docs/dev/table/concepts/time_attributes/

Flink can process data based on different notions of time.

Processing time refers to the machine’s system time (also known as epoch time, e.g. Java’s System.currentTimeMillis()) that is executing the respective operation.
Event time refers to the processing of streaming data based on timestamps that are attached to each row. The timestamps can encode when an event happened.

******************************************************************************************************************************
--------------------------------------
SQL Windowing
--------------------------------------

https://developer.confluent.io/courses/apache-flink/streaming-analytics-exercise/

This is a materializing query, meaning they keep some state indefinitely.

```
SELECT
  FLOOR(ts TO SECOND) AS window_start,
  count(url) as cnt
FROM pageviews
GROUP BY FLOOR(ts TO SECOND);
```

There is a problem with doing windowing this way, and that problem has to do with state retention. When windowing is expressed this way, Flink SQL engine doesnt know anything about the semantics of FLOOR(ts TO SECOND). We know that the result of this function is connected to time, and that the timestamps are (approximately) ordered by time.

A better approach to windowing using time attributes and table-valued functions

What Flink SQL needs in order to safely implement windowing (i.e., windows that will be cleaned up once they're no longer changing) is

- an input table that is append-only (which we have), and
- a designated timestamp column with timestamps that are known to be advancing (which we don't have (yet))
	
Flink SQL calls a timestamp column like this a time attribute, and time attributes come in two flavors: processing time (also known as wall clock time) and event time. For our use case, where we are counting page views per second, the distinction is this:

- When windowing with processing time, a page view is counted based on the second during which it is processed, rather than when it occurred.
- When windowing with event time, a page view is counted based on when it occured, rather than when it is processed.
	
When using a time attribute column based on processing time, its obvious that time will be advancing. On the other hand, working with event time is trickier: just because a table has a timestamp column does not necessarily imply that those timestamps are advancing. For example, our pageviews table could have a created_at column indicating when the page was created. The order in which pages are created has little (if anything) to do with the order in which they are viewed.

To deal with this, Flink relies something called watermarks to measure the progress of event time.

For this exercise, you will experiment with windows that use processing time. This requires adding a timestamp column that is tied to the systems time-of-day clock. This processing-time-based timestamp column is an example of a computed column, meaning a column whose value is computed, rather than being physically present in the event stream. In this case we want to use the built-in PROCTIME() function.

Flink SQL includes special operations for windowing, which we can now take advantage of. This requires setting up the query in a particular way, using one of the built-in window functions, such as TUMBLE:

```
SELECT
  window_start, count(url) AS cnt
FROM TABLE(
  TUMBLE(TABLE pageviews, DESCRIPTOR(proc_time), INTERVAL '1' SECOND))
GROUP BY window_start;
```	
	
The built-in TUMBLE function used in this query is an example of a table-valued function. This function takes three parameters

- a table descriptor (TABLE pageviews)
- a column descriptor for the time attribute (DESCRIPTOR(proc_time))
- a time interval to use for windowing (one second)

and it returns a new table based on the input table, but with two additional columns added to aid with windowing. To see how this works, try executing this part of the windowing query on its own:

```
SELECT *
FROM TABLE(TUMBLE(TABLE pageviews, DESCRIPTOR(proc_time), INTERVAL '1' SECOND));
```

What you're seeing is that the table returned by the TUMBLE function has window_start and window_end columns indicating which one-second-long window each pageview event has been assigned to.

GROUP BY window_start aggregates together all of the pageview events assigned to each window, making it easy to compute any type of aggregation function on these windows, such as counting.

https://nightlies.apache.org/flink/flink-docs-stable/docs/dev/table/sql/queries/window-tvf/

Apache Flink provides 4 built-in windowing TVFs: TUMBLE, HOP, CUMULATE and SESSION. The return value of windowing TVF is a new relation that includes all columns of original relation as well as additional 3 columns named “window_start”, “window_end”, “window_time” to indicate the assigned window. In streaming mode, the “window_time” field is a time attributes of the window. In batch mode, the “window_time” field is an attribute of type TIMESTAMP or TIMESTAMP_LTZ based on input time field type. The “window_time” field can be used in subsequent time-based operations, e.g. another windowing TVF, or interval joins, over aggregations. The value of window_time always equal to window_end - 1ms.



******************************************************************************************************************************
--------------------------------------
SQL Client Initialise
--------------------------------------

```
git clone https://github.com/confluentinc/learn-apache-flink-101-exercises.git
cd learn-apache-flink-101-exercises
docker compose up --build -d
docker compose run sql-client
docker compose down -v
```

--------------------------------------
SQL Client Demo 1
--------------------------------------

https://nightlies.apache.org/flink/flink-docs-master/docs/dev/table/sqlclient/

```
sql-client.execution.result-mode

"TABLE":     Materializes results in memory and visualizes them in a regular, paginated table representation.
"CHANGELOG": Visualizes the result stream that is produced by a continuous query.
"TABLEAU":   Display results in the screen directly in a tableau format.

The table mode materializes results in memory and visualizes them in a regular, paginated table representation. It can be enabled by executing the following command in the CLI:

SET 'sql-client.execution.result-mode' = 'table';


                           name         age isHappy        dob                         height
                          user1          20    true 1995-12-03                            1.7
                          user2          30    true 1972-08-02                           1.89
                          user3          40   false 1983-12-23                           1.63
                          user4          41    true 1977-11-13                           1.72
                          user5          22   false 1998-02-20                           1.61
                          user6          12    true 1969-04-08                           1.58
                          user7          38   false 1987-12-15                            1.6
                          user8          62    true 1996-08-05                           1.82




Q Quit                     + Inc Refresh              G Goto Page                N Next Page                O Open Row
R Refresh                  - Dec Refresh              L Last Page                P Prev Page

The changelog mode does not materialize results and visualizes the result stream that is produced by a continuous query consisting of insertions (+) and retractions (-).

SET 'sql-client.execution.result-mode' = 'changelog';

 op                           name         age isHappy        dob                         height
 +I                          user1          20    true 1995-12-03                            1.7
 +I                          user2          30    true 1972-08-02                           1.89
 +I                          user3          40   false 1983-12-23                           1.63
 +I                          user4          41    true 1977-11-13                           1.72
 +I                          user5          22   false 1998-02-20                           1.61
 +I                          user6          12    true 1969-04-08                           1.58
 +I                          user7          38   false 1987-12-15                            1.6
 +I                          user8          62    true 1996-08-05                           1.82




Q Quit                                       + Inc Refresh                                O Open Row
R Refresh                                    - Dec Refresh


The tableau mode is more like a traditional way which will display the results in the screen directly with a tableau format. The displaying content will be influenced by the query execution type (execution.type)

SET 'sql-client.execution.result-mode' = 'tableau';

+----+--------------------------------+-------------+---------+------------+--------------------------------+
| op |                           name |         age | isHappy |        dob |                         height |
+----+--------------------------------+-------------+---------+------------+--------------------------------+
| +I |                          user1 |          20 |    true | 1995-12-03 |                            1.7 |
| +I |                          user2 |          30 |    true | 1972-08-02 |                           1.89 |
| +I |                          user3 |          40 |   false | 1983-12-23 |                           1.63 |
| +I |                          user4 |          41 |    true | 1977-11-13 |                           1.72 |
| +I |                          user5 |          22 |   false | 1998-02-20 |                           1.61 |
| +I |                          user6 |          12 |    true | 1969-04-08 |                           1.58 |
| +I |                          user7 |          38 |   false | 1987-12-15 |                            1.6 |
| +I |                          user8 |          62 |    true | 1996-08-05 |                           1.82 |
+----+--------------------------------+-------------+---------+------------+--------------------------------+
Received a total of 8 rows

***

```
SET 'execution.runtime-mode' = 'streaming'; -- execution mode either 'batch' or 'streaming'

Note that when you use this mode with streaming query, the result will be continuously printed on the console. If the input data of this query is bounded, the job will terminate after Flink processed all input data, and the printing will also be stopped automatically. Otherwise, if you want to terminate a running query, just type CTRL-C in this case, the job and the printing will be stopped.

All these result modes can be useful during the prototyping of SQL queries. In all these modes, results are stored in the Java heap memory of the SQL Client. In order to keep the CLI interface responsive, the changelog mode only shows the latest 1000 changes. The table mode allows for navigating through bigger results that are only limited by the available main memory and the configured maximum number of rows (sql-client.execution.max-table-result.rows).


******************************************************************************************************************************
--------------------------------------
SQL Client Demo 2
--------------------------------------

```
Flink SQL> SET 'sql-client.execution.result-mode' = 'tableau';
[INFO] Execute statement succeed.

Flink SQL> SET 'execution.runtime-mode' = 'batch';
[INFO] Execute statement succeed.

Flink SQL>
> SELECT
>   name,
>   COUNT(*) AS cnt
> FROM
>   (VALUES ('Bob'), ('Alice'), ('Greg'), ('Bob')) AS NameTable(name)
> GROUP BY name;
+-------+-----+
|  name | cnt |
+-------+-----+
| Alice |   1 |
|   Bob |   2 |
|  Greg |   1 |
+-------+-----+
3 rows in set


Flink SQL>
> SET 'execution.runtime-mode' = 'streaming';
[INFO] Execute statement succeed.

Flink SQL> SELECT
>   name,
>   COUNT(*) AS cnt
> FROM
>   (VALUES ('Bob'), ('Alice'), ('Greg'), ('Bob')) AS NameTable(name)
> GROUP BY name;
+----+--------------------------------+----------------------+
| op |                           name |                  cnt |
+----+--------------------------------+----------------------+
| +I |                            Bob |                    1 |
| +I |                          Alice |                    1 |
| +I |                           Greg |                    1 |
| -U |                            Bob |                    1 |
| +U |                            Bob |                    2 |
+----+--------------------------------+----------------------+
Received a total of 5 rows
```

******************************************************************************************************************************
--------------------------------------
SQL Client Demo 3
--------------------------------------

```
CREATE TABLE `bounded_pageviews` (
  `url` STRING,
  `user_id` STRING,
  `browser` STRING,
  `ts` TIMESTAMP(3)
)
WITH (
  'connector' = 'faker',
  'number-of-rows' = '500',
  'rows-per-second' = '100',
  'fields.url.expression' = '/#{GreekPhilosopher.name}.html',
  'fields.user_id.expression' = '#{numerify ''user_##''}',
  'fields.browser.expression' = '#{Options.option ''chrome'', ''firefox'', ''safari'')}',
  'fields.ts.expression' =  '#{date.past ''5'',''1'',''SECONDS''}'
);

CREATE TABLE `pageviews` (
  `url` STRING,
  `user_id` STRING,
  `browser` STRING,
  `ts` TIMESTAMP(3)
)
WITH (
  'connector' = 'faker',
  'rows-per-second' = '100',
  'fields.url.expression' = '/#{GreekPhilosopher.name}.html',
  'fields.user_id.expression' = '#{numerify ''user_##''}',
  'fields.browser.expression' = '#{Options.option ''chrome'', ''firefox'', ''safari'')}',
  'fields.ts.expression' =  '#{date.past ''5'',''1'',''SECONDS''}'
);

ALTER TABLE `pageviews` SET ('rows-per-second' = '10');
```

******************************************************************************************************************************
--------------------------------------
SQL Client Demo 4
--------------------------------------

```
This configuration:

- connects to Hive catalogs and uses MyCatalog as the current catalog with MyDatabase as the current database of the catalog,
- defines a table MyTable that can read data from a CSV file,
- defines a view MyCustomView that declares a virtual table using a SQL query,
- defines a user-defined function myUDF that can be instantiated using the class name,
- uses streaming mode for running statements and a parallelism of 1,
- runs exploratory queries in the table result mode,
- and makes some planner adjustments around join reordering and spilling via configuration options.

-- Define available catalogs

CREATE CATALOG MyCatalog
  WITH (
    'type' = 'hive'
  );

USE CATALOG MyCatalog;

-- Define available database

CREATE DATABASE MyDatabase;

USE MyDatabase;

-- Define TABLE

CREATE TABLE MyTable(
  MyField1 INT,
  MyField2 STRING
) WITH (
  'connector' = 'filesystem',
  'path' = '/path/to/something',
  'format' = 'csv'
);

-- Define VIEW

CREATE VIEW MyCustomView AS SELECT MyField2 FROM MyTable;

-- Define user-defined functions here.

CREATE FUNCTION myUDF AS 'foo.bar.AggregateUDF';

-- Properties that change the fundamental execution behavior of a table program.

SET 'execution.runtime-mode' = 'streaming'; -- execution mode either 'batch' or 'streaming'
SET 'sql-client.execution.result-mode' = 'table'; -- available values: 'table', 'changelog' and 'tableau'
SET 'sql-client.execution.max-table-result.rows' = '10000'; -- optional: maximum number of maintained rows
SET 'parallelism.default' = '1'; -- optional: Flink's parallelism (1 by default)
SET 'pipeline.auto-watermark-interval' = '200'; --optional: interval for periodic watermarks
SET 'pipeline.max-parallelism' = '10'; -- optional: Flink's maximum parallelism
SET 'table.exec.state.ttl' = '1000'; -- optional: table program's idle state time
SET 'restart-strategy.type' = 'fixed-delay';

-- Configuration options for adjusting and tuning table programs.

SET 'table.optimizer.join-reorder-enabled' = 'true';
SET 'table.exec.spill-compression.enabled' = 'true';
SET 'table.exec.spill-compression.block-size' = '128kb';

```

******************************************************************************************************************************

```
This configuration:

- defines a temporal table source users that reads from a CSV file,
- set the properties, e.g job name,
- set the savepoint path,
- submit a sql job that load the savepoint from the specified savepoint path.

CREATE TEMPORARY TABLE users (
  user_id BIGINT,
  user_name STRING,
  user_level STRING,
  region STRING,
  PRIMARY KEY (user_id) NOT ENFORCED
) WITH (
  'connector' = 'upsert-kafka',
  'topic' = 'users',
  'properties.bootstrap.servers' = '...',
  'key.format' = 'csv',
  'value.format' = 'avro'
);

-- set sync mode
SET 'table.dml-sync' = 'true';

-- set the job name
SET 'pipeline.name' = 'SqlJob';

-- set the queue that the job submit to
SET 'yarn.application.queue' = 'root';

-- set the job parallelism
SET 'parallelism.default' = '100';

-- restore from the specific savepoint path
SET 'execution.state-recovery.path' = '/tmp/flink-savepoints/savepoint-cca7bc-bb1e257f0dab';

INSERT INTO pageviews_enriched
SELECT *
FROM pageviews AS p
LEFT JOIN users FOR SYSTEM_TIME AS OF p.proctime AS u
ON p.user_id = u.user_id;

```

******************************************************************************************************************************

### Deploy SQL Files to an Application Cluster 

SQL Client also supports deploying a SQL script file to an Application Cluster with the -f option, if you specify the deployment target in the config.yaml or startup options. Here is an example to deploy script file in an Application Cluster.

```
./bin/sql-client.sh -f oss://path/to/script.sql \
      -Dexecution.target=kubernetes-application \
      -Dkubernetes.cluster-id=${CLUSTER_ID} \
      -Dkubernetes.container.image.ref=${FLINK_IMAGE_NAME}
```
	  
******************************************************************************************************************************

### Execute a set of SQL statements	  

SQL Client execute each INSERT INTO statement as a single Flink job. However, this is sometimes not optimal because some part of the pipeline can be reused. SQL Client supports STATEMENT SET syntax to execute a set of SQL statements.

```
CREATE TABLE pageviews (
  user_id BIGINT,
  page_id BIGINT,
  viewtime TIMESTAMP,
  proctime AS PROCTIME()
) WITH (
  'connector' = 'kafka',
  'topic' = 'pageviews',
  'properties.bootstrap.servers' = '...',
  'format' = 'avro'
);

CREATE TABLE pageview (
  page_id BIGINT,
  cnt BIGINT
) WITH (
  'connector' = 'jdbc',
  'url' = 'jdbc:mysql://localhost:3306/mydatabase',
  'table-name' = 'pageview'
);

CREATE TABLE uniqueview (
  page_id BIGINT,
  cnt BIGINT
) WITH (
  'connector' = 'jdbc',
  'url' = 'jdbc:mysql://localhost:3306/mydatabase',
  'table-name' = 'uniqueview'
);

EXECUTE STATEMENT SET
BEGIN

INSERT INTO pageview
SELECT page_id, count(1)
FROM pageviews
GROUP BY page_id;

INSERT INTO uniqueview
SELECT page_id, count(distinct user_id)
FROM pageviews
GROUP BY page_id;

END;

```
******************************************************************************************************************************

### Execute DML statements sync/async

```
By default, SQL Client executes DML statements asynchronously. That means, SQL Client will submit a job for the DML statement to a Flink cluster, and not wait for the job to finish. So SQL Client can submit multiple jobs at the same time. This is useful for streaming jobs, which are long-running in general.

Flink SQL> INSERT INTO MyTableSink SELECT * FROM MyTableSource;
[INFO] Table update statement has been successfully submitted to the cluster:
Cluster ID: StandaloneClusterId
Job ID: 6f922fe5cba87406ff23ae4a7bb79044

However, for batch users, it’s more common that the next DML statement requires waiting until the previous DML statement finishes. In order to execute DML statements synchronously, you can set table.dml-sync option to true in SQL Client.

Flink SQL> SET 'table.dml-sync' = 'true';
[INFO] Session property has been set.

Flink SQL> INSERT INTO MyTableSink SELECT * FROM MyTableSource;
[INFO] Submitting SQL update statement to the cluster...
[INFO] Execute statement in sync mode. Please wait for the execution finish...
[INFO] Complete execution of the SQL update statement.
```

******************************************************************************************************************************

### Start a SQL Job from a savepoint 

Flink supports to start the job with specified savepoint. In SQL Client, it’s allowed to use SET command to specify the path of the savepoint.

```
Flink SQL> SET 'execution.state-recovery.path' = '/tmp/flink-savepoints/savepoint-cca7bc-bb1e257f0dab';
[INFO] Session property has been set.

-- all the following DML statements will be restroed from the specified savepoint path
Flink SQL> INSERT INTO ...
When the path to savepoint is specified, Flink will try to restore the state from the savepoint when executing all the following DML statements.

Because the specified savepoint path will affect all the following DML statements, you can use RESET command to reset this config option, i.e. disable restoring from savepoint.

Flink SQL> RESET 'execution.state-recovery.path';
[INFO] Session property has been reset.
For more details about creating and managing savepoints, please refer to Job Lifecycle Management.
```

******************************************************************************************************************************

### Define a Custom Job Name 

SQL Client supports to define job name for queries and DML statements through SET command.

```
Flink SQL> SET 'pipeline.name' = 'kafka-to-hive';
[INFO] Session property has been set.

-- all the following DML statements will use the specified job name.
Flink SQL> INSERT INTO ...
Because the specified job name will affect all the following queries and DML statements, you can also use RESET command to reset this configuration, i.e. use default job names.

Flink SQL> RESET 'pipeline.name';
[INFO] Session property has been reset.
If the option pipeline.name is not specified, SQL Client will generate a default name for the submitted job, e.g. insert-into_<sink_table_name> for INSERT INTO statements.
```

******************************************************************************************************************************

### Monitoring Job Status 

SQL Client supports to list jobs status in the cluster through SHOW JOBS statements.

```
Flink SQL> SHOW JOBS;
+----------------------------------+---------------+----------+-------------------------+
|                           job id |      job name |   status |              start time |
+----------------------------------+---------------+----------+-------------------------+
| 228d70913eab60dda85c5e7f78b5782c | kafka-to-hive |  RUNNING | 2023-02-11T05:03:51.523 |
+----------------------------------+---------------+----------+-------------------------+
```

******************************************************************************************************************************

### Terminating a Job 

SQL Client supports to stop jobs with or without savepoints through STOP JOB statements.

```
Flink SQL> STOP JOB '228d70913eab60dda85c5e7f78b5782c' WITH SAVEPOINT;
+-----------------------------------------+
|                          savepoint path |
+-----------------------------------------+
| file:/tmp/savepoint-3addd4-0b224d9311e6 |
+-----------------------------------------+
The savepoint path could be specified with execution.checkpointing.savepoint-dir either in the cluster configuration or session configuration (the latter would take precedence).
```

******************************************************************************************************************************

Explore the GUI

```
select count(*) from pageviews

http://localhost:8081

EXPLAIN select count(*) from pageviews
```
