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
- Idle Load Balancers
  - Remove unused load balancers since we pay per load balancer
- Amazon EBS Volumes
  - EBS volumes cost, even when not in-use
  - Delete unused volumes
    - Take a snapshot if you want to keep the data
    - Snapshots are usually smaller in size and we only need one snapshot for a volume
  - Provisioned IOPS cost more
  - Downsize volumes that aren’t anywhere near full capacity
- Unassociated Elastic IP Addresses
  - EIPs cost money when not in use – disassociate them
  - Having more than one EIP associated to an instance costs money
  - EIPs on stopped instances cost an hourly fee
- Idle Amazon RDS DB Instances
  - Take snapshots of unused DB instances, and delete them

#### 4.Using the AWS Price List API and Cost Explorer
- AWS Billing Dashboard - Cost Explorer - Launch Cost Explorer

- Quiz
1. What would be the best method for attempting to fix a failing system status check?
- By stopping and starting the instance, the instance will most likely be launched on a different physical host. Since system failures are related to the physical host then this is the best method for resolving the failure.
2. You've created a CloudWatch alarm to monitor ElastiCache evictions. The CloudWatch alarm begins to alert you that the number of evictions has surpassed your applications requirements. How might you go about resolving the high eviction amount issue?
- Increaseing the size of the ElastiCache instance, Adding another node to the ElastiCache cluster
3. AWS Allows billing metrics across all consolidated billing accounts from the payer account.
- True
4. In order to monitor operating system-level metrics such as disk usage, swap usage, and memory usage, you must install EC2 monitoring scripts. These scripts put custom metric data into Amazon CloudWatch. What do you need to do in order to give the "instance permissions" to put those custom metrics in CloudWatch?
- Assign a `role` to the EC2 instance which will be sending custom metrics to CloudWatch.

### High Availability
#### Implement Scalability and Elasticity Based on Scenario
#### 1.Scalability and Elasticity Essentials
#### 2.Determining Reserved Instance Purchases Based on Business NEEDS
#### 3.AutoScaling vs. Resizing
#### 4.Elastic Load Balancer Sticky Sessions
#### Ensure Level of Fault Tolerance Based on Business Needs
#### 1.DEMONSTRATE YOUR ABILITY TO ENSURE LEVELS OF FAULT TOLERANCE BASED ON BUSINESS NEEDS
- Quiz:
1. We currently have one `Bastion` host instance in one of our public subnets, but we're worried about availability if the bastion host or the availability zone that it's in goes down. What can we do?
- Set up two Bastion hosts in two `separate Availability Zones` and assign an `Elastic IP` Address to the `main Bastion` host.
2. What is the best practice to setup and implement a Bastion host in a `VPC`?
- Create the instance in your `public subnet` and assign it a `public IP` address. Then, use `ssh-agent` forwarding or `OpenSSH ProxyCommand` to connect to your private instances.
3. Which of these `services` give us access to the underlying `Operating System`?
- Amazon EMR, EC2
4. What are conditions that can trigger an Amazon `RDS failover` to happen?
- Loss of `network` connectivity to the primary instance, `Storage` failure on the primary database, Some kind of `resource` failure with the underlying virtual resources.

### Analysis:
#### Optimize the Environment to Ensure Maximum Performance
#### 1.Offloading Database Workload
- Read replicas used for offload work from main DB. Writes goes to `Source Instance` ,Reads goes to `Read Replicas`
- Read Replicas uses `Asynchronous replication` & we can have Lag increases.we can check it through cloud watch metrics.
- Read replicas are different from Multi-AZ failover.
- AWS Console—>RDS—>Launch MySQL Instance(Free Tier)—>Select launched Instance—>Instance Actions—>Modify—> Backup Retention—>Select days, Time —> Save.
- To create Read Replica, Select Instance—>Instance Actions—>Create Read Replica —> Give DB Instance Identifier & required credentials—>create —>select created replica—>show monitoring—>check Replica Lag(if increase in this metric is a serious issue).
- MySQL instance & its read replica instance `Endpoints names will be different`.So we can point `Writes` to MYSQL instance Whereas `Reads` to  Read-Replica instance using Load Balancer Algorithms.
- To Promote Read Replica instance to a "Stand-alone Instance”.
- Select Read Replica instance—>Right click—>Promote Read Replica—>Enable automative backups & retention period—>continue—>promote read replica.
- After this,DB is now being promoted to stand alone instance, which we could use for another application to read & write data.Once this is done, we can enable `Multi-AZ failover` for this DB since it is now Stand alone DB not a read replica anymore.
#### AWS RDS Read Replication vs. Multi-AZ failover deployments
- Read replicas are built primarily for performance and offloading work
- Multi-AZ deployments are used for high availability and durability
- Multi-AZ deployments give us synchronous replication instead of asynchronous
- Multi-AZ deployments are only used to perform a failover, they are idle the rest of the time
- Read replicas are used to serve legitimate traffic
- It is often beneficial to use both of these as complements

