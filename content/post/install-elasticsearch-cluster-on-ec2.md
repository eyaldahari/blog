# Elasticsearch nodes discovery on Amazon EC2

Elasticsearch cluster is built from multiple nodes. Usually each node is located on a different machine. 
When a node is loaded in the cluster it reads it’s configuration and tries to discover the other nodes.
Elasticsearch comes with a default nodes discovery protocol called [Zen Discovery](https://www.elastic.co/guide/en/elasticsearch/reference/current/modules-discovery-zen.html).

There are more specialized discovery plugins for Elasticsearch cluster which runs on different cloud providers 
like [Microsoft Azure](https://www.elastic.co/guide/en/elasticsearch/reference/current/modules-discovery-azure.html), [Google Compute Engine](https://www.elastic.co/guide/en/elasticsearch/reference/current/modules-discovery-gce.html) and [Amazon EC2](https://www.elastic.co/guide/en/elasticsearch/reference/current/modules-discovery-ec2.html).

Here I am going to explain on how to use [Amazon EC2](https://www.elastic.co/guide/en/elasticsearch/reference/current/modules-discovery-ec2.html) discovery plugin.

## Prerequisites:
1. Install Elasticsearch on each node on the cluster
2. Install AWS cloud plugin:
   * Make sure that you have an internet connection open
   * Run the following command to install the plugin:    

```bash
sudo bin/plugin install cloud-aws
```    

## Node Configuration
Configure the relevant settings in elasticsearch.yaml. Here are some basic settings to start with:

```yaml
cluster.name: <your-cluster-name>
node.name: ${HOSTNAME}
bootstrap.mlockall: true
network.host: _ec2_

cloud:
      aws:
          access_key: <your mamazon access key>
          secret_key: <your mamazon secret key>

          protocol: http
          
          s3:
             protocol: https
          
          ec2:
             protocol: http

region: us-west-2

discovery:
          type: ec2
```

You may refer [here](https://www.elastic.co/guide/en/elasticsearch/plugins/2.3/cloud-aws-discovery.html) for more configuration options.

## EC2 permissions
EC2 discovery requires making a call to the EC2 service. Setup an IAM policy to allow this.
You can create a custom policy via the IAM Management Console. 
If you are logged in to your Amazon account, the following [link](https://console.aws.amazon.com/iam/home?region=us-west-2#policies) should take you to the policies page 
on which you'll need to select _"create your own policy"_ like in the image below. 
Pay attention that you are on the correct region. The link I use works on us-west-2.

![Amazon policy page](/static/img/amazon-policy.png)

Give the policy a name and paste the following block in the _"Policy Document"_ field

```json
{
  "Statement": [
    {
      "Action": [
        "ec2:DescribeInstances"
      ],
      "Effect": "Allow",
      "Resource": [
        "*"
      ]
    }
  ],
  "Version": "2012-10-17"
}
```

Click on _"Create Policy"_

## Start Elasticsearch cluster and verify nodes
Now you are ready to start all the nodes in the EC2 cluster.
In order to validate that the nodes are discoverable, run the query below in your browser.
You should see the number of nodes at the *"number_of_nodes"* json response field.

```bash
#request:
http://<your amazon instance ip/name>:9200/_cluster/health

#example response
{
   "cluster_name": "your-cluster-name",
   "status": "green",
   "timed_out": false,
   "number_of_nodes": 2,
   "number_of_data_nodes": 2,
   "active_primary_shards": 0,
   "active_shards": 0,
   "relocating_shards": 0,
   "initializing_shards": 0,
   "unassigned_shards": 0,
   "delayed_unassigned_shards": 0,
   "number_of_pending_tasks": 0,
   "number_of_in_flight_fetch": 0,
   "task_max_waiting_in_queue_millis": 0,
   "active_shards_percent_as_number": 100
}
```

Hope that helps :)
Follow me on:

[Twitter](https://twitter.com/EyalDahari) | [Linkedin](https://twitter.com/EyalDahari) | [Stackoverflow](http://stackexchange.com/users/7651751/e-dahari?tab=activity) | [GitHub](https://github.com/eyaldahari)