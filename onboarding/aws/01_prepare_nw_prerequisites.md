# Prepare network and prerequisites

Users have to have sufficient permission to conduct below command on the AWS account. 

Users have to login the AWS account before conducting below command.

## Step 1 - Get terraform repo
Clone terraform repo: https://github.com/nicknameyu/cdp-private-network
go to `aws` directory of the repo
```
# git clone git@github.com:nicknameyu/cdp-private-network.git
# cd cdp-private-network/aws
```

## Step 2 - Prepare terraform variable file
Go to CDP console -> Management console -> Shared Resources -> Credentials -> Create Credential. Write down the `Service Manager Account ID` and `External ID`. 

Example: `terraform.tfvars`
```
owner = "dyu"                                        # owner variable is for tagging and some default variable calculation.
region = "us-east-1"                                 # when not provided, default region is us-west-2

cdp_bucket_name = "dyu-cdp-poc-bucket"               # name of the S3 bucket to be used for CDP environment

cross_account_role = "dyu-cdp-poc"                   # cross account role name for CDP control plane to send AWS api calls to create environment
create_cross_account_role = true                     # If a role already exists, this can be set to false. 
cdp_xaccount_account_id = "<Service Manager Account ID>"
cdp_xaccount_external_id = "<External ID>"           # external ID can be found when creating CDP credential on CDP UI. 
cmk_key_name = "dyu-cdp-poc-key"                     # this is to prepare CMK for CDP. Not necesary if CMK is not in the practice.

default_permission = true                            # to configure whether to use defaule permission for cross account role. when set to false, reduced permission is attached.
custom_dns = false                                   # When set to true, jump server on hub VPC will be used as the DNS server. P.S. CDW doesn't support custom DNS.
//create_eks_role = true                             # create_eks_role is only for reduced permission. Consult CDP expert for this option. 

tags = {
    usecase = "testing"
}

## These flags are to enable the policies for CAI, CDE, CDF, CDW, and Custom Managed Key support. 
enable_ai = true
enable_cmk = true
enable_de = true
enable_df = true
enable_dw = true
create_eks_role = false                             # this variable is used to reduced permission. set to false when using default permision.
```

## Step 3 - create resources
```
# terraform init
# terraform plan
# terraform apply
```
This terraform template below resources
- A pair of hub-spoke VPCs
  - a transit gateway for connectivity between hub and spoke VPC
  - hub VPC
    - 3 public subnets and associated route tables
    - 3 private subnets and assocoated route tables
    - a internet gateway
    - a NAT gateway subnet 
    - a NAT Gateway
    - a firewall and firewall subnet for egress control
    - a linux DNS server on one of the public subnet
    - S3 gateway endpoint
  - spoke VPC
    - 3 private subnets and assocoated route tables
    - S3 gateway endpoint
- CDP prerequisites
  - a S3 bucket
  - a cross account role with associated permission policies
  - 4 CDP roles
    - assumer role and policies
    - idbroker role and policies
    - logger role and policies
    - ranger audit role and policies
  - KMS Key
  - AWS SSH Key resource