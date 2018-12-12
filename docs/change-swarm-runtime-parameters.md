# Change Swarm runtime parameters

Limitations:
* It takes few minutes for changes to be applied.

```
export SWARM_RUNTIME_PARAMETERS="--debug --verbosity 1 --bzznetworkid 3"
export STACK_NAME="ethereum-swarm-ec2-instance"

# get current parameters for EC2 instance stack
STACK_PARAMETERS=$(aws cloudformation describe-stacks --stack-name ${STACK_NAME} --query 'Stacks[0].Parameters')

# update swarm runtime parameters
NEW_PARAMETERS=$(jq -r "map((select(.ParameterKey == \"SwarmParameters\") | .ParameterValue) |= \"${SWARM_RUNTIME_PARAMETERS}\")" <<< "${STACK_PARAMETERS}")

# update the stack with new parameters
aws cloudformation update-stack --stack-name ${STACK_NAME} --use-previous-template --parameters ${NEW_PARAMETERS} 
```
