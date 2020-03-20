# 301 Spark basics

Module 1, Big Data course (81932), University of Bologna.

## 301-1 Launching the shell or submitting jobs

Apache Spark admits two ways of running applications: interactive and batch.

Interactive applications can be written using the *shell*. 
No need to use an IDE, just write and execute jobs. 
Most suitable for exploratory activities and live demos.

- Launch with ```spark-shell``` to run Spark 1
- Launch with ```spark2-shell``` to run Spark 2 (cluster only)
- Further parameters can be used to force a certain deployment configuration (e.g., the number of executors); to be discussed in 302.

Batch applications can be submitted by invoking the *spark-submit* program. 
Write applications using an IDE (e.g., Intellij), compile and submit the jar. 
Most suitable for production jobs.

- Submit with ```spark-submit <jarFile>``` to run on Spark 1
- Submit with ```spark2-submit <jarFile>``` to run on Spark 2
- Same considerations for further parameters apply here

*NOTE*: Spark libraries are heavy; use the fat JAR *only if* you need some library that is not in the cluster/VM.

### Example Weather

Goal: calculate the average temperature for every month; dataset is ```weather-sample```.

- Via shell
  - Get the code for file ```example-weather.scala```
- Via submit
  - Spark 2 (cluster only)
    - Compile with ```./gradlew```
    - Copy the JAR to your local folder on the cluster
    - Submit with ```spark2-submit --class ExampleWeather2 BD-301-spark-basics.jar```
  - Spark 1
    - Comment Spark 2 libraries in ```build.gradle``` and uncomment Spark 1 libraries
    - Comment the code in ```src/main/scala/ExampleWeather2.scala``` and uncomment 
    the code in ```src/main/scala/ExampleWeather1.scala```
    - Compile with ```./gradlew```
    - Copy the JAR to your local folder on the cluster/VM
    - Submit with ```spark-submit --class ExampleWeather1 BD-301-spark-basics.jar```

## 301-2 Spark warm-up

Launch the Spark shell and load the ```capra``` and ```divinacommedia``` datasets.

```
val rddCapra = sc.textFile("hdfs:/bigdata/dataset/capra/capra.txt")
val rddDC = sc.textFile("hdfs:/bigdata/dataset/divinacommedia")
```

Try the following actions:
- Show their content (```collect```)
- Count their rows (```count```)
- Split phrases into words (```map``` or ```flatMap```; what’s the difference?)
- Check the results (remember: evaluation is lazy)

## 301-3 From MapReduce to Spark

Reproduce on Spark the exercises done on Hadoop MapReduce on the capra and divinacommedia datasets.

- Jobs:
  - Count the number of occurrences of each word
    - Result: (sopra, 1), (la, 4), …
    ```rddDC.flatMap(_.split(' ')).filter(!_.isEmpty).filter(p => p != ",").map(_.toLowerCase).map(v => (v, 1)).reduceByKey(_+_).sortBy(_._2, false).collect```
  - Count the number of occurrences of words of given lengths
    - Result: (2, 4), (5, 8)
    ```rddDC.flatMap(_.split(' ')).filter(!_.isEmpty).filter(p => p != ",").map(_.toLowerCase).map(v => (v.length, 1)).reduceByKey(_+_).sortBy(_._2, false).collect```
  - Count the average length of words given their first letter (hint: check the example in 301-1)
    - Result: (s, 5), (l, 2), …
    ```rddDC.flatMap(_.split(' ')).filter(!_.isEmpty).filter(p => p != ",").map(_.toLowerCase).map(v => (v.substring(0,1), (v.length, 1))).reduceByKey((a,b) => (a._1+b._1,a._2+b._2)).mapValues(v => v._1 / v._2).sortBy(_._2, false).collect```
  - Return the inverted index of words
    - Result: (sopra, (0)), (la, (0, 1)), … 
    ```rddDC.zipWithIndex.map(v => (v._1.split(' ').filter(!_.isEmpty).filter(p => p != ","), v._2)).flatMap(v => v._1.map(a => (a,v._2))).groupByKey.sortBy(_._2.size, false).collect```
- How does Spark compare with respect to MapReduce? (performance, ease of use)
- How is the output sorted? How can you sort by value?
