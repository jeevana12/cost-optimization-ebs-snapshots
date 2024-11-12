# cost-optimization-ebs-snapshots
A Python script that utilizes the Boto3 library to manage Amazon EC2 EBS snapshots.
This script identifies the stale ebs snapshots that are no longer needed and deletes if it is not associated with the active ec2 instances.This will help organisations to be maintain their storage costs.
Lambda functions: This are event driven functions .Here in this we will invoke the function through cloud watch.
Code structure:
1.import Boto3: Will import boto3 module so that it interacts with aws apis.
2.def lambda_handler(event, context): Here we are defining a function lambda_handler where it has 2 params event(that contains data passed to the function) and context (runtime info)
3.ec2 = boto3.client('ec2'): This specifies that the boto3 module will make api calls to the ec2 service.
4.response = ec2.describe_snapshots(OwnerIds=['self']): Fetching all the snapshots associated with the ec2 service owned by our account and stores it in a response variable.
5.instances_response = ec2.describe_instances(Filters=[{'Name': 'instance-state-name', 'Values': ['running']}]): Fetches all the running ec2 instances.
6.active_instance_ids = set(): This initalizes an empty set and stores all the active instances id.
7.for reservation in instances_response['Reservations']: Iterates through the instances that has been called through api.
8.for instance in reservation['Instances']: Nested loop iterates through the current rservation.
9.active_instance_ids.add(instance['InstanceId']): This adds the instance id to an empty set that we created before.
10.for snapshot in response['Snapshots']:  This loop iterates through each snapshot retrieved earlier.
11.snapshot_id = snapshot['SnapshotId']: This line extracts the snapshot ID from the current snapshot.
12.volume_id = snapshot.get('VolumeId'): This retrieves the associated volume ID, if available.
13.if not volume_id:: This condition checks if the snapshot is not associated with any volume.
14.ec2.delete_snapshot(SnapshotId=snapshot_id): If the snapshot is unattached, this line deletes it.
15.print(f"Deleted EBS snapshot {snapshot_id} as it was not attached to any volume."): This line logs the deletion of the snapshot.
16.else:: This block executes if the snapshot is associated with a volume.
try:: This begins a try block to handle potential exceptions.
17.volume_response = ec2.describe_volumes(VolumeIds=[volume_id]): This line retrieves details about the volume associated with the snapshot.
18.if not volume_response['Volumes'][0]['Attachments']:: This condition checks if the volume has any attachments.
19.ec2.delete_snapshot(SnapshotId=snapshot_id): If the volume is not attached, the snapshot is deleted.
20.print(f"Deleted EBS snapshot {snapshot_id} as it was taken from a volume not attached to any running instance."): This logs the deletion of the snapshot.
21.except ec2.exceptions.ClientError as e:: This block catches any client errors that occur during the volume description.
22.if e.response['Error']['Code'] == 'InvalidVolume.NotFound':: This checks if the error indicates that the volume was not found.
23.ec2.delete_snapshot(SnapshotId=snapshot_id): If the volume is not found, the snapshot is deleted.
24.print(f"Deleted EBS snapshot {snapshot_id} as its associated volume was not found."): This logs the deletion of the snapshot due to the missing volume.








