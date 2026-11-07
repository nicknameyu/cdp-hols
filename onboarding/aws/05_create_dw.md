# Create Datawarehouse

NOTE: CDW does NOT support custom DNS service. VPC DNS setting must be default DNS. If configured `custom_dns = false` in the terraform template, it must be set to true. 

## Create a public Data Warehouse
```
ENVNAME=dyu-aws-env-poc
ENVDETAILS=$(cdp environments describe-environment --environment-name $ENVNAME)
ENVCRN=$(echo $ENVDETAILS |jq .environment.crn |tr -d '"')
cdp dw create-aws-cluster \
--environment-crn $ENVCRN \
--use-overlay-network \
--no-use-private-load-balancer \
--use-public-worker-node \
--no-enable-private-eks \
--worker-subnet-ids $(echo $ENVDETAILS | jq -r '.environment.network.subnetIds[]') \
--lb-subnet-ids $(echo $ENVDETAILS | jq -r '.environment.network.subnetIds[]') \
```

## Create a private data warehouse

```
ENVNAME=dyu-aws-env-poc
ENVDETAILS=$(cdp environments describe-environment --environment-name $ENVNAME)
ENVCRN=$(echo $ENVDETAILS |jq .environment.crn |tr -d '"')

cdp dw create-aws-cluster \
--environment-crn $ENVCRN \
--use-overlay-network \
--use-private-load-balancer \
--no-use-public-worker-node \
--enable-private-eks \
--worker-subnet-ids $(echo $ENVDETAILS | jq -r '.environment.network.subnetIds[]') \
--lb-subnet-ids $(echo $ENVDETAILS | jq -r '.environment.network.subnetIds[]')
```