#### 2.Initializing (Pre-warming) EBS Volumes
- EC2—>Snapshots—>create a snapshot—>select created snapshot—>Right click —>Create Volume with Volume Type “General Purpose SSD” & select Availability zone same as creating EC2 instance.
- EC2—>Volumes—>Volume must be created in same availability zone as EC2 instance
- EC2—>launch an Amazon linux instance with SG-SSH & existing essential key pair “EBS volume” in same availability zone as in Volume.
- EC2—>Volumes—>Select created Volume—>Right click—>Attach—>select running instance.
- EC2—>Instances—>connect—>ssh into ec2 instance by going to CLI.
```ssh
[ec2@..]$lsblk  /* shows all of the different volumes where they are mounted.as of now there will be no moutpoint
[ec2@..]$sudo dd if=/dev/xvdf of=/dev/null bs=1M  /* ”if" - i/p file, “of" - o/p file “bs”- block size of the read operation
[ec2@..]$sudo yum install -y fio   /* to install fio utility
[ec2@..]$sudo fio --filename=/dev/xvdf --rw=randread --bs=128k --iodepth=32 --ioengine=libaio --direct=1 --name=volume-initialize
```

#### 3.Pre-warming The Elastic Load Balancer
- `Pre-warming` means configuring the elastic load balancer to have enough capacity to handle whatever traffic/demand there is going to be.
- HTTP 503 Error (ELB cannot handle anymore requests)
  - Does not queue requests but instead drops them
- ELB is designed to increase its resource capacity with gradual increases in traffic
- When expecting significant spikes in traffic it is possible the traffic is sent faster than the ELB can “expand”
  - Contact AWS for “pre-warming” of the ELB
- To know what capacity our load balancer can handle, one way is to find out is to use “Load Testing” tools.

- Quiz:
1. One of your customers needs business analytics from data stored in your Amazon RDS database. The problem is, this database is also used to serve data to other customers. The business analytics queries require a lot of table joins and expensive calculations that can't be offloaded to the client, and so you're worried about degrading performance. What can you do?
- Create a read replica specifically for this customer and their business analytics queries. Give them the replica's endpoint.
2. We’re restoring a volume from a snapshot and we need maximum performance as soon as we put the volume in production. How can we ensure maximum performance?
- Initialize (pre-warm) the volume by reading from every single block.

#### Identify Performance Bottlenecks and Implement Remedies
#### 1.Resizing or Changing EBS Root Volume
- To change vol size or vol type to provision iops. First backup the information by creating a snapshot.
- EC2 Dashboard—>Volumes—>Select volume which attached to running instance—> Actions—>create snapshot by giving name & description.
- Check the created snapshot by going to EC2—>Snapshots.
- To create bigger vol or different type of volumes with provisioned iops by using the created snapshot
- EC2—>snapshots—>select created snapshot—>Actions—>Create Volume with vol type “general purpose SSD”, size “100”, Availability Zone same as snapshot’s.
- Now we have to attach this vol to an running instance, but problem is  we want it to be a root vol instead of  being extra attached vol.so we want to replace the smaller vol with this larger vol.i.e., we are going to have some downtime because we have to stop the instance to replace the vols.
- EC2 instance—>select running instance—>stop
- EC2—>Volumes—>select the smaller size vol—>detach it by going to actions—> select bigger volume size—>Actions—>Attach Vol—>select running Instance—> device—>/dev/xvda(which is same as instance root instance)—>attach.
- EC2 instance—>restart—>Connect & ssh into it by going to CLI.
```ssh
[ec2@user…]lsbk     /*shows Root devices details with mount point.
[ec2@user…]df -h   /*to get more info
[ec2@user…]sudo file -s /dev/xvda
[ec2@user…]sudo file -s /dev/xvda1
[ec2@user…]sudo resize2fs  /dev/xvda1 /* gives nothing do because it has 100gib.
```
#### 2.SSL on Elastic Load Balancer
- SSL Certificates can be taxing on an instance and can cause performance issues with spikes in traffic.To remedy that by applying the SSL certificate to the Elastic Load Balancer instead.
- EC2—>Load Balancers—>create load balancer with default VPC—>HTTP & HTTPS protocol with “HTTP” protocol for both—>select a new SG with name & description, Type 2 “Custom TCP Rule”,Port Range “80 & 443” with “Anywhere” source—>Configure security settings—>select upload cert to IAM for 3rd party certificates with name,public key & private keys of certificate—>create.

