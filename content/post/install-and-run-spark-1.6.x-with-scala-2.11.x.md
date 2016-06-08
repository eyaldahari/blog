# How to install Apache Spark 1.6.X with Scala 2.11.X in no time

At the time this lines are written, there is no official installation for Spark 1.6.X and Scala 2.11.X.  
In this post I'll explain in few simple steps how to do so.

Lest start!

Prerequisites:  

- A Linux flavor of your choice 
- Java 7+
- When building on an encrypted filesystem refer to [building-spark](http://spark.apache.org/docs/latest/building-spark.html#building-with-buildmvn)

Steps:  

- Download Spark sources:  

```bash
wget https://github.com/apache/spark/archive/branch-1.6.zip
```

- Run the following commands to build Spark locally (pay attention to the versions):  

```bash
unzip branch-1.6.zip
cd spark-branch-1.6  
./dev/change-scala-version.sh 2.11
./build/mvn -Pyarn -Phadoop-2.6 -Dscala-2.11 -DskipTests clean package
```
  
After maven build finishes successfully, you should have a working Spark installation package!
To verify run:

```bash
./bin/run-example SparkPi 10
```

If you get network errors you may set environment at conf/spark-env.sh.template (don't forget to remove the .template suffix)
 or simply run:

```bash
export SPARK_LOCAL_IP=127.0.0.1
```

Now you may run the Spark shell:
```bash
./bin/spark-shell --master local[2]
```
 
Hope that helps and enjoy the ride! :)


