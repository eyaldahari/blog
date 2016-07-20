#How to add new EBS volume to AWS EC2 Linux instance

Now you have clicked 'Launch Instance' and run through all tedious stages of creating a new machine on AWS EC2, YaY!
But, alas! the default instance storage is only 8GB in size. What to do? go though all the pain of defining 'Instance Type', 'Security Goups', 'Network VPC', 'Subnet', 'Network Interfaces', shell I continue?
Well, don't panic, Amazon Web Services provide you with 'Volumes'.

Volumes are like phizical hard drives on your PC but they are virtual and you can add, remove, take snapshots of them and more. 
Just like you do on your personal computer. There are many types of Volumes and I wont get into that in this post.

Assumptions:
-you have already got an instance of EC2
