# Change availability zone

Limitations:
* You cannot change Availability Zone for your resources without re-creating all stacks.
* Public IP will change (EIP will be re-created).

If you don't care about Swarm data (Ethereum account/password generated for you), you can delete all stacks and re-create them in Availability Zone of your preference.

If you care about Swarm data, here is how you can re-create all stacks without Swarm data loss:

```
PERSISTENT_RESOURCES_STACK_NAME="ethereum-swarm-persistent-resources"
EC2_INSTANCE_STACK_NAME="ethereum-swarm-ec2-instance"

# save EC2 instance stack parameters
EC2_INSTANCE_STACK_PARAMETERS=$(aws cloudformation describe-stacks --stack-name ${EC2_INSTANCE_STACK_NAME} --query 'Stacks[0].Parameters')

# print parameters before deleting a stack
echo ${EC2_INSTANCE_STACK_PARAMETERS}

# delete EC2 instance stack
aws cloudformation delete-stack --stack-name ${EC2_INSTANCE_STACK_NAME}

export VOLUME_ID=$(aws cloudformation describe-stacks --stack-name ${PERSISTENT_RESOURCES_STACK_NAME}  --output text --query 'Stacks[0].Outputs[?OutputKey==`SwarmDataVolume`].OutputValue')
export SNAPSHOT_ID=$(aws ec2 create-snapshot --volume-id ${VOLUME_ID} --output text --query 'SnapshotId')

echo ${VOLUME_ID}
echo ${SNAPSHOT_ID}

# wait for snapshot completion
watch -n15 "aws ec2 describe-snapshots --snapshot-ids ${SNAPSHOT_ID} | jq '.Snapshots[0].Progress'"

# delete stack for persistent resources
aws cloudformation delete-stack --stack-name ${PERSISTENT_RESOURCES_STACK_NAME}

# wait for the stack to be deleted
# stack deletion may fail due to a timeout on snapshot creation
# if that happens, delete the stack again (we manually created a snapshot in the previous step)
watch -n5 "aws cloudformation describe-stacks --stack-name ${PERSISTENT_RESOURCES_STACK_NAME} --output text --query 'Stacks[0].StackStatus'"

# new availability zone
export AZ="us-east-1b"

# create persistent resources stack
aws cloudformation create-stack --stack-name ${PERSISTENT_RESOURCES_STACK_NAME} --template-body file://ethereum-swarm-persistent-resources.yaml --parameters ParameterKey=AvailabilityZone,ParameterValue=${AZ} ParameterKey=SnapshotID,ParameterValue=${SNAPSHOT_ID}

# wait for the stack to be created
watch -n5 "aws cloudformation describe-stacks --stack-name ${PERSISTENT_RESOURCES_STACK_NAME} --output text --query 'Stacks[0].StackStatus'"

# create EC2 instance stack
aws cloudformation create-stack --stack-name ${EC2_INSTANCE_STACK_NAME} --template-body file://ethereum-swarm-ec2-instance.yaml --parameters ${EC2_INSTANCE_STACK_PARAMETERS}
```
