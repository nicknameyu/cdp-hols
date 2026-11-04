# Create a public environment
```
vpcid=`terraform state show module.hub-spoke.module.hub_vpc.aws_vpc.vpc |grep " id "|grep -oE 'vpc-[0-9a-f]{17}'`
pvt_subnet1=`terraform state show 'module.hub-spoke.module.hub_vpc.module.private_subnet["pvt_subnet_1"].aws_subnet.default'|grep " id "|grep -oE 'subnet-[0-9a-f]{17}'`
pvt_subnet2=`terraform state show 'module.hub-spoke.module.hub_vpc.module.private_subnet["pvt_subnet_2"].aws_subnet.default'|grep " id "|grep -oE 'subnet-[0-9a-f]{17}'`
pvt_subnet3=`terraform state show 'module.hub-spoke.module.hub_vpc.module.private_subnet["pvt_subnet_3"].aws_subnet.default'|grep " id "|grep -oE 'subnet-[0-9a-f]{17}'`
pub_subnet1=`terraform state show 'module.hub-spoke.module.hub_vpc.module.public_subnet["pub_subnet_1"].aws_subnet.default'|grep " id "|grep -oE 'subnet-[0-9a-f]{17}'`
pub_subnet2=`terraform state show 'module.hub-spoke.module.hub_vpc.module.public_subnet["pub_subnet_2"].aws_subnet.default'|grep " id "|grep -oE 'subnet-[0-9a-f]{17}'`
pub_subnet3=`terraform state show 'module.hub-spoke.module.hub_vpc.module.public_subnet["pub_subnet_3"].aws_subnet.default'|grep " id "|grep -oE 'subnet-[0-9a-f]{17}'`

knox_sg_id=`terraform state show module.pub_vpc_sgs.aws_security_group.knox |grep " id "|grep -oE 'sg-[0-9a-f]{17}'`
default_sg_id=`terraform state show module.pub_vpc_sgs.aws_security_group.default |grep " id "|grep -oE 'sg-[0-9a-f]{17}'`
key_arn=`terraform show|grep "kms_key_arn ="|awk -F'"' '{print $2}'`
```
cdp environments create-aws-environment \
--environment-name dyu-aws-env-poc \
--credential-name dyu-aws-poc \
--region "us-east-1" \
--security-access defaultSecurityGroupId=$default_sg_id,securityGroupIdForKnox=$knox_sg_id \
--workload-analytics \
--enable-tunnel \
--authentication publicKeyId="dyu-ssh-key" \
--log-storage storageLocationBase=s3a://dyu-cdp-poc-bucket/logs/,instanceProfile=arn:aws:iam::007856030109:instance-profile/dyu-log-access-instance-profile,backupStorageLocationBase=s3a://dyu-cdp-poc-bucket/logs/ \
--vpc-id $vpcid \
--subnet-ids $pub_subnet1 $pub_subnet2 $pub_subnet3 \
--encryption-key-arn $key_arn \
--no-create-private-subnets \
--no-create-service-endpoints \
--free-ipa multiAz=false