# AWS-SysOps-Docs
### Monitoring & Metrics
#### Virtualization Types
HVM AMIs (Hardware virtual machine):
- Can use special hardware extensions
- Can use PV drivers for network and storage
- Usually the same or better performance than PV alone

PV AMIs (Paravirtual):
- Historically faster than HVM, but no longer the case

#### Demonstrate Ability to Monitor Availability and Performance
#### 1.Understanding AWS Instance Types, Utilization, and Performance
Instance Types – General Purpose
- T2 instances:
  - Intended for work loads that do not use the full CPU often or consistently
  - Provide Burstable Performance
  - EBS-only storage
- M3 instances:
  - Provide a balance of compute, memory, and network resources
  - SSD Storage (Instance store)
- M4 instances:
  - Provide a balance of compute, memory, and network resources
  - Support Enhanced Networking
  - EBS-optimized

Instance Types – Compute Optimized
- Lowest price/compute performance in EC2
- C3 instances
  - SSD-backed instance storage
  - Support for Enhanced Networking and Clustering
- C4 instances
  - Latest generation of Compute-optimized instances
  - Highest performing processors (optimized specifically for EC2)
  - Support for Enhanced Networking and Clustering
  - EBS-optimized

Instance Types – Memory Optimized
- Lowest price per amount (GiB of RAM) and memory performance
- R3 instances
  - SSD-backed instance storage
  - High memory capacity
  - Support for Enhanced Networking

Instance Types – GPU
- Graphics and general purpose GPU compute
- G2 instances
  - High frequency processors
  - High-performance NVIDIA GPUs
  - On-board hardware video encoder
  - Low-latency frame capture and encoding, enabling interactive streaming
  - Useful for GPU compute workloads, machine learning, video encoding, 3D   
application streaming, etc…

Instance Types – Storage Optimized
- Very fast SSD-backed instance storage optimized for high random I/O performance and high IOPS (I/O operations per second)
- I2 instances
  - High I/O performance (including high random performance)
  - High frequency processors
  - SSD storage
  - Supports TRIM
  - Supports Enhanced Networking

Instance Types – Burstable Performance
- CPU Credits are used to “burst” past the baseline performance up to 100% of a  CPU core

#### 2.EC2 Instance & System Status Checks
- System Status Checks
  - Loss of network connectivity
  - Loss of system power
  - Software issues on the physical host
  - Hardware issues on the physical host
- Solution:
  - Stop and start instances
  - Terminate and re-launch instances
  - Contact AWS

- Instance Status Checks
  - Failed system status checks
  - Incorrect networking or startup configuration
  - Exhausted memory
  - Corrupted file system
  - Incompatible kernel
- Solution:
  - Solve what is causing the issue
  - Stop and start instances
  - Terminate and re-launch instances with more memory, a different kernel, or different networking configuration

#### 3.Creating CloudWatch Alarms

#### 4.Installing and Configuring Monitoring Scripts for Amazon EC2 Instances
- [ Links used:-sudo yum install perl-Switch perl-DateTime perl-Sys-Syslog
  perl-LWP-Protocol-https
 -curl http://aws-cloudwatch.s3.amazonaws.com/downloads/CloudWatchMonitoringScripts-1.2.1.zip  -O    
 -unzip CloudWatchMonitoringScripts-1.2.1.zip
 -./mon-put-instance-data.pl --mem-util --mem-used --mem-avail --swap-util --swap-used --disk-space-util --disk-space-used --disk-space-avail --memory-units=megabytes --disk-space-units=gigabytes --disk-path=/dev/xvda1  ]  

- Create a IAM Role “cloudwatch-ec2-metrics” - “Amazon EC2” Role type -
  “CloudWatchFullAccess” Policy - Create.
- Launch an EC2 Instance with Default VPC & "cloudwatch-ec2-metrics” IAM
  Role - Review & launch - launch - create New KeyPair "cloudwatch-ec2-test” - launch.
