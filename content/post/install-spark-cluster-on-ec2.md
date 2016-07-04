
# How to set Apache Spark cluster on Amazon EC2 in no time

So you have your Apache Spark [up and running](https://t.co/wFWF0F5AoG) on a local node. 
Now you want to get more advance and enjoy the real power of Spark computation on your Amazon EC2. 
You can do it manually and it may take you some time, but how about starting stopping and generally managing the cluster?

This is not a simple job, right? 

For this reason (and many more) Apache Spark package is bundled with an automation scripts for EC2 to make our lives easier.

Lets start!

The following instructions and all relevant details about parameters etc. are taken from: http://spark.apache.org/docs/latest/ec2-scripts.html 

>Before You Start
 Create an Amazon EC2 key pair for yourself. This can be done by logging into your Amazon Web Services account through the AWS console, clicking Key Pairs on the left sidebar, and creating and downloading a key. Make sure that you set the permissions for the private key file to 600 (i.e. only you can read and write it) so that ssh will work.
 Whenever you want to use the spark-ec2 script, set the environment variables AWS_ACCESS_KEY_ID and AWS_SECRET_ACCESS_KEY to your Amazon EC2 access key ID and secret access key. These can be obtained from the AWS homepage by clicking Account > Security Credentials > Access Credentials.

Under your Spark installation find ec2 directory and go there. If you can't find this directory you may download it from [here](https://github.com/apache/spark/tree/branch-1.6/ec2)

Use your private key file and security keys to run:

```bash
chmod 400 /root/.ssh/eyald-oregon.sh
export AWS_SECRET_ACCESS_KEY=yoursecret
export AWS_ACCESS_KEY_ID=yourkey

./ec2/spark-ec2 --slaves=2 --region=us-west-2 --zone=us-west-2b --hadoop-major-version=2 --key-pair=eyald-oregon --identity-file=/root/.ssh/eyald-oregon.sh launch spark-ed-cluster-1
```

It may take a while to finish. At the end when looking at your EC2 instances you may see, 
 according to the arguments in the the command above, one master and two workers EC2 Spark instances
 
### How to add/remove a worker/slave node from the cluster?
It appears like there is no good solution for this requirements using the spark-ec2 scripts.
According to [this](http://apache-spark-user-list.1001560.n3.nabble.com/Having-spark-ec2-join-new-slaves-to-existing-cluster-td3783.html) mail thread and [this](https://issues.apache.org/jira/browse/SPARK-2008) Jira closed issue, it is not going to be automatically supported.
Nonetheless, you can go about this as described in the mail thread above as follows (continue the example above):

- Stop the master node:
```bash
./ec2/spark-ec2 --region=us-west-2 stop spark-ed-cluster-1
```
- Add a new Amazone instance to your cluster by using the same security group as the other slaves in the cluster
- Start the cluster again. The new slave/s will be auto discovered by the spark-ec2 scripts:
```bash
 ./ec2/spark-ec2 -i /root/.ssh/eyald-oregon.sh --region=us-west-2 start spark-ed-cluster-1
```
- To destroy the cluster completely run:
```bash
./ec2/spark-ec2 --region=us-west-2 destroy spark-ed-cluster-1
```
 
For more advanced command arguments you may read the documentation at: http://spark.apache.org/docs/latest/ec2-scripts.html

Hope that helps and enjoy :)
