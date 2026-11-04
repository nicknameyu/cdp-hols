# Prepare the Credential

CDP Cloud credential is the principal for CDP to access the customer's Cloud tenant. For AWS, it is the cross account role. 

** Step 1 ** - Get the Cross Account Role ARN 
If the cross account role was precreated before this lab, use below command to get the cross account role ARN. 
```
XACCTROLEARN=terraform state show 'module.cdp_cross_account_role.data.aws_iam_role.cross_account[0]' \
  | grep '^[[:space:]]*arn[[:space:]]*=' \
  | awk -F'"' '{print $2}'
```

If the cross account role was precreated in the first step of this lab, use below command to get the cross account role ARN. 
```
XACCTROLEARN=terraform state show 'module.cdp_cross_account_role.aws_iam_role.cross_account[0]' \
  | grep '^[[:space:]]*arn[[:space:]]*=' \
  | awk -F'"' '{print $2}'
```

** Step 2 ** - Create credential
```
CREDENTIAL=<credential name>
cdp environments create-aws-credential \
--credential-name $CREDENTIAL \
--role-arn $XACCTROLEARN
```