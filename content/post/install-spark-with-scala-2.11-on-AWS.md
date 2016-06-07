
Currently, there is no official installation for Spark 1.6.X and Scala 2.11.X
Moreover, if you want to install a Spark cluster on #Amazon EC2 manually it may take you a while
not to mention the overhead of starting stopping and generally managing the cluster.

In this blog post I explain in few simple steps how to do so.

Lest start!

Prerequisites:  

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
  
You should now have a working Spark installation package!
To verify run:

```bash
./bin/run-example SparkPi 10
```

If you get network errors you may set environment at conf/spark-env.sh.template (don't forget to remove the .template suffix)
 or simply run:

```bash
export SPARK_LOCAL_IP=127.0.0.1
```

- Now build a cluster on your EC2
The following instructions and all relevant details about parameters etc. are taken from: http://spark.apache.org/docs/latest/ec2-scripts.html 

>Before You Start
 Create an Amazon EC2 key pair for yourself. This can be done by logging into your Amazon Web Services account through the AWS console, clicking Key Pairs on the left sidebar, and creating and downloading a key. Make sure that you set the permissions for the private key file to 600 (i.e. only you can read and write it) so that ssh will work.
 Whenever you want to use the spark-ec2 script, set the environment variables AWS_ACCESS_KEY_ID and AWS_SECRET_ACCESS_KEY to your Amazon EC2 access key ID and secret access key. These can be obtained from the AWS homepage by clicking Account > Security Credentials > Access Credentials.

- Now that you have your private key file and security keys at hand run:

```bash
chmod 400 /root/.ssh/eyald-oregon.sh
export AWS_SECRET_ACCESS_KEY=yoursecret
export AWS_ACCESS_KEY_ID=yourkey

# read the documentation at (http://spark.apache.org/docs/latest/ec2-scripts.html) for more advanced command arguments
./ec2/spark-ec2 --slaves=2 --region=us-west-2 --zone=us-west-2b --hadoop-major-version=2 --key-pair=eyald-oregon --identity-file=/root/.ssh/eyald-oregon.sh launch spark-ed-cluster-1
```

- It may take a while to finish. At the end when looking at your EC2 instances you may see, 
 according to the arguments in the the command above, one master and two workers EC2 Spark instances

Hope that helps and enjoy :)