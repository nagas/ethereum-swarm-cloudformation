# Resize Swarm disk

Limitations:
* You can't resize disk more often than every few hours.
* Decreasing the size of Swarm disk is not supported.

```
export SWARM_DISK_DATA_SIZE=100
export STACK_NAME="ethereum-swarm-persistent-resources"

# update the stack to modify disk size
aws cloudformation update-stack --stack-name ${STACK_NAME} --use-previous-template --parameters ParameterKey=SwarmDataDiskSize,ParameterValue=${SWARM_DISK_DATA_SIZE} ParameterKey=AvailabilityZone,UsePreviousValue=true

# get IP address of EC2 instance
export IP=$(aws cloudformation describe-stacks --stack-name ${STACK_NAME} --output text --query 'Stacks[0].Outputs[?OutputKey==`PublicIP`].OutputValue')

# online resize EBS volume
# (make sure proper ssh key name is added to ssh-agent)
ssh ec2-user@$IP 'sudo /root/resize-swarm-volume.sh; df -h'
```
