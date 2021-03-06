
# Core Cloudformation template

This CloudFormation template provides a Core VPC configuration, which includes public, private and database subnets, routing tables etc. It also includes a single combined NAT, Bastion & OpenVPN instance. The template currently supports 2 or 3 availability zones (as a parameter) in order to keep things simple.

Note that this stack is not currently built with high availability nor high performance Internet-facing traffic via the NAT instance in mind.

# Build

The instructions below assume that the AWS CLI is used; but it's also possible to build with the AWS Console.

## Prerequisites

1. Clone this repo to an appropriate location on your PC

2. The following environment variables will need to be set correctly for template and configuration locations before running the commands in the following section (adjust as necessary):

```bash
TMPL_DIR=$HOME/dev/git/virtuability/aws-lz/core/template
CNF_DIR=$HOME/dev/git/virtuability/aws-lz/core/config
STACK_NAME=core-development
STACK_REGION=eu-west-1
```

3. An existing SSH key (and name) in the AWS account/region that the CloudFormation stack is created in

4. Adjust the CloudFormation parameters as required in the aws-lz/core/config/$STACK_NAME-parameters.json files

5. The necessary administrative permissions to create the stack

## Core Stack Creation

The following AWS CLI commands are used to validate the template and create, update etc. the stack:

```bash
pip3 install --user --upgrade awscli cfn-lint aws-sam-cli

# Validate the template with cfn-lint
cfn-lint $TMPL_DIR/core.yaml

aws cloudformation validate-template --template-body file://$TMPL_DIR/core.yaml

aws cloudformation create-stack --stack-name $STACK_NAME --template-body file://$TMPL_DIR/core.yaml --parameters file://$CNF_DIR/$STACK_NAME-$STACK_REGION.json --capabilities CAPABILITY_IAM

aws cloudformation update-stack --stack-name $STACK_NAME --template-body file://$TMPL_DIR/core.yaml --parameters file://$CNF_DIR/$STACK_NAME-$STACK_REGION.json --capabilities CAPABILITY_IAM

aws cloudformation list-stacks --stack-status-filter CREATE_IN_PROGRESS CREATE_COMPLETE UPDATE_IN_PROGRESS UPDATE_COMPLETE DELETE_IN_PROGRESS ROLLBACK_COMPLETE ROLLBACK_IN_PROGRESS

aws cloudformation delete-stack --stack-name $STACK_NAME
```

### Change Set Validation

One would be encouraged to use the change sets to validate changes before actually making them.

```bash
aws cloudformation create-change-set  --stack-name $STACK_NAME --change-set-name $STACK_NAME-1 --template-body file://$TMPL_DIR/core.yaml --parameters file://$CNF_DIR/$STACK_NAME-$STACK_REGION.json --change-set-type UPDATE --capabilities CAPABILITY_IAM

aws cloudformation execute-change-set --stack-name $STACK_NAME --change-set-name $STACK_NAME-1

aws cloudformation delete-change-set --stack-name $STACK_NAME --change-set-name $STACK_NAME-1
```

## OpenVPN Client configuration

Once the template has completed building, it is easy to set-up one or more OpenVPN clients.

The utility script /usr/bin/build-client.sh in the NAT VM can create the OpenVPN client configuration as an OVPN file.

Simply login to the NAT instance via SSH and run the script as the `root` user and provide a client name, e.g. "client1", as parameter:

```bash
/usr/bin/build-client.sh client1
```

Once complete, you may find the configuration OVPN file in /etc/openvpn/easy-rsa/client1.ovpn.

Copy the file (or use copy/paste via clipboard) to the OpenVPN client system and import the OVPN file (with Windows client: Import file...)
