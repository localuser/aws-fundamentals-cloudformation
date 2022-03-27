## Requirements
The following are requirements for this solution and are beyond the scope of this document:

1. Installed and configured awscli with your environment credentials
1. Installed curl package on local environment
1. This configuration uses a `t1.micro` instance type. This type is not available in al AWS regions, so please select carefully a region (like `us-east-1`) that supports it. You can configure your AWS region by setting the `AWS_DEFAULT_REGION` environment variable.

## Before you start
1. ELB classic load balancer was used in the Cloudformation template because the assignment instructions appear to indicate this. 
1. HTTP listener is used due to brief only mentioning this protocol. HTTPS would obviously be preferred but that would also require an ACM certificate and a hosted zone for DNS validation to automate the process. Seems out of scope for this assignment.
1. The S3 bucket provisioning from Stage 2 was added to this Cloudformation stack to make life easier. I didn't want to think about a name for the bucket so I just let CF decide and injected the generated name into my EC2 instance user_data for the HTML page.
1. An older Amazon Linux AMI has been used in the Cloudformation script. The correct AMI will be chosen based on your configured AWS region for deployment. The map of AMI IDs was available from a template somewhere and saved me some time looking for the latest versions in every region. Updates are done on boot for security reasons.

## Getting Started
Follow these steps to get this solution running in your environment.

1. Create a new EC2 key pair: 
    * `aws ec2 create-key-pair --key-name mykey --query 'KeyMaterial' --output text > MyKey.pem`
    * The value of "KeyMaterial" from the output has been saved to "MyKey.pem"; this is your private ssh key
1. Check that you have a default VPC in your confgured AWS region: 
    * `aws ec2 describe-vpcs --filters Name=isDefault,Values=true --query 'Vpcs[0].CidrBlock'`
    * The CidrBlock of this default VPC must be injected into your cloudformation command in a later step. Save this value.
1. Check your current public IP: 
    * `curl icanhazip.com`
    * This IP can be used as a Cloudformation parameter to allow ssh access to your address only. Save this value.
1. Create the Cloudformation stack. (I have injected the 2 above commands here to save effort): 
    * `aws cloudformation create-stack --stack-name mystack --template-body file://cloudformation.json --parameters ParameterKey=SSHLocation,ParameterValue="$(curl icanhazip.com)/32" ParameterKey=DefaultVPCCidr,ParameterValue=$(aws ec2 describe-vpcs --filters Name=isDefault,Values=true --query 'Vpcs[0].CidrBlock')`

## Connecting to the instance

Your local machine should be able to make ssh connections to the new EC2 instance. Login with your private key from the Getting Started steps, and the username "ec2-user".

## Cleanup
To cleanup resources from your environment simply:

1. Empty any objects from the S3 bucket created as part of the Cloduformation stack
1. Delete the Cloudformation stack:
    * `aws cloudformation delete-stack --stack-name mystack`

## Closing
Thank you for this opportunity.