#### 3.Network Bottlenecks
- Potential Networking Issues
  - One of the primary network bottlenecks comes from EC2 instances
  - Potential causes for bottlenecks
    - Instances are in different Availability Zones, regions, or continents
    - EC2 instance sizes (larger instances generally have better bandwidth performance)
    - Not using enhanced networking features
  - We can check network performance with `iperf3`
    - https://github.com/esnet/iperf
  - VPCs can use VPC Peering to create a reliable connection
    - No single point of failure for communication or bandwidth bottlenecks
1. Create two EC2 instances on AWS
2. Install iperf3 to each instance
3. Start the iperf3 server on one instance
4. Run an iperf3 test from the other instance
- EC2—>launch an linux instance—>configure instance details—>No. of instances “2”—>Auto-assign Public IP—>Enable & leave remaining as it is—> SG with HTTP & SSH inbound custom rules—>download pem file—>name the created 2 instances.
- Select one running instance & connect to it through ssh.
```ssh
[ec2@user…]$sudo yum --enablerepo=epel install iperf iperf3 /*to install iperf3
[ec2@user…]$sudo iperf3 -s -p 80  /* for server listening on port 80
```
- open another terminal window,
- Select another running instance & connect to it through ssh.
```ssh
[ec2@user…]$sudo yum --enablerepo=epel install iperf iperf3 /*to install iperf3
```
- Copy the Public IP of 1st selected instance
```ssh
[ec2@user…]$sudo iperf3 -c <paste copied public ip> -i 1 -t 10 -p 80 /*[ for connection on port 80  “i”-interval of 1sec, “t”- total time of 10 sec]
```
- Using a VPN to access our AWS VPC from our on-premise network means we have to communicate over the open Internet,for that We can use `AWS Direct Connect`

#### Identify Potential Issues on a Given Application Deployment
#### 1.EBS Root Devices on Terminated Instances - Ensuring Data Durability
- Create an Linux AWS EC2 instance & delete to check its volume.
```ssh
$aws ec2 run-instances --image-id=“ami=paste linux AMI” --instance-type "t2.micro” --profile “la” --region ‘us-east-1’ /*instance will be created & running
```
- EC2—>Volumes—>check there will be root vol in “available” state & "in use" vol which is attached to running instance.
OR
- we can launch instance through AWS Console,just uncheck the “Delete on Termination” in Add Storage Tab before launching an instance to not to delete Root Volume on termination  
- Copy Instance ID
```ssh
$aws ec2 terminate-instances --instance-ids=“paste instance id” --profile  “la” --region “us-east-1”   /* to terminate instance
```
- After instance termination,attached “in use” vol will also be terminated.But Root vol will not be deleted.
- To backup data, we can create a snapshot from volume before deleting. Anytime after deletion of a volume, we can create a volume from created snapshot
- if we run the instance command again,
```ssh
$aws ec2 run-instances --image-id=“ami=paste linux AMI” --instance-type "t2.micro” --profile “la” --region ‘us-east-1’ /*instance will be created with default 8Gib volume
```
- Now we have Root vol & default vol, just attach the Root vol to running instance by going to Actions.
- Now copy the new instance id of  running one.
```ssh
$aws ec2 terminate-instances --instance-ids=“paste instance id” --profile  “la” --region “us-east-1” /*terminate instance with 8Gib vol but not Root vol.
```
- This way we can backup the data even instances are terminated.

