Set nodes discovery for Elasticsearch cluster on Amazon EC2 Elasticsearch cluster is built from multiple nodes. Usually each node is located on a different machine. When a node is loaded in the cluster it reads it’s configuration and tries to discover the other nodes. Elasticsearch comes with a default nodes discovery protocol called Zen Discovery. 

There are more specialized discovery plugins for Elasticsearch cluster which runs on different cloud providers like Microsoft Azure, Google Compute Engine and Amazon EC2.

Here I am going to explain on how to use Amazon EC2 discovery plugin.

Prerequisites: Install Elasticsearch on each node on the cluster
Install AWS cloud plugin:

Make sure that you have an internet connection open
Run the following command to install the plugin:

sudo bin/plugin install cloud-aws

Node Configuration Configure the relevant settings in elasticsearch.yaml. Here are some basic settings to start with:

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

You may refer here for more configuration options.

EC2 permissions EC2 discovery requires making a call to the EC2 service. Setup an IAM policy to allow this. You can create a custom policy via the IAM Management Console. If you are logged in to your Amazon account, the following link should take you to the policies page (pay attention that you are on the correct region. the link I use work on us-west-2) on which you’ll need to select create your own policy like in the image below. 







Give the policy a name and paste the following block in the Policy Document field

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

Click on “Create Policy”.

Start Elasticsearch cluster and verify nodes Now you are ready to start all the  nodes in the EC2 cluster.

In order to validate that the nodes are discoverable, run the query below in your browser. You should see the number of nodes at the “number_of_nodes” json response field.

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

Hope that helps :)

Follow me on Twitter | Linkedin | Stackoverflow | GitHub