## Requirements
The following are requirements for this solution and are beyond the scope of this document:

1. Installed and configured awscli with your environment credentials
1. Installed curl package on local environment
1. This configuration uses a `t1.micro` instance type. This type is not available in al AWS regions, so please select carefully a region (like `us-east-1`) that supports it. You can configure your AWS region by setting the `AWS_DEFAULT_REGION` environment variable.

## Before you start
1. ELB classic load balancer was used in the Cloudformation template because the assignment instructions appear to indicate this. 
1. HTTP listener is used due to brief only mentioning this protocol. HTTPS would obviously be preferred but that would also require an ACM certificate and a hosted zone for DNS validation to automate the process. Seems out of scope for this assignment.
1. The S3 bucket provisioning from Stage 2 was added to this Cloudformation stack to make life easier. I didn't want to think about a name for the bucket so I just let CF decide and injected the generated name into my EC2 instance user_data for the HTML page.

## Getting Started
Follow these steps to get this solution running in your environment.

1. Create a new EC2 key pair: `aws ec2 create-key-pair --key-name mykey --query 'KeyMaterial' --output text > MyKey.pem`
    * The value of "KeyMaterial" from the output has been saved to "MyKey.pem"; this is your private ssh key
1. `aws cloudformation create-stack --stack-name mystack --template-body file://cloudformation.json`
1. Check that you have a default VPC in your confgured AWS region: `aws ec2 describe-vpcs --filters Name=isDefault,Values=true --query 'Vpcs[0].CidrBlock'`
    * The CidrBlock of this default VPC must be injected into your cloudformation command in a later step. Save it.
1. Check your current public IP: `curl icanhazip.com`
    * This IP can be used as a Cloudformation parameter to allow ssh access to your address only. Save the value.
1. Create the Cloudformation stack. I have injected the above 2 commands here to save effort: `aws cloudformation create-stack --stack-name mystack --template-body file://cloudformation.json --parameters ParameterKey=SSHLocation,ParameterValue="$(curl icanhazip.com)/32" ParameterKey=DefaultVPCCidr,ParameterValue=$(aws ec2 describe-vpcs --filters Name=isDefault,Values=true --query 'Vpcs[0].CidrBlock')`

## Cleanup
To cleanup resources from your environment simply follow these steps:

1. Empty any objects from the S3 bucket created as part of the stack
1. Delete the Cloudformation stack:
    * `aws cloudformation delete-stack --stack-name mystack`

