# Stack update

With every template change, we will be providing detailed instructions on how to update/migrate stacks without data loss.

You can update stacks on your own, but that might destroy all your data.

Limitations:
* Some changes might require EC2 instance replacement (e.g., AMI, instance type). In that case, you should delete the stack first and create it again (Swarm data and IP address will be preserved).
* We recommend to use AWS Management Console to update stacks as it gives more visibility (e.g., `change set` details).

```
export STACK_NAME="ethereum-swarm-ec2-instance"
export STACK_FILE="ethereum-swarm-ec2-instance.yaml"

export STACK_PARAMETERS=$(aws cloudformation describe-stacks --stack-name ${STACK_NAME} --output text --query 'Stacks[0].Parameters')

aws cloudformation update-stack --stack-name ${STACK_NAME} --template-body file://${STACK_FILE}--parameters ${STACK_PARAMETERS}
```
