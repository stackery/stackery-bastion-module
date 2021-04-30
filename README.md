# CloudFormation Module: Stackery::Open::Bastion::MODULE

This is the source for the Stackery::Open::Bastion::MODULE that provisions an EC2 machine as a [bastion server](https://en.wikipedia.org/wiki/Bastion_host) into a VPC in your AWS account for you to connect to over SSH. In particular, you can use [AWS EC2 Instance Connect](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/Connect-using-EC2-Instance-Connect.html) or [AWS Systems Manager Session Manager](https://docs.aws.amazon.com/systems-manager/latest/userguide/session-manager.html). Once connected to the bastion server you can then tunnel ports or otherwise access further private resources like Amazon RDS Databases connected in the same VPC.

## Usage
First, enable the Stackery::Open::Bastion::MODULE [AWS CloudFormation](https://aws.amazon.com/cloudformation/) module (instructions TBD).

Next, provision a bastion server by adding this to an [AWS CloudFormation](https://aws.amazon.com/cloudformation/) template:

```yaml
Resources:
  Bastion:
    Type: Stackery::Open::Bastion::MODULE
    Properties:
      VPCId: <VPC ID>
      VPCSubnets:
        - <Public Subnet ID 1>
        - <Public Subnet ID 2>
        - ...
```

## Parameters

### VPCId
**Required**

The ID of the VPC you want the bastion server placed into.

### VPCSubnets
**Required**

A list of VPC subnet IDs you want the bastion server placed into. Only one bastion server will run at a time, but it will be placed into one of these subnets. By providing multiple subnets spread across multiple availability zones (AZ) you will ensure connectivitity even if an AZ goes down. A new bastion server will be started inside a subnet in a different AZ if this occurs.

These subnets must be "public" subnets, meaning the bastion instances will receive public IP addresses when they are provisioned and can reach the internet through an Internet Gateway. You can find an example VPC and subnet configuration [here](https://docs.aws.amazon.com/vpc/latest/userguide/VPC_Scenario1.html).

### InstanceClass
*Optional*
Default: t4g.nano

The EC2 instance class to provision. For low bandwidth, low usage scenarios the default t4g.nano class works well at a low price point. Override the class if you need higher bandwidth or CPU requirements.

## Operations
The instance sends logs and metrics to [AWS CloudWatch](https://aws.amazon.com/cloudwatch/) for monitoring.

An [AWS Systems Manager State Manager](https://docs.aws.amazon.com/systems-manager/latest/userguide/systems-manager-state.html) Association runs once each day to update to the latest Systems Manager and CloudWatch Agents.

### Metrics
Standard EC2 metrics along with additional CPU, disk, swap, and memory metrics are sent to CloudWatch Metrics with dimensions **AutoScalingGroupName**, **ImageId**, **InstanceId**, and **InstanceType**.

### Logs
The /var/log/secure, /var/log/audit/audit.log, and /var/log/messages log files are sent to CloudWatch Logs under the /<stack name>/bastion/secure, /<stack name>/bastion/audit.log, /<stack name>/bastion/messages log groups. Log messages are sent to a log stream with the EC2 instance ID as the stream name.