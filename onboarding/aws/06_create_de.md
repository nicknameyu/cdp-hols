# Create Data Engineering

## Create a public Data Engineering
```
DENAME=dyu-aws-de
ENVNAME=dyu-aws-env-poc
ENVDETAILS=$(cdp environments describe-environment --environment-name $ENVNAME)

cdp de enable-service \
--name $DENAME \
--env $ENVNAME \
--instance-type r5.4xlarge \
--minimum-instances 0 \
--maximum-instances 50 \
--root-volume-size 100 \
--enable-public-endpoint \
--no-enable-private-network \
--subnets $(echo $ENVDETAILS | jq -r '.environment.network.subnetIds[]') 
```


## Create a private Data Engineering

```
DENAME=dyu-aws-de
ENVNAME=dyu-aws-env-poc
ENVDETAILS=$(cdp environments describe-environment --environment-name $ENVNAME)

cdp de enable-service \
--name $DENAME \
--env $ENVNAME \
--instance-type r5.4xlarge \
--minimum-instances 0 \
--maximum-instances 50 \
--root-volume-size 100 \
--no-enable-public-endpoint \
--enable-private-network \
--subnets $(echo $ENVDETAILS | jq -r '.environment.network.subnetIds[]') 
```