# Spark Benchmark Suite (SparkBench) #
## SparkBench - the micro-benchmark suite for Spark ##

---

- Current version: 0.4
- Release data: TBD
- Contact: [Lv Qi](mailto:qi.lv@intel.com), [Grace Huang](mailto:jie.huang@intel.com)
- Homepage: https://github.com/intel-bigdata/Sparkbench

- Contents:
  1. Overview
  2. Getting Started
  3. Running

---
### OVERVIEW ###

This benchmark suite contains 9 typical micro workloads:

  **Spark Core:**

1. Sort (sort)

    This workload sorts its *text* input data, which is generated
    using RandomTextWriter.

2. WordCount (wordcount)

    This workload counts the occurrence of each word in the input
    data, which are generated using RandomTextWriter. It is
    representative of another typical class of real world MapReduce
    jobs - extracting a small amount of interesting data from large
    data set.

3. Sleep (sleep)

    This workload sleep an amount of seconds in each task to verify
    task scheduler.

  **Spark SQL:**

4. Scan (scan)

    Measure the throughput of the SparkSql cluster by query a large
    table and write the result back to HDFS:

    `FROM rankings SELECT *`

5. Join (join)

    Join two large tables with some `WHERE` conditions, `GROUP BY` and
    `ORDER BY` operations:
   
    `SELECT sourceIP, sum(adRevenue) as totalRevenue, avg(pageRank)
    FROM rankings R JOIN (SELECT sourceIP, destURL, adRevenue FROM
    uservisits UV WHERE (datediff(UV.visitDate, '1999-01-01')>=0 AND
    datediff(UV.visitDate, '2000-01-01')<=0)) NUV ON (R.pageURL =
    NUV.destURL) group by sourceIP order by totalRevenue DESC limit 1`

6. Aggregate (aggregation)

   Query large table with `SUM` and `GROUP BY` operations:

   `SELECT sourceIP, SUM(adRevenue) FROM uservisits GROUP BY sourceIP`

  **Spark Graphx:**

7. PageRank (pagerank)

    This workload benchmarks PageRank algorithm implemented in
    Spark-MLLib examples. The data source is generated from Web data
    whose hyperlinks follow the Zipfian distribution.

  **Spark MLLib:**

8. Bayesian Classification (bayes)

    This workload benchmarks NaiveBayesian Classification implemented
    in Spark-MLLib examples.

    Large-scale machine learning is another important use of
    MapReduce. This workload tests the Naive Bayesian (a popular
    classification algorithm for knowledge discovery and data mining)
    trainer in Mahout 0.7, which is an open source (Apache project)
    machine learning library. The workload uses the automatically
    generated documents whose words follow the zipfian
    distribution. The dict used for text generation is also from the
    default linux file /usr/share/dict/linux.words.

9. K-means clustering (kmeans)

    This workload tests the K-means (a well-known clustering algorithm
    for knowledge discovery and data mining) clustering in Mahout
    0.7. The input data set is generated by GenKMeansDataset based on
    Uniform Distribution and Guassian Distribution.

---
### Getting Started ###

