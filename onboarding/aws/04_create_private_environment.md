# Create a private environment

## Step 1: Collect required information
1. Get VPC ID and public subnet IDs
```
vpcid=`terraform state show module.hub-spoke.module.spoke_vpc.aws_vpc.vpc |grep " id "|grep -oE 'vpc-[0-9a-f]{17}'`
pvt_subnet1=`terraform state show 'module.hub-spoke.module.spoke_vpc.module.private_subnet["pvt_subnet_1"].aws_subnet.default'|grep " id "|grep -oE 'subnet-[0-9a-f]{17}'`
pvt_subnet2=`terraform state show 'module.hub-spoke.module.spoke_vpc.module.private_subnet["pvt_subnet_2"].aws_subnet.default'|grep " id "|grep -oE 'subnet-[0-9a-f]{17}'`
pvt_subnet3=`terraform state show 'module.hub-spoke.module.spoke_vpc.module.private_subnet["pvt_subnet_3"].aws_subnet.default'|grep " id "|grep -oE 'subnet-[0-9a-f]{17}'`
```
2. Collect prerequisites names and IDs
```
bucket_name=`terraform state show module.env_prerequisites.aws_s3_bucket.cdp |grep " bucket "|awk -F'"' '{print $2}'`
data_dir_name=`terraform state show 'module.env_prerequisites.aws_s3_object.folders["data"]'|grep " id "|awk -F'"' '{print $2}'`
log_dir_name=`terraform state show 'module.env_prerequisites.aws_s3_object.folders["logs"]'|grep " id "|awk -F'"' '{print $2}'`
storage_location="s3a://${bucket_name}/${data_dir_name}"
log_location="s3a://${bucket_name}/${log_dir_name}"

data_access_inst_profile_arn=`terraform state show module.env_prerequisites.aws_iam_instance_profile.data_access |grep " arn "|awk  -F'"' '{print $2}'`
log_access_inst_profile_arn=`terraform state show module.env_prerequisites.aws_iam_instance_profile.log_access |grep " arn "|awk  -F'"' '{print $2}'`

data_admin_role_arn=`terraform state show module.env_prerequisites.aws_iam_role.dl_admin |grep " arn "|awk  -F'"' '{print $2}'`
idbroker_role_arn=`terraform state show module.env_prerequisites.aws_iam_role.idbroker |grep " arn "|awk  -F'"' '{print $2}'`
log_role_arn=`terraform state show module.env_prerequisites.aws_iam_role.log |grep " arn "|awk  -F'"' '{print $2}'`
ranger_role_arn=`terraform state show module.env_prerequisites.aws_iam_role.ranger |grep " arn "|awk  -F'"' '{print $2}'`

knox_sg_id=`terraform state show module.private_vpc_sgs.aws_security_group.knox |grep " id "|grep -oE 'sg-[0-9a-f]{17}'`
default_sg_id=`terraform state show module.private_vpc_sgs.aws_security_group.default |grep " id "|grep -oE 'sg-[0-9a-f]{17}'`

key_arn=`terraform show|grep "kms_key_arn ="|awk -F'"' '{print $2}'`
ssh_key_name=`terraform state show 'module.env_prerequisites.aws_key_pair.ssh_pub[0]'|grep " key_name " |awk -F'"' '{print $2}'`
```

3. Create environment
```
ENVNAME=dyu-aws-env-poc
cdp environments create-aws-environment \
--environment-name $ENVNAME \
--credential-name $CREDENTIAL \
--region "us-east-1" \
--security-access defaultSecurityGroupId=$default_sg_id,securityGroupIdForKnox=$knox_sg_id \
--workload-analytics \
--enable-tunnel \
--authentication publicKeyId=$ssh_key_name \
--log-storage storageLocationBase=$log_location,instanceProfile=$log_access_inst_profile_arn,backupStorageLocationBase=$log_location \
--vpc-id $vpcid \
--subnet-ids $pvt_subnet1 $pvt_subnet2 $pvt_subnet3 \
--encryption-key-arn $key_arn \
--create-private-subnets \
--no-create-service-endpoints \
--free-ipa multiAz=false
```

4. Create IDBroker mapping
```
cdp environments set-id-broker-mappings \
--environment-name $ENVNAME \
--data-access-role $data_admin_role_arn \
--ranger-audit-role $ranger_role_arn \
--ranger-cloud-access-authorizer-role $ranger_role_arn \
--set-empty-mappings
```

5. Create datalake
```
DATALAKENAME=dyu-aws-poc-dl

cdp datalake create-aws-datalake \
--datalake-name $DATALAKENAME \
--environment-name $ENVNAME \
--cloud-provider-configuration instanceProfile=$data_access_inst_profile_arn,storageBucketLocation=$storage_location \
--scale LIGHT_DUTY \
--runtime 7.3.2 \
--enable-ranger-raz 
```