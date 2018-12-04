# ethereum-swarm-cloudformation
CloudFormation template for Ethereum Swarm.

## Features
* Swarm version auto-update
* Ability to online resize Swarm disk
* Ability to change Swarm parameters without rebuilding the stack
* Automatic Swarm volume snapshot upon deletion
* Ability to restore Swarm volume from a snapshot

## Create stacks
Ethereum Swarm CloudFormation template consists of two templates:
1. `ethereum-swarm-persistent-resources.yaml` template which is used to manage EBS volume for Swarm data and EIP (persistent IP address).
2. `ethereum-swarm-ec2-instance.yaml` template which is used to manage EC2 instance on which Swarm is running.

You have to provision persistent resources first and only then EC2 instance.

You can terminate/re-create EC2 instance at any time. Existing EBS volume and EIP address will be assigned to the newly created instance. No Swarm data will be lost.

If you delete persistent resources stack, EIP address will be released from your AWS account and EBS volume will be deleted (volume snapshot will be created first).

You can also create these stacks through Amazon Management Console through CloudFormation service section.

### Persistent resources
```
# Swarm data disk size
export SWARM_DATA_DISK_SIZE=50

# set availability zone manually
export AZ="us-east-1a"

# or take it automatically from your default configuration in $HOME/.aws/credentials
export AZ=$(aws ec2 describe-availability-zones --query 'AvailabilityZones[0].ZoneName')

# name for persistent resources stack
export PERSISTENT_RESOURCES_STACK_NAME="ethereum-swarm-persistent-resources"

# create a stack for persistent resources
aws cloudformation create-stack --stack-name ${PERSISTENT_RESOURCES_STACK_NAME} --template-body file://ethereum-swarm-persistent-resources.yaml --parameters ParameterKey=AvailabilityZone,ParameterValue=${AZ} ParameterKey=SwarmDataDiskSize,ParameterValue=${SWARM_DATA_DISK_SIZE}
```

### EC2 instance
```
# Swarm runtime parameters
export SWARM_PARAMETERS="--debug --verbosity 3 --bzznetworkid 3"

# password for Swarm Ethereum account
export SWARM_ACCOUNT_PASSWORD=$(openssl rand -base64 32)

# existing key pairs name in AWS
export KEY_NAME="valid-key-pairs-name"

# instance type to be used
# keep in mind that not all instance types are supported in this template
# and not all instance types are available in all AWS regions
export INSTANCE_TYPE="t3.medium"

# name for EC2 instance stack
export EC2_INSTANCE_STACK_NAME="ethereum-swarm-ec2-instance"

# create a stack for EC2 instance
aws cloudformation create-stack --stack-name ${EC2_INSTANCE_STACK_NAME} --template-body file://ethereum-swarm-ec2-instance.yaml --parameters ParameterKey=SwarmParameters,ParameterValue=${SWARM_PARAMETERS} ParameterKey=SwarmAccountPassword,ParameterValue=${SWARM_ACCOUNT_PASSWORD} ParameterKey=KeyName,ParameterValue=${KEY_NAME} ParameterKey=InstanceType,ParameterValue=${INSTANCE_TYPE}
```

## Maintenance tasks
* [Stack update](docs/stack-update.md)
* [Resize Swarm disk](docs/resize-swarm-disk.md)
* [Change availability zone](docs/change-availability-zone.md)
* [Change Swarm runtime parameters](docs/change-swarm-runtime-parameters.md)

## Improvements
- [ ] Ability to deploy stacks to other VPC than the default VPC.
- [ ] Create automation (scripts) for maintenance tasks.
- [ ] Support for Swarm volume encryption (AWS KMS key).
- [ ] Separate EBS and EIP, so that EIP can be preserved even when re-creating EBS volume?
