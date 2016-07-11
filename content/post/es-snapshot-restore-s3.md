#Elasticsearch snapshot and restore

Elasticsearch snapshot and restore API's allows to create snapshots of individual indices or an entire cluster into a remote repository. 
The API's allows to take a snapshot and save it to many repository types like file system, shared UNC paths, Amazon S3 (and other cloud providers), HDFS and etc.

In this post I will briefly explain how to take a cluster snapshot running on one machine and restore it on another. I will focus on how to 
take a snapshot specifically on Amazon EC2 instance using Amazon S3 as a repository and restore it on another Amazon EC2 instance.

I deliberately keep it as simple as possible and if you wish to have more advanced options, you can always refer to Elasticsearch documentation 
[here](https://www.elastic.co/guide/en/elasticsearch/reference/current/modules-snapshots.html) and [here](https://www.elastic.co/guide/en/elasticsearch/plugins/current/cloud-aws-repository.html)


##Prerequisites
- In order to continue I assume that you have at least two running Amazon EC2 instances
- On each instance you already have Elasticsearch installed
- On each instance you need to install `cloud-aws` plugin. Here is how to install it:

```bash
  #Under your Elasticsearch installation usually under /usr/share/elasticsearch run
  sudo bin/plugin install cloud-aws
    
  #You'll need to restart elasticsearch (make sure you know how to do it without harming your cluster) 
  service elasticsearch restart
```

- For elaborated information on how to have the prerequisites done, you may refer to the following [article](https://medium.com/@eyaldahari/set-nodes-discovery-for-elasticsearch-cluster-on-amazon-ec2-6a3ae673645c#.5cs8pr2c4)

##REST APIs for cluster snapshot to S3 and restore
Define a the snapshot configuration in Elasticsearch
```bash
  #Set snapshot definitions. Refer to Elasticsearch documentation for advanced options
  PUT _snapshot/my_snapshot
  {
    "type": "s3",
    "settings": {
      "bucket": "your_predifined_s3_bucket",
      "region": "us-west-1",
      "base_path": "path_under_s3_bucket",
      "access_key": "your_amazon_s3_accesskey",
      "secret_key": "your_amazon_s3_secretkey"
    }
  }
```

Take a snapshot and give it a name (ex. snapshot_1)

```bash
  #Run the snapshot process
  PUT /_snapshot/my_snapshot/snapshot_1?wait_for_completion=true
```

Get snapshot process status (it may take time to complete the operation)
```bash
  #Get snapshot status
  GET /_snapshot/my_snapshot/_status
```

Validate the snapshot
```bash
  #Validate snapshot
  POST /_snapshot/my_snapshot/_verify
```

##Reload the snapshot on another machine
Define the same snapshot configuration as above:
```bash
  #In order to reload cluster at another machine
  PUT _snapshot/my_snapshot
  {
    "type": "s3",
    "settings": {
      "bucket": "your_predifined_s3_bucket",
      "region": "us-west-1",
      "base_path": "path_under_s3_bucket",
      "access_key": "your_amazon_s3_accesskey",
      "secret_key": "your_amazon_s3_secretkey"
    }
  }
```

Run the restore process on the second cluster
```bash
  #Run restore
  POST /_snapshot/my_snapshot/snapshot_1/_restore
```

Run validations as above on the second cluster


Please note that you can have control on many parameters like the indexes to be snapshot/restored, index metadata, the snapshot rate, bulk sizes and 
many more parameters. 
All are explained well in the following [Elasticsearch documentation](https://www.elastic.co/guide/en/elasticsearch/reference/current/modules-snapshots.html)


Hope that this post helps :)

Follow me on:

[Medium](https://medium.com/@eyaldahari) | [Twitter](https://twitter.com/EyalDahari) | [Linkedin](https://www.linkedin.com/in/eyaldahari) | [Stackoverflow](http://stackexchange.com/users/7651751/e-dahari?tab=activity) | [GitHub](https://github.com/eyaldahari)