- Goto CLI
```sh
$cd downloads/                /* where Key pair file was downloaded
$chmod 400 cloudwatch-ec2-test.pem
$ssh into EC2 instance by using “connect” option which gives ssh command
[ec2…….]$sudo yum install perl-Switch perl-DateTime perl-Sys-Syslog perl-LWP-Protocol-https    /* to install some scripts we need “perl” software,so install it by using above command of Amazon linux AMI
[ec2…….]$curl http://aws-cloudwatch.s3.amazonaws.com/downloads/CloudWatchMonitoringScripts-1.2.1.zip  -O   /*to download scripts zip file
[ec2…..]$unzip CloudWatchMonitoringScripts-1.2.1.zip /*to unzip above file
[ec2…..]$cd aws-scripts-mon/          
[ec2…..aws-scripts-mon]$ls /*lists creds template,put,get,.pm,txt files.
[ec2…..aws-scripts-mon]$./mon-put-instance-data.pl --mem-util --mem-used --mem-avail --swap-util --swap-used --disk-space-util --disk-space-used --disk-space-avail --memory-units=megabytes --disk-space-units=gigabytes --disk-path=/dev/xvda1  /*telling the scripts to monitor these metrics
[ec2…..aws-scripts-mon]$crontab -e /* to monitor metrics every 5mins
*/5 * * * * ~aws-scripts-mon/mon-put-instance-data.pl --mem-util --mem-used --mem-avail --swap-util --swap-used --disk-space-util --disk-space-used --disk-space-avail --memory-units=megabytes --disk-space-units=gigabytes --disk-path=/dev/xvda1   
:wq!  /* [ */5 * * * * means every 5mins every hour,every day,every week, every month going to execute this cron job]
[ec2…..aws-scripts-mon]$sudo tail -f  /var/log/cron /*to check cron expression is executing every 5mins
```
- Goto AWS Console, CloudWatch - Metrics - All Metrics - “Linux System metrics” will be created.If we check that, we can see created instance-ids with different metric names, can get respective graphs.
- We can create Alarm for any Metrics by just selecting the instance-id of a desired metric name.

#### 5.Dedicating an Instance to Monitoring
- NAT instance: It allows our instances,subnets to download updates without having to make them public.Ideal for security purposes.
- To monitor NAT instance & swapped if its not working,this can be done by PING.
- Launch two instances “NAT” & “Monitoring” in “Custom VPC” not in Default VPC by creating a “Public subnet”, put these two instances in that Public Subnet.
- By using Connect option ssh into “Monitoring” instance which have Public IP
- Both instances will have different Security Groups but same VPC & Subnet
- we have to monitor NAT instance here by using Pinging method
- Now goto CLI, where “Monitoring” ssh has done & connected to its EC2 instance
```ssh
[ec2……]$ping  [paste private-ip of NAT instance] /*it will not work because of SG which inbound rules has only SSH
```
- Now goto EC2 dashboard - Security Groups - select the “Monitoring” Instance SG & Copy its "Group-Id" from description.
- Now goto NAT instance - Security Group - Edit inbound rules by allowing “All ICMP” traffic  - Source “custom IP - paste the copied SG-id” - save
- Now goto CLI, do ping again
```ssh
[ec2……]$ping  [paste private-ip of NAT instance]  /* now ping responds
```
#### 6.Monitoring EBS for Performance and Availability
- EBS(Elastic Block Store) Performance Essentials
  - EBS uses IOPS (I/O operations per second) as a performance measure
- EBS Performance – SSD-backed volumes
  - Two different types of SSD volumes: io1 and gp2
  - gp2 – General Purpose (default)

#### 7.Monitoring RDS for Performance and Availability
- Amazon RDS – Monitoring Metrics
  - CPUUtilization : Percentage of CPU utilization
  - DatabaseConnections : Number of connections that we have at a given point in time
  - DiskQueueDepth : Number of read/write requests waiting to access the disk
  - FreeableMemory : Amount of available RAM
  - FreeStorageSpace : Amount of available storage space
  - SwapUsage :
    - When data is stored in memory on disk
    - Increase in this usually has to do with running out of available RAM
  - ReadIOPS/WriteIOPS:
    - IOPS represent the number of I/O operations completed per second
    - If we don’t have enough IOPS, performance will slow down
  - ReadLatency/WriteLatency
    - Average amount of time taken per disk I/O operation (input/output)
    - Higher latency can be solved with more IOPS
  - ReadThroughput/WriteThroughput  
    - Average number of bytes read or written to or from disk per second

