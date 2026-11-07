# Create Data Flow

## Create a public Data Flow
```
ENVNAME=dyu-aws-env-poc
ENVDETAILS=$(cdp environments describe-environment --environment-name $ENVNAME)
ENVCRN=$(echo $ENVDETAILS |jq .environment.crn |tr -d '"')
cdp df enable-service \
  --environment-crn $ENVCRN \
  --min-k8s-node-count 3 \
  --max-k8s-node-count 20 \
  --use-public-load-balancer \
  --no-private-cluster \
  --cluster-subnets $(echo $ENVDETAILS | jq -r '.environment.network.subnetIds[]') \
  --load-balancer-subnets $(echo $ENVDETAILS | jq -r '.environment.network.subnetIds[]')
```

## Create a private Data Flow
```
ENVNAME=dyu-aws-env-poc
ENVDETAILS=$(cdp environments describe-environment --environment-name $ENVNAME)
ENVCRN=$(echo $ENVDETAILS |jq .environment.crn |tr -d '"')
cdp df enable-service \
  --environment-crn $ENVCRN \
  --min-k8s-node-count 3 \
  --max-k8s-node-count 20 \
  --no-use-public-load-balancer \
  --private-cluster \
  --cluster-subnets $(echo $ENVDETAILS | jq -r '.environment.network.subnetIds[]') \
  --load-balancer-subnets $(echo $ENVDETAILS | jq -r '.environment.network.subnetIds[]')
```