#### 2.Troubleshooting Auto Scaling Issues
- Attempting to use the wrong subnet
- Availability is no longer available or supported
- Security group does not exist
- Key pair associated does not exist
- Auto Scaling configuration is not working correctly
- Instance type specification is not supported in that Availability Zone
- Auto Scaling service is not enabled on the account
- Invalid EBS device mapping
- Attempting to attach EBS block device to instance-store AMI
- AMI issues
- Placement group attempting to use m1.large (wrong instance type)
- “We currently do not have sufficient instance capacity in the AZ that you requested”
- Updating instance in Auto Scaling group with “suspended state”

### Deployment & Provisioning
#### Demonstrate the Ability to Provision Cloud Resources and Manage Implementation Automation
#### 1.OpsWorks: Overview
- OpsWorks gives us a flexible way to create & manage resources for our applications,as well as the applications themselves.
- we can create a Stack of resources & manage those resources collectively in different layers.These layers can have built-in Chef recipes.
- we can use OpsWorks to:
    - Automate deployments
    - Monitor deployments
    - Maintain deployments
#### AWS OpsWorks
- OpsWorks removes a lot of leg work associated with creating & maintaining application in AWS
- OpsWorks provides abstraction from the underlying infrastructure while still giving plenty of control. Infact, OpsWorks can give us more customization than Elastic Beanstalk.
- It uses `Chef` which is an open source tool that automates infrastructure by turning it into code.This means we can create custom recipes to dictate what our infrastructure & configurations should look like.
- This is a useful tool for longer application life-cycles.
#### AWS OpsWorks - Anatomy
- Stacks:
    - Represent a set of resources that we want to manage as a group. Eg: EC2 instances,EBS vol,load balancers
    - We could build a stack for a development,staging or product environment
- Layers:
    - Used to represent & configure components of a stack. Eg: A layer for web app servers, a layer for the DB, & a layer for the load balancer
    - we can use built-in layers & customize those or create completely custom layers
    - Recipes are added to layers.
- Instances:
    - Must be associated with atleast one layer
    - can run as : 24/7, Load-based, Time-based
-Apps:
    -Apps are deployed to the app layer through a source code repo like Git,SVN or even S3
    -We can deploy an app against a layer & have OpsWorks execute recipes to prepare instances for the application.
AWS OpsWorks -Recipes
-Recipes:
    -Created using the Ruby lang & based off of the Chef deployment software
    -Custom recipes can customize different layers in an app
    -Recipes are run at certain pre-defined events within a stack
        -Setup  they : occurs on a new instance after first boot
        -Configure : occurs on all stack instances when they enter or leave the online state
        -Deploy : occurs when we deploy an app
        -Undeploy : happens when we delete an app from a set of app instances
        -Shutdown : happens when we shut down an instance(but before it is actually stopped)
#### 2.OpsWorks: Creating our First Stack
- AWS Console—>OpsWorks—>Goto OpsWorks Stacks—>Stacks—>Add ur First Stack—>Chef 11 Stack—>Give Name,leave remaining as it is default options—> Advanced—>leave as it is,default—>Add stack.
- Next Add a Layer—>layer type—>php app server—>add layer
- we can check Settings,Recipes,Networks,etc,…of a added layer
- Add another layer—>layer type—>Ganglia—>add layer
- We can add instances to created layers,
- Add instance—>size—>t2.micro—>Advanced—>leave as it is—>Add instance—> start.
- While starting the instance, create an ELB in EC2
- EC2—>load balancer—>Classic load balancer—>name—>default vpc—>select an existing SG—>select default vpc SG—>health check—>TCP,80—>create
- OpsWorks—>layers—>settings—>Network—>select ELB which created above—> shut down instance without w8ing for connections to drain—>save
- OpsWorks—>Apps—>Add App—>Name—>type vl get by default—> data source “none”—>Repo type “git”—>Repo URL `https://github.com/pinehead/opsworks-sysops.git`—>Add App—>Deploy—> command—>select Deploy—> Advanced—> instances—>select “PHP APP server” layer & its instance—>deploy.
- Click on instance we can see logs,public IP,… Goto Public IP,it should work,pulls php page.
- Opsworks—>layers—>ELB—>it will not work,so check SG of ELB by going into EC2
- EC2—>ELB—>SG—>Inbound—>Edit—>source—>0.0.0.0/0—>save.
- Now go back to Opsworks—>layers—>ELB—>it should work.

#### 3.CloudFormation: Essentials