2. Prerequisites

  1. Setup JDK-1.6(suggested)

      Download Oracle-JDK-1.6 and setup properly.

      Note: Due to
      [SPARK-1703](https://issues.apache.org/jira/browse/SPARK-1703)
      and
      [SPARK-1911](https://issues.apache.org/jira/browse/SPARK-1911),
      Oracle-JDK-1.6 is the suggested option which passes all tests.

      Other JDK versions such as Oracle-JDK-1.7 and Oracle-JDK-1.8 are
      also supported if python related workloads can be discarded. 

  2. Setup HiBench-3.0

      Download/Checkout HiBench-3.0 benchmark suite from
      [https://github.com/intel-hadoop/HiBench](https://github.com/intel-hadoop/HiBench).
      Download packages depended by hibench:

            cd HiBench/common/hibench
            mvn process-sources

      Recompile DataTools for HiBench with JDK1.6:

            cd HiBench/common/autogen
            ant package

  3. Setup Hadoop

      Before you run any workload in the package, please verify the
      Hadoop framework is running correctly. Both Hadoop 1.x and
      Hadoop-YARN are supported & tested. Currently, the suggested
      version is Hadoop 1.0.4 which is heavily tested.

  4. Setup Spark

      Download/Checkout spark from
      [https://github.com/apache/spark](https://github.com/apache/spark).
      Use spark 1.1.1 or later versions.

      For hadoop 1.0.4, standalone mode only:
      
      `./make-distribution.sh --name spark --tgz -Phive`

      For hadoop with yarn support:

      `./make-distribution.sh --name spark-yarn --tgz -Phive -Pyarn -Dhadoop.version=2.x.x -Dyarn.version=2.x.x`
      
      Please refer to `Known Issues` to set `conf/spark-default.conf`
      properly.

  5. Setup HiBench

      Download/checkout HiBench benchmark suite from
      [https://github.com/Intel-bigdata/Sparkbench/tree/v4.0-branch](https://github.com/Intel-bigdata/Sparkbench/archive/v4.0-branch.zip)

  6. Setup `numpy` in all nodes for Python related MLLib workloads. (numpy version > 1.4)

     For CentOS(6.2+):
     
     `yum inlstall numpy`

     For Ubuntu/Debian:

     `aptitude install python-numpy`

  7. Setup for HiBench/report_gen_plot.py (Optional)
  
     Install python-matplotlib with verion of 0.9+

     For CentOS(6.2+):
     
     `yum inlstall python-matplotlib`

     For Ubuntu/Debian:

     `aptitude install python-matplotlib`

2. Configure

     Quick start for minimum requirements: edit
     `conf/99-user_defined_properties.conf`, make sure below
     properties has been set:

  	  hibench.hadoop.home      The Hadoop installation location
	  hibench.spark.home       The Spark installation location
	  hibench.hdfs.master      HDFS master
	  hibench.spark.master     SPARK master
	  
     Note: For YARN mode, set `hibench.spark.master` to `yarn-client`.

     Parallelism, memory, exector number tuning:
     
          hibench.default.map.parallelism	Mapper numbers in MR, 
                                                partition numbers in Spark
          hibench.default.shuffle.parallelism   Reducer numbers in MR, shuffle 
                                                partition numbers in Spark
          hibench.yarn.exectors.num             Number executors in YARN mode
          hibench.yarn.exectors.cores           Number executor cores in YARN mode 
          hibench.yarn.exectors.memory          Number of executor memory in YARN mode
          spark.executor.memory                 Spark executor memory
          
     Compress options:

          hibench.compress.profile              Compression option `enable` or `disable`
          hibench.compress.codec.profile        Compression codec, `snappy`, `lzo` or `default`
     
     Data scale profile selection:

          hibench.scale.profile                 Data scale profile, `tiny`, `small`, `large`
                                                  
     You can add more data scale profiles in
     `conf/10-data-scale-profile.conf`. And please don't change
     `conf/00-default-properties.conf` if you have no confidence.

3. Configure parsing sequence and rules:

     1. All configurations will be loaded in a nested folder structure:

           - conf/*.conf                                         => Configure globally
           - workloads/<workload>/conf/*.conf                    => Configure for each workload
           - workloads/<workload>/<language APIs>/.../*.conf     => Configure for various languages

     2. For configurations in same folder, the loading sequence will be
     sorted according to configure file name. 

     3. Values in latter configures will override former.

     4. The final values for all properties will be stored in a single
     config file located at `report/<workload><language APIs>/conf/<workload>.conf`,
     which contain all values and pinpoint the source of the configures.

     5. All `spark.*` properties will be passed to Spark runtime configuration.

3. Configure each workload

    You can add a new data scale profile in
    `conf/10-data-scale-profile.conf`. Or edit
    `workloads/<workload>/conf/00-<workload>-default.conf` for each
    workload directly. However, latter approach will broke data scale
    profile selection mechanism, which is not suggested.

4. Synchronize the time on all nodes (optional but highly recommended)

5. Build

    Run `mvn clean package` in `src` folder:

          cd <HiBench-4.0 root>/src
          mvn clean package
          
---
### Running ###

- Run several workloads sequentially    (FIXME: not working currently)

  The `conf/benchmarks.lst` file under the package folder defines the
  workloads to run when you execute the `bin/run-all.sh` script under
  the package folder. Each line in the list file specifies one
  workload. You can use `#` at the beginning of each line to skip the
  corresponding bench if necessary.

- Run each workload separately

  You can also run each workload separately. In general, there are 3
  different files under one workload folder.

      prepare/prepare.sh            Generate or copy the job input 
                                    data into HDFS.
      mapreduce/bin/run.sh          run MapReduce language API
      spark/java/bin/run.sh         run Spark/java language API
      spark/scala/bin/run.sh        run Spark/scala language API
      spark.python/bin/run.sh       run Spark/python language API
      

  Follow the steps below to run a workload:

  1. Configure the benchmark:

      Set or override properties values for hibench and spark by
      modifying `<workload>/conf/*.conf` and `<workload>/<language
      APIs>/*.conf if necessary.

  2. Prepare data:

      `prepare/prepare.sh` to prepare input data in HDFS for running the
      benchmark.

  3. Run the benchmark:

      `<workload>/<language APIs>/bin/run.sh` to run the corresponding benchmark.

  4. Plot the report:
      
      `<HiBench root>/bin/report_gen_plot.py` to generate report figures.

      Note:
      
        `report_gen_plot.py` requires `python2.x` and `python-matplotlib`.

---
### Known issues ###

    