#### 8.Monitoring ElastiCache for Performance and Availability
- Monitoring ElastiCache
  - ElastiCache supports two engines:
    - Memcached
    - Redis
  - Monitoring Metrics:
    1. CPU Utilization
    2. Evictions
    3. CurrConnections
    4. Swap Usage (Memcached)
1. Monitoring ElastiCache – CPU Utilization
  - CPU host-level metrics
    - Memcached is multi-threaded
    - Redis is single-threaded
  - Memcached
    - Can handle loads of up to 90%
    - Above 90% becomes a problem
    - Solution: Increase the size of the node or scale out by adding more nodes
  - Redis
    - Calculate the threshold: (90 / # of CPU cores)
    - Solution:
    - For read-heavy workloads, increase the number of read replicas
    - For write-heavy workloads, use a larger cache instance
2. Monitoring ElastiCache – Evictions
- Evictions happen when a new item is added but there is no more memory space. An older item must be deleted to make space.
  - Memcached solution : Increase instance size or add nodes to your cluster
  - Redis solution : Increase the node size
3. Monitoring ElastiCache – Current Connections
- An increase in CurrConnections could indicate a larger problem with your application
- The application may not be releasing connections
- Choose a threshold based off of your application requirements
4. Monitoring ElastiCache – Swap usage
- Monitor swap usage for Memcached. Swap could be caused because the memory allocated for connection information and other overhead items gets maxed out
- Swap affects performance and should be avoided
- Solution:
- Increase node size
- Increase our ConnectionOverhead parameter value (this will decrease memory available for caching data)

#### 9.Monitoring the Elastic Load Balancer for Performance and Availability
- Amazon ELB – Monitoring Metrics  
  - Latency
    - Time it takes to receive a response
    - Measure the AVG and MAX values to spot abnormal activity
  - BackendConnectionErrors
    - Number of connections that were not successfully established between our load balancer and registered instances
    - Measure SUM and use the different between the minimum and maximums to spot issues
- Elastic Load Balancer (ELB) – Monitoring Metrics
  - SurgeQueueLength
    - Measures the total number of requests that are waiting to be routed by the load balancer
  - SpilloverCount
    - If the SurgeQueueLength is full, requests “spill over” and get dropped Measure the SUM
  - Pre-warming
    - If you are expecting a sudden and very large increase in traffic, you need to pre-warm your ELB to avoid dropped requests
- Elastic Load Balancer (ELB) – Other Metrics
  - HTTP Responses
  - RequestCount : Number of completed requests or connections made
  - HealthyHostCount and UnHealthyHostCount

#### Quiz:
1. Differences between EBS-backed storage and SSD-backed instance store?
- SSD-backed instance store is usually faster because it is physically attached to the host computer, while EBS volumes transfer data over the network which adds latency., SSD-backed instance store is ephemeral while EBS-backed storage is persistent.
2. What best describes burstable performance for t2.micro instances?
- Burstable performance gives you a baseline performance and CPU credits that allow you to burst above this baseline if needed.
3. ElastiCache Redis cluster is under a lot of load and needs to scale. Which of these is the best way to scale your cluster?
- If the load is read-heavy, scale by adding read replicas to your cache cluster. If the load is write-heavy, scale vertically by increasing the node size.
4. Which of the following metrics do not get automatically reported to Amazon CloudWatch from Amazon EC2?
- The amount of memory being used, The amount of swap space used, How much disk space is available

#### Demonstrate Ability to Monitor and Manage Billing and Cost Optimization Processes
#### 1.AWS Billing and Linking AWS Accounts
- AWS Billing Dashboard - Consolidated Billing - Get Started
- In Consolidated billing, we can manage `multiple AWS accounts` in one organization,organize groups & accounts,apply control policies over the usage of AWS resources. We can see all of the billing in individual user account level.
- Each linked account is independent in every other way (i.e.,linked accounts resources can’t be used by us even we are paying bills, only account holders have access to their respective resources not by other linked accounts or organizations.)
- Only disadvantage is for Free Tier, because if every linked account uses free tier,then it will be considered as whole usage across all accounts instead of individual accounts.
- Benefits:
  - Volume Discounts for large enterprises using lot of data.
  - EC2 Reserved Instances
  - Amazon RDS DB instances
  - AWS Credits
- Limits:
  - Limit of 20 consolidated accounts that can be linked to a paying account

#### 2.AWS Billing Dimensions and Metrics for CloudWatch
- AWS Billing Dashboard - Preferences - Receive billing alerts
- Can create Alarms in CloudWatch for billing metrics.

#### 3.Cost Optimizing
- Optimizing Costs – EC2 Reserved Instances
  - Save costs by purchasing reserved instances
  - Reserve instances for 1 to 3 years at a discounted rate
  - Some instances can be sold for a fee
- Low Utilization
  - Save costs by minimizing the number of EC2 instances in-use
  - Set CloudWatch alarms to spin down underutilized instances
  - Find the right balance between availability and cost
Idle Load Balancers
-Remove unused load balancers since we pay per load balancer
Amazon EBS Volumes
-EBS volumes cost, even when not in-use
-Delete unused volumes
    Take a snapshot if you want to keep the data
    Snapshots are usually smaller in size and we only need one snapshot for a volume
-Provisioned IOPS cost more
-Downsize volumes that aren’t anywhere near full capacity
Unassociated Elastic IP Addresses
-EIPs cost money when not in use – disassociate them
-Having more than one EIP associated to an instance costs money
-EIPs on stopped instances cost an hourly fee
Idle Amazon RDS DB Instances
-Take snapshots of unused DB instances, and delete them

4.Using the AWS Price List API and Cost Explorer
AWS Billing Dashboard - Cost Explorer - Launch Cost Explorer

Quiz
Q.What would be the best method for attempting to fix a failing system status check?
-By stopping and starting the instance, the instance will most likely be launched on a different physical host. Since system failures are related to the physical host then this is the best method for resolving the failure.
Q.You've created a CloudWatch alarm to monitor ElastiCache evictions. The CloudWatch alarm begins to alert you that the number of evictions has surpassed your applications requirements. How might you go about resolving the high eviction amount issue?
-Increaseing the size of the ElastiCache instance, Adding another node to the ElastiCache cluster
Q.AWS Allows billing metrics across all consolidated billing accounts from the payer account.
-True
Q.In order to monitor operating system-level metrics such as disk usage, swap usage, and memory usage, you must install EC2 monitoring scripts. These scripts put custom metric data into Amazon CloudWatch. What do you need to do in order to give the "instance permissions" to put those custom metrics in CloudWatch?
-Assign a role to the EC2 instance which will be sending custom metrics to CloudWatch.

High Availability
Implement Scalability and Elasticity Based on Scenario
1.Scalability and Elasticity Essentials

DEMONSTRATE YOUR ABILITY TO ENSURE LEVELS OF FAULT TOLERANCE BASED ON BUSINESS NEEDS
Quiz:
Q.We currently have one Bastion host instance in one of our public subnets, but we're worried about availability if the bastion host or the availability zone that it's in goes down. What can we do?
-Set up two Bastion hosts in two separate Availability Zones and assign an Elastic IP Address to the main Bastion host.
Q.What is the best practice to setup and implement a Bastion host in a VPC?
-Create the instance in your public subnet and assign it a public IP address. Then, use ssh-agent forwarding or OpenSSH ProxyCommand to connect to your private instances.
Q.Which of these services give us access to the underlying Operating System?
-Amazon EMR, EC2
Q.What are conditions that can trigger an Amazon RDS failover to happen?
-Loss of network connectivity to the primary instance, Storage failure on the primary database, Some kind of resource failure with the underlying virtual resources.
