# Cloud cost optimization using Boto3

### Introduction

What are the reasons for companies to migrate from on-premises servers to the Cloud? 
- Maintenance Overhead
- Highly Expensive
- Not very efficient

What if a company migrated to the Cloud and these problems still exist? What if the costs aren't being saved either?
Is it the cloud provider's fault? No. It is a shared responsibilty.
To explain it - cloud providers like AWS, Azure etc, provide you with numerous services for compute, storage, monitoring and many more, with least amount of effort on your end. This doesn't mean you can use the resources irresponsibly, it will incur additional charges and you won't benefit from the migration. So, it is a shared responsibility. Use resources responsibly, reap all the benefits.

As a DevOps Engineer, one of your responsibilities is to maintain all the created resources and ensure the deletion of stale resources

### Problem Explanation

Imagine you created an EC2 instance with an attached root EBS volume, to store the data related to your application hosted on that instance. You decided to take a backup of that EBS volume by taking a snapshot, to restore it as a new volume in a different AZ when a disaster strikes. But, you need the snapshots only as long as the application exists. If you decide to shut down your application on the EC2 instance, there will be no need for the snapshot. In that scenario, the snapshots associated with the instance needs to be deleted to be financially efficient. 

If you just have one or two EC2 intances to manage, this won't be a big deal, and you can easily manage creation and deletion snapshots manually. But, companies usually have a lot of resources and EC2 instances to manage in various AWS regions. So the above task is almost impossible to be done manually.

This is when Lambda functions in AWS emerge.

### Why Lambda?

Lambda functions are serverless, it doesn't mean there aren't any servers. It just means that you don't have to manage the underlying servers. All you have to do is write code to perform your desired actions. For this function, you have to add a trigger i.e., for the function to be triggered when an EC2 instance is deleted. This action leads to the running of the function, which will check if any stale snapshots created from the deleted instance exist and delete.

A Lambda function is developed utilizing AWS Boto3, which is an AWS SDK, to automate the identification and deletion of stale snapshots, thus optimizing cloud expenses. Through proactive management of snapshots, the Lambda function contributes to cost optimization strategies within AWS environments.

### CODE STRUCTURE:
 
 >import Boto3: Will import boto3 module so that it interacts with aws apis.

>def lambda_handler(event, context): Here we are defining a function lambda_handler where it has 2 params event(that contains data passed to the function) and context 
   (runtime info)

>ec2 = boto3.client('ec2'): This specifies that the boto3 module will make api calls to the ec2 service.

>response = ec2.describe_snapshots(OwnerIds=['self']): Fetching all the snapshots associated with the ec2 service owned by our account and stores it in a response variable.

>instances_response = ec2.describe_instances(Filters=[{'Name': 'instance-state-name', 'Values': ['running']}]): Fetches all the running ec2 instances.

>active_instance_ids = set(): This initalizes an empty set and stores all the active instances id.

>for reservation in instances_response['Reservations']: Iterates through the instances that has been called through api.

>for instance in reservation['Instances']: Nested loop iterates through the current rservation.

>active_instance_ids.add(instance['InstanceId']): This adds the instance id to an empty set that we created before.

>for snapshot in response['Snapshots']:  This loop iterates through each snapshot retrieved earlier.

>snapshot_id = snapshot['SnapshotId']: This line extracts the snapshot ID from the current snapshot.

>volume_id = snapshot.get('VolumeId'): This retrieves the associated volume ID, if available.

>if not volume_id:: This condition checks if the snapshot is not associated with any volume.

>ec2.delete_snapshot(SnapshotId=snapshot_id): If the snapshot is unattached, this line deletes it.

>print(f"Deleted EBS snapshot {snapshot_id} as it was not attached to any volume."): This line logs the deletion of the snapshot.

>else:: This block executes if the snapshot is associated with a volume.

>try:: This begins a try block to handle potential exceptions.

>volume_response = ec2.describe_volumes(VolumeIds=[volume_id]): This line retrieves details about the volume associated with the snapshot.

>if not volume_response['Volumes'][0]['Attachments']:: This condition checks if the volume has any attachments.

>ec2.delete_snapshot(SnapshotId=snapshot_id): If the volume is not attached, the snapshot is deleted.

>print(f"Deleted EBS snapshot {snapshot_id} as it was taken from a volume not attached to any running instance."): This logs the deletion of the snapshot.

>except ec2.exceptions.ClientError as e:: This block catches any client errors that occur during the volume description.

>if e.response['Error']['Code'] == 'InvalidVolume.NotFound':: This checks if the error indicates that the volume was not found.

>ec2.delete_snapshot(SnapshotId=snapshot_id): If the volume is not found, the snapshot is deleted.

>print(f"Deleted EBS snapshot {snapshot_id} as its associated volume was not found."): This logs the deletion of the snapshot due to the missing volume.








