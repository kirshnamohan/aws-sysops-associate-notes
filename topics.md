* NIC driver
* instance store
* rds read replicas vs. snapshots & performance
* NACl rules order
* evictions in elastic cache cluster
* elastic cachec node size, when to upgrade etc.
* when to add node to cluster cache & when to increase node size
* cloudwatch custom scripts
* bandwith & performance for EC2 instances
* rds read replicas cross region. what is required vpc peering, db version etc.
* snapshots vs backup for rds and redis
* snapshot creation time
* asg and elb when includes new instances
* what is the reason when new ec2 are added to asg but not taken by ELB
* ec2 standard metrics
* SQS operations
* burstable performance

###### NACL
* Rules are evaluated starting with the lowest **numbered rule**. As soon as a rule matches traffic, it's applied regardless of any higher-numbered rule that might contradict it.

###### SQS
* Messages stored for max 14 days
* SQS vs Amazon MQ - use MQ when deploying legacy apps. Amazon MQ uses industry standard interfaces so no refactoring needed to app.
* Standard SQS - **ordering messages not guaranteed** (use sequencing information in messages). **At least once ordering**
* SQS FIFO - guarantees messaging ordering. **Exactly once ordering**
* Long pooling - does not return response till message not delivered
* Change queue type -  You must choose the queue type when you create it. However, it is possible to move to a FIFO queue.
* Cost optimize - use batches (request containing batch or single message cost the same).
* `DeleteMessage` - When you issue a DeleteMessage request on a previously-deleted message, Amazon SQS returns a success response.
* `ReceiveMessage` -  SQS returns a message to you, the message stays in the message queue whether or not you actually receive the message. You're responsible for deleting the message
* visibility timeout - a period of time during which Amazon SQS prevents other consumers from receiving and processing the message.
 - Default 30 seconds
 - Minimum 0 seconds
 - Maximum 12 hours


###### S3 Durability - 11 nines! 99.999999999%
* S3 max object size 5 TB

**Reduced latency, security, disaster recovery, and other use cases:**
* CRR (cross-region replication)
* SRR (single-region replication)

###### EC2
API Calls:
`RunInstances` - start instances (max 20 by default)
**Complete the Amazon EC2 limit increase request form** - region based!
`DescribeInstances` - status of instances
`TerminateInstances` - terminate instances

Local instance store vs. EBS for root device.
* EBS - By using Amazon EBS, data on the root device will persist independently from the lifetime of the instance(start/stop). When instance terimantes EBS is deleted. It can be set flag: `Delete On Terminate` to **NOT** remove EBS when terminating instance.
* Local instance store - only persists during the life of the instance(start/stop)

* It is **supported** to attach **multiple** volumes to instance
* It is **NOT supported** to attach **multiple** instances to one volume




###### Create AMI
```
aws ec2 create-image
```
###### Create EBS Snapshot
```
aws ec2 create-snapshot
```
How to restore volume? (create volume from snapshot, detach old volume, attach new one)
```
1. aws ec2 create-volume –-region us-west-1b –-snapshotid mysnapshot-id
2. aws ec2 detach-volume --volume-id oldvolume-id –
instance-id myec2instance-id
3.  aws ec2 attach-volume --volume-id newvolume-id
--instance-id myec2instance-id --device /dev/sdf
```
*  Snapshots & AMIs can be shared across regions. Stored in S3
* `Private Snapshots` - snapshots shared with me
* `Public Snapshots` - snapshots shared globally

###### Database backup approaches:
* Using EBS Snapshot when DB deployed in regular instance. Should be done in read replica (second instance)
* RDS backups:
 - Automated backup. Up to 35 days (uses transaction logs). `LatestRestorableTime` (usually 5 minutes). `DescribeDBInstances` to find latest restorable time.
 - DB Snapshot. Can be done at every frequency. Restore at every time you want. Use AWS Console or `CreateDBSnapshot` to restore DB. New endpoint will be created (possible to create multiple databases). Console or `DeleteDBSnapshot` to remove old backup(s). Console or `DeleteDBInstance` to remove old instances.

###### Achieving HA
 * Use ELB (users connect via DNS name). ELB points to instances in multiple AZs & verify via health checks
 * Use Elastic IP - If the instance fails, **reassociate** the Elastic IP address to a different instance, which will then start receiving the traffic immediately.**This method is not recommended** because only one instance is receiving all the traffic while the other instance(s) are sitting idle. It also requires a mechanism to detect failure and reassociate the Elastic IP (which you must do yourself).
 * Use ENI - all EC2s have ENIs. They can have **additional** ENIs.
 It is possible to direct traffic to a secondary ENI and then move that secondary ENI to another instance. This is similar to reassigning an Elastic IP address. **ENIs can be reassociated within single AZ!**

Long story short - ELB where possible to achieve